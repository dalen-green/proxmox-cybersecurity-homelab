# LAB-03 Evidence Pack

> **Technical status:** In progress\
> **Portfolio status:** Drafting — evidence collection in progress\
> **Evidence captured:** 6 of 9 core artifacts reviewed\
> **Last reviewed:** 2026-09-26

## 1. Purpose

I am collecting this evidence so I can explain the [Ubuntu baseline](../README.md) using saved results. Six core artifacts are reviewed so far, and three still need to be collected. The existing files describe their capture dates; revising this index does not rerun the checks.

I am learning to distinguish:

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
| `UBU-E01` | [`01-proxmox-ubuntu-hardware.png`](01-proxmox-ubuntu-hardware.png) | **Captured — reviewed** | VM `101` has one network adapter on `vmbr1`, with CPU, memory-display values, and disk configuration visible; startup options are not shown |
| `UBU-E02` | [`02-ubuntu-platform-and-updates.txt`](02-ubuntu-platform-and-updates.txt) | **Captured — reviewed** | Dated Ubuntu 26.04.1 LTS platform state, synchronized UTC time, successful metadata refresh, no outstanding reboot requirement, and two explicitly identified upgrades deferred by phased rollout |
| `UBU-E03` | [`03-ubuntu-account-separation.txt`](03-ubuntu-account-separation.txt) | **Captured — reviewed** | Separate interactive administrative and standard accounts use Bash shells, with only the administrative role holding sudo-group membership |
| `UBU-E04` | [`04-ubuntu-services-and-ssh.txt`](04-ubuntu-services-and-ssh.txt) | **Captured — reviewed** | Valid OpenSSH syntax, socket activation, TCP/22 listeners, and hardened effective settings; separate setup notes record key-login and password-rejection tests, whose client screenshots are not published |
| `UBU-E05` | [`05-ubuntu-ufw-status.txt`](05-ubuntu-ufw-status.txt) | **Captured — reviewed** | Active UFW state, low-volume logging, default-deny inbound policy, allowed outbound traffic, disabled routed traffic, and SSH limited to `10.10.10.2` |
| `UBU-E06` | [`06-ubuntu-services-and-auth-log.txt`](06-ubuntu-services-and-auth-log.txt) | **Captured — reviewed** | Timestamped running-service inventory and a sanitized successful public-key SSH event from the authorized `10.10.10.2` management source; a separate all-listener diagnostic was reviewed during collection |
| `UBU-E07` | `07-ubuntu-ufw-independent-test.txt` | **Needed** | A fresh connection from the allowed management source succeeds, while an independent unauthorized source is blocked with matching UFW evidence; a temporary unapproved listener makes the port-denial test meaningful |
| `UBU-E08` | `08-proxmox-ubuntu-snapshot.png` | **Needed** | A clearly labeled clean Ubuntu snapshot exists in Proxmox |
| `UBU-E09` | `09-ubuntu-rollback-validation.txt` | **Needed** | A controlled post-snapshot change disappears after rollback and expected network, SSH, and UFW health checks still pass |

An item remains **Needed** until the exact file is present, legible, sanitized, and reviewed against its caption. The project must not be marked **Verified** merely because all filenames exist.

## 4. Capture Instructions and Draft Captions

### `UBU-E01` — Proxmox Ubuntu hardware

In Proxmox, select VM `101` → **Hardware**. Keep the processor, memory, disk, and every network-device row visible.

Before uploading or publishing, obscure virtual MAC addresses and any unrelated identifiers. Retain VM ID `101`, storage names, resource values, bridge names, and device types.

> **Caption:** Proxmox hardware view for Ubuntu VM `101`, showing the memory row as `2.00 GiB / 4.00 GiB`, two CPU cores, a 40 GiB main virtual disk, and one VirtIO adapter on `vmbr1`. The NIC firewall option is checked and the virtual MAC address is redacted. This view does not establish a tested Proxmox firewall policy or startup order.

I need a configuration review before describing the memory display more specifically. The earlier fixed-2-GiB caption omitted the second value shown in the screenshot.

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

The initial capture showed valid syntax and TCP/22 listeners but also exposed broader-than-needed settings. A follow-up diagnostic established that Ubuntu uses `ssh.socket` to hold both listeners and activate `ssh.service`; the service's disabled unit-file state is therefore intentional rather than a startup failure.

