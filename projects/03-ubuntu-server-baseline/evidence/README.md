# LAB-03 Evidence Pack

> **Technical status:** In progress  
> **Portfolio status:** Drafting — evidence collection in progress  
> **Evidence captured:** 3 of 9 core artifacts reviewed  
> **Last reviewed:** 2026-09-09

## 1. Purpose

This pack will support the claims in the [Ubuntu Server Security Baseline project](../README.md) with selected, sanitized configuration and functional evidence.

The pack deliberately distinguishes:

- A setting that is present
- A service that is currently running
- A control that has been functionally tested
- A recovery procedure that has actually restored the expected state

## 2. Existing Cross-Project Evidence

LAB-02 already contains reviewed evidence that the Ubuntu endpoint participates in the isolated IPv4 network. These files should be referenced rather than duplicated:

| Existing evidence | What it supports |
|---|---|
| [`OPN-E03` — OPNsense DHCP lease](../../02-opnsense-segmentation/evidence/03-opnsense-dhcp-lease.png) | Ubuntu received the lab address `10.10.10.149` from OPNsense |
| [`OPN-E04` — Ubuntu IPv4 egress](../../02-opnsense-segmentation/evidence/04-ubuntu-ipv4-egress.txt) | Ubuntu used `10.10.10.1` for its default route and DNS service and received an approved HTTPS response |
| [`OPN-E06` and `OPN-E07` — correlated SSH test](../../02-opnsense-segmentation/evidence/README.md#opn-e06--denied-ssh-test) | Ubuntu generated a controlled IPv4 TCP/22 flow that was blocked and logged by OPNsense |

These artifacts validate network participation and the routed firewall path. They do not validate Ubuntu's account controls, effective SSH configuration, UFW policy, service exposure, or recoverability.

## 3. Core Evidence Required for Publication

| Evidence ID | Planned filename | Status | Required proof |
|---|---|---|---|
| `UBU-E01` | [`01-proxmox-ubuntu-hardware.png`](01-proxmox-ubuntu-hardware.png) | **Captured — reviewed** | VM `101` has one intended network adapter on `vmbr1`, with its relevant CPU, memory, disk, and startup configuration visible |
| `UBU-E02` | [`02-ubuntu-platform-and-updates.txt`](02-ubuntu-platform-and-updates.txt) | **Captured — reviewed** | Ubuntu 26.04.1 LTS platform state, synchronized UTC time, successful metadata refresh, completed reboot, and two explicitly identified upgrades deferred by phased rollout |
| `UBU-E03` | [`03-ubuntu-account-separation.txt`](03-ubuntu-account-separation.txt) | **Captured — reviewed** | Separate interactive administrative and standard accounts use Bash shells, with only the administrative role holding sudo-group membership |
| `UBU-E04` | `04-ubuntu-services-and-ssh.txt` | **Working validation complete — final recapture pending** | OpenSSH socket activation and a forced Ed25519 key-only login are validated; broader-than-needed effective settings require hardening before final capture |
| `UBU-E05` | `05-ubuntu-ufw-status.txt` | **Needed** | UFW active state, default policy, logging state, and explicit allow rules |
| `UBU-E06` | `06-ubuntu-services-and-auth-log.txt` | **Needed** | Reviewed running services and a short, timestamped authentication-log excerpt from a controlled event |
| `UBU-E07` | `07-ubuntu-ufw-independent-test.txt` | **Needed** | An independent source on `vmbr1` reaches an approved service while a controlled unapproved listening service is blocked and logged by UFW |
| `UBU-E08` | `08-proxmox-ubuntu-snapshot.png` | **Needed** | A clearly labeled clean Ubuntu snapshot exists in Proxmox |
| `UBU-E09` | `09-ubuntu-rollback-validation.txt` | **Needed** | A controlled post-snapshot change disappears after rollback and expected network, SSH, and UFW health checks still pass |

An item remains **Needed** until the exact file is present, legible, sanitized, and reviewed against its caption. The project must not be marked **Verified** merely because all filenames exist.

## 4. Capture Instructions and Draft Captions

### `UBU-E01` — Proxmox Ubuntu hardware

In Proxmox, select VM `101` → **Hardware**. Keep the processor, memory, disk, and every network-device row visible.

Before uploading or publishing, obscure virtual MAC addresses and any unrelated identifiers. Retain VM ID `101`, storage names, resource values, bridge names, and device types.

> **Caption:** Proxmox hardware view for Ubuntu VM `101`, showing 2 GiB of memory, two CPU cores, a 40 GiB virtual disk, and one VirtIO network adapter attached to internal bridge `vmbr1` with Proxmox firewalling enabled. The virtual MAC address is redacted.

### `UBU-E02` — Platform and update state

Run the following from the administrative account. It refreshes package metadata but does not automatically install or remove packages:

```bash
{
  date -u +'%Y-%m-%dT%H:%M:%SZ'
  printf 'Package metadata refresh: '
  if sudo apt-get update -qq; then printf 'successful\n'; else printf 'failed\n'; fi
  . /etc/os-release
  printf 'OS: %s\n' "$PRETTY_NAME"
  printf 'Kernel: '
  uname -r
  printf 'Architecture: '
  dpkg --print-architecture
  timedatectl show --property=Timezone --property=NTPSynchronized
  printf 'Pending upgrades: '
  apt list --upgradable 2>/dev/null | sed '1d' | wc -l
  printf 'Reboot required: '
  if [ -e /var/run/reboot-required ]; then printf 'yes\n'; else printf 'no\n'; fi
} 2>&1 | tee ~/02-ubuntu-platform-and-updates.txt
```

If pending upgrades or a required reboot are reported, preserve this initial result, review and apply the approved maintenance, and recapture the final publication artifact afterward.

If packages remain pending after approved maintenance, use a simulated upgrade to record whether Ubuntu is deliberately deferring them. This appends only the relevant phasing summary to the same artifact:

```bash
{
  printf '\nPhased-update review: '
  date -u +'%Y-%m-%dT%H:%M:%SZ'
  sudo apt-get -s upgrade 2>&1 | sed -n '/deferred due to phasing:/,/^[0-9][0-9]* upgraded,/p'
} | tee -a ~/02-ubuntu-platform-and-updates.txt
```

> **Caption:** Timestamped Ubuntu platform output recording a successful package-metadata refresh, Ubuntu 26.04.1 LTS on kernel `7.0.0-31-generic`, `amd64` architecture, synchronized UTC time, and no outstanding reboot requirement. The two remaining audit-library upgrades are explicitly identified as deferred by Ubuntu's phased rollout.

### `UBU-E03` — Account separation

The initial diagnostic capture showed only one interactive local account. A separate non-sudo standard account was then created and the final artifact was recaptured with stable role-based labels.

Replace the two placeholders with the actual Ubuntu account names before running:

```bash
ADMIN_ACCOUNT='replace-with-admin-account'
STANDARD_ACCOUNT='replace-with-standard-account'

{
  date -u +'%Y-%m-%dT%H:%M:%SZ'
  printf '%s\n' 'Interactive local accounts:'
  getent passwd "$ADMIN_ACCOUNT" |
    awk -F: '{printf "admin_username uid=%s shell=%s\n", $3, $7}'
  getent passwd "$STANDARD_ACCOUNT" |
    awk -F: '{printf "standard_username uid=%s shell=%s\n", $3, $7}'
  printf '\n%s\n' 'Account identities and group memberships:'
  id "$ADMIN_ACCOUNT" | sed "s/${ADMIN_ACCOUNT}/admin_username/g"
  id "$STANDARD_ACCOUNT" | sed "s/${STANDARD_ACCOUNT}/standard_username/g"
  printf '\n%s\n' 'Sudo group membership:'
  getent group sudo | sed "s/${ADMIN_ACCOUNT}/admin_username/g;s/${STANDARD_ACCOUNT}/standard_username/g"
} 2>&1 | tee ~/03-ubuntu-account-separation.txt
```

The collection command replaces account names with consistent role-based labels while retaining UIDs, shells, and group relationships.

> **Caption:** Sanitized Ubuntu account inventory showing separate Bash-capable administrative and standard accounts, with sudo membership limited to the administrative role.

### `UBU-E04` — Services, SSH, and listening sockets

The initial capture showed valid syntax, an active SSH daemon, and TCP/22 listening on IPv4 and IPv6. A follow-up diagnostic confirmed that `ssh.socket` is enabled and active, holds both listeners, and activates `ssh.service`; the service's disabled unit-file state is therefore intentional rather than a startup failure. A forced Ed25519 key-only login then succeeded for the administrative account. Password authentication, key-based root login, X11 forwarding, and TCP forwarding remain enabled in the initial state. Complete the approved hardening plan before recapturing the publication artifact.

The working SSH capture and activation diagnostic are not included as core artifacts. The final `UBU-E04` recapture will consolidate the validated activation model, hardened effective settings, and listening sockets without retaining superseded intermediate files.

Run:

```bash
{
  date -u +'%Y-%m-%dT%H:%M:%SZ'
  printf 'SSH configuration syntax: '
  if sudo sshd -t; then printf 'valid\n'; else printf 'invalid\n'; fi
  printf '\n%s\n' 'SSH service and socket activation:'
  printf 'ssh.service enabled: '
  systemctl is-enabled ssh.service
  printf 'ssh.service active: '
  systemctl is-active ssh.service
  printf 'ssh.socket enabled: '
  systemctl is-enabled ssh.socket
  printf 'ssh.socket active: '
  systemctl is-active ssh.socket
  systemctl list-sockets ssh.socket --no-pager
  printf '\n%s\n' 'Selected effective SSH settings:'
  sudo sshd -T | grep -E '^(port|addressfamily|listenaddress|permitrootlogin|passwordauthentication|pubkeyauthentication|kbdinteractiveauthentication|permitemptypasswords|maxauthtries|x11forwarding|allowtcpforwarding|allowagentforwarding|allowusers|allowgroups) '
  printf '\n%s\n' 'Listening sockets:'
  sudo ss -lntup
} 2>&1 | tee ~/04-ubuntu-services-and-ssh.txt
```

Do not publish private keys, authorized-key contents, password hashes, or the complete SSH configuration file.

> **Draft caption:** Timestamped Ubuntu output showing valid OpenSSH configuration syntax, enablement and runtime state, selected effective security settings, and the host's listening sockets.

### `UBU-E05` — UFW policy

Run:

```bash
{
  date -u +'%Y-%m-%dT%H:%M:%SZ'
  sudo ufw status verbose
} 2>&1 | tee ~/05-ubuntu-ufw-status.txt
```

> **Draft caption:** Timestamped UFW status showing the active host firewall, default policy, logging state, and explicitly permitted inbound services.

### `UBU-E06` — Running services and authentication log

After one controlled successful SSH login from the authorized administrative workstation, run:

```bash
{
  date -u +'%Y-%m-%dT%H:%M:%SZ'
  printf '%s\n' 'Running services:'
  systemctl --no-pager --type=service --state=running
  printf '\n%s\n' 'Recent SSH events:'
  sudo journalctl -u ssh --since '-30 minutes' --no-pager | tail -n 50
} 2>&1 | tee ~/06-ubuntu-services-and-auth-log.txt
```

The log excerpt must be sanitized for upstream addresses, usernames, hostnames, and unrelated events while retaining timestamps, service, authentication result, and event meaning.

> **Draft caption:** Sanitized Ubuntu baseline showing currently running services and a timestamped SSH authentication event from an authorized administrative session.

### `UBU-E07` — Independent UFW validation

This test requires an independent, authorized source attached to `vmbr1`. It should demonstrate both:

1. The approved SSH service remains reachable.
2. A deliberately started test listener on an unapproved port is blocked and produces a corresponding UFW log entry.

The listener must be temporary, bound only for the controlled test, and stopped immediately afterward. Exact commands will be selected after the independent test endpoint is available so the procedure matches its operating system and installed tools.

> **Draft caption:** Correlated client and Ubuntu firewall evidence showing that the approved SSH service is reachable while a controlled unapproved listening service is blocked and logged by UFW.

### `UBU-E08` and `UBU-E09` — Snapshot and rollback

Create a clearly labeled clean snapshot only after the preceding baseline review is complete. Then make one harmless, documented change, roll back, and verify that:

- The post-snapshot marker is absent
- Ubuntu receives its expected lab networking
- OpenSSH is active
- UFW is active with the expected policy

The snapshot view establishes that a recovery point exists; the post-rollback text artifact establishes that restoration actually returned the expected state.

> **Draft caption for `UBU-E08`:** Proxmox snapshot view showing the labeled clean recovery point for Ubuntu VM `101`.

> **Draft caption for `UBU-E09`:** Timestamped post-rollback checks showing removal of the controlled change and restoration of expected Ubuntu networking, OpenSSH, and UFW state.

## 5. Sanitization Record

Remove or replace:

- Proxmox management, upstream-network, and controlled-test-target addresses
- Personal or unique usernames and hostnames
- MAC addresses and DHCP client identifiers
- SSH public-key fingerprints when they are not needed for a claim
- Private keys, password hashes, credentials, tokens, and recovery material
- Unrelated household-device names or log entries
- Serial numbers, UUIDs, machine IDs, and raw configuration exports

Retain because it supports the documented claims:

- VM ID `101` and generic Ubuntu role labels
- Bridge `vmbr1`, guest interface names, and the lab-only `10.10.10.0/24` network
- Operating-system release, kernel, architecture, and UTC timestamps
- UIDs, shells, and sanitized group relationships
- Service names, enablement/runtime state, listening ports, and protocols
- Selected effective SSH settings that contain no secrets
- UFW status, default policy, logging state, rule action, and tested ports
- Snapshot label, test sequence, and post-rollback health results

Redaction must cover each protected value completely without hiding the surrounding label, action, port, or timestamp required to interpret the evidence.

## 6. Publication Verification

- [ ] Every core artifact is present under its planned filename.
- [ ] Captions describe only what the corresponding artifact visibly proves.
- [x] VM `101` has one intended network adapter on `vmbr1` and none on `vmbr0`.
- [x] Account evidence distinguishes administrative and standard privileges.
- [x] Current patch state is recorded after refreshing package metadata.
- [ ] Effective SSH settings, runtime state, and listening sockets have been reviewed.
- [ ] UFW configuration and independent allow/block behavior both have evidence.
- [ ] Authentication evidence is short, relevant, timestamped, and sanitized.
- [ ] Snapshot existence and successful rollback are supported by different evidence.
- [ ] No artifact contains credentials, keys, password hashes, protected addresses, MAC addresses, or unique machine identifiers.
- [ ] The project does not call a snapshot an independent backup.
- [ ] Project README, roadmap, architecture, security boundaries, lessons learned, and root status agree at publication time.

The project remains **In progress / Drafting** until this checklist is complete.