The administrative account was enrolled with an Ed25519 public key before hardening. After validating the proposed settings with `sshd -t` and `sshd -T`, the configuration was reloaded while an existing administrative session remained open. A new forced key-only connection succeeded, and separate client attempts with public-key authentication disabled were rejected without a password prompt. The working screenshots are not published because they contain client and server identifiers; the consolidated artifact records the resulting effective server policy.

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
  sudo sshd -T |
    grep -E '^(port|addressfamily|listenaddress|maxauthtries|permitrootlogin|pubkeyauthentication|passwordauthentication|kbdinteractiveauthentication|permitemptypasswords|authenticationmethods|x11forwarding|disableforwarding|allowtcpforwarding|allowstreamlocalforwarding|allowagentforwarding|allowusers) ' |
    sed -E 's/^allowusers .*/allowusers admin_username/'
  printf '\n%s\n' 'SSH listening sockets:'
  sudo ss -lntp '( sport = :22 )'
} 2>&1 | tee ~/04-ubuntu-services-and-ssh.txt
```

Do not publish private keys, authorized-key contents, password hashes, or the complete SSH configuration file.

> **Caption:** Timestamped Ubuntu output showing valid OpenSSH syntax; enabled and active socket activation; active daemon state; IPv4 and IPv6 TCP/22 listeners; public-key-only authentication for the administrative role; denied root login; reduced authentication attempts; and disabled password, keyboard-interactive, agent, TCP, stream-local, and X11 forwarding paths. The account name is replaced by a stable role label.

### `UBU-E05` — UFW policy

Run these two short commands:

```bash
date -u | tee ~/05-ubuntu-ufw-status.txt
```

```bash
sudo ufw status verbose | tee -a ~/05-ubuntu-ufw-status.txt
```

The first command creates the timestamped evidence file. The second appends the effective UFW policy without overwriting the timestamp.

> **Caption:** Timestamped UFW status showing an active host firewall with low-volume logging, default-deny inbound policy, allowed outbound traffic, disabled routed traffic, and TCP/22 permitted only from the temporary Proxmox management endpoint at `10.10.10.2`.

### `UBU-E06` — Running services and authentication log

The final artifact was created with three short commands after one controlled successful SSH login:

```bash
date -u | tee ~/06-ubuntu-services-and-auth-log.txt
systemctl --type=service --state=running --no-pager | tee -a ~/06-ubuntu-services-and-auth-log.txt
sudo journalctl -u ssh --no-pager | grep 'Accepted publickey' | tail -n 1 | tee -a ~/06-ubuntu-services-and-auth-log.txt
```

A separate `sudo ss -lntup` diagnostic was reviewed during collection. It showed SSH as the only remotely listening server service. DNS and chrony listeners were loopback-only, and the DHCP client listener was expected on `ens18`. Optional-looking units such as ModemManager, multipathd, udisks2, and upower did not expose listening network ports, so none was disabled solely because of its name.

The publication copy replaces the hostname, account name, process ID, ephemeral source port, and public-key fingerprint. It retains the lab-only source `10.10.10.2` because that address correlates the successful authentication event with the source-restricted UFW rule in UBU-E05.

> **Caption:** Timestamped Ubuntu inventory of 20 running service units and a sanitized SSH log event showing successful Ed25519 public-key authentication for the administrative role from the authorized `10.10.10.2` management source. Unique host, account, process, port, and key identifiers are redacted.

### `UBU-E07` — Independent UFW validation

The current rule permits SSH only from `10.10.10.2`, the temporary Proxmox management source. A second lab VM with a different source address is not expected to reach SSH automatically. The test must match that policy.

The planned checks are:

1. Establish a fresh SSH connection through the allowed management path and record Ubuntu's observed source and the successful result.
2. From a separate authorized test VM on `vmbr1`, attempt SSH and record the expected denial with a corresponding UFW log entry.
3. Start a temporary listener on an unapproved port on Ubuntu, confirm that it is listening, and attempt a connection from the independent VM. Record the client result and matching UFW block.
4. Stop the temporary listener and remove any deliberately added test rule. Recheck the original UFW policy and confirm temporary management-path cleanup after administration is complete.

If a permitted-service test from the independent VM is needed, its exact source and service require an explicitly documented temporary allow rule. Its result must be labeled as a test under that temporary policy. The default plan above leaves the existing source restriction in place. No test is complete yet, and exact commands will be chosen when the second endpoint is available.

> **Draft caption:** Correlated client and Ubuntu evidence showing successful SSH from the permitted management source, denied SSH from an independent source, and a UFW block for a confirmed temporary listener on an unapproved port, followed by cleanup.

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
- [x] Patch state at the recorded capture time is documented after refreshing package metadata.
- [x] Effective SSH settings, runtime state, and listening sockets have been reviewed.
- [x] Active UFW configuration, defaults, logging, and the scoped SSH rule have been reviewed.
- [ ] A fresh allowed management connection and independent UFW denial tests have been documented against the actual source-specific policy.
- [x] Running services and all TCP/UDP listeners have been reviewed against one another.
- [ ] Effective-permission review beyond the captured sudo-group membership is complete.
- [ ] Cleanup of temporary test services/rules and the latest temporary management path has been confirmed.
- [x] Authentication evidence is short, relevant, timestamped, and sanitized.
- [ ] Snapshot existence and successful rollback are supported by different evidence.
- [ ] No artifact contains credentials, keys, password hashes, protected addresses, MAC addresses, or unique machine identifiers.
- [ ] The project does not call a snapshot an independent backup.
- [ ] Project README, roadmap, architecture, security boundaries, lessons learned, and root status agree at publication time.

The project remains **In progress / Drafting** until this checklist is complete.
