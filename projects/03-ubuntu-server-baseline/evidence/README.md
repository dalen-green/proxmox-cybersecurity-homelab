# LAB-03 Evidence Pack

> **Technical status:** Verified for the documented IPv4 baseline\
> **Portfolio status:** Published\
> **Evidence captured:** 9 of 9 core artifacts reviewed\
> **Last reviewed:** 2026-09-29

## 1. Purpose and claim boundary

This pack supports the [Ubuntu Server baseline](../README.md) with selected screenshots and command output. It distinguishes three different kinds of proof:

- **Configuration evidence** shows that a setting or recovery point exists.
- **Runtime evidence** shows what was active at a recorded moment.
- **Functional evidence** shows the observed result of a controlled test.

All nine planned core artifacts are present, legible, sanitized, and reviewed. The result is limited to the documented Ubuntu state and IPv4 tests. It does not establish continuous compliance, equivalent IPv6 enforcement, comprehensive upstream-network isolation, or an independent backup and restore capability.

## 2. Cross-project network evidence

LAB-02 already records Ubuntu's participation in the isolated IPv4 network, so those artifacts are referenced instead of duplicated:

| Existing evidence | What it supports |
|---|---|
| [`OPN-E03` — OPNsense DHCP lease](../../02-opnsense-segmentation/evidence/03-opnsense-dhcp-lease.png) | Ubuntu received `10.10.10.149` from OPNsense |
| [`OPN-E04` — Ubuntu IPv4 egress](../../02-opnsense-segmentation/evidence/04-ubuntu-ipv4-egress.txt) | Ubuntu used `10.10.10.1` as gateway and DNS service and completed an approved HTTPS request |
| [`OPN-E06` and `OPN-E07` — correlated routed SSH test](../../02-opnsense-segmentation/evidence/README.md#opn-e06--denied-ssh-test) | OPNsense blocked and logged one controlled IPv4 TCP/22 flow from Ubuntu toward the protected routed destination |

Those files validate the routed path. They do not replace the Ubuntu account, SSH, UFW, service, or recovery evidence below.

## 3. Final artifact index

| ID | Artifact | Review status | What it directly supports |
|---|---|---|---|
| `UBU-E01` | [`01-proxmox-ubuntu-hardware.png`](01-proxmox-ubuntu-hardware.png) | **Captured — reviewed** | VM `101` resources and its single intended VirtIO adapter on `vmbr1`; virtual MAC redacted |
| `UBU-E02` | [`02-ubuntu-platform-and-updates.txt`](02-ubuntu-platform-and-updates.txt) | **Captured — reviewed** | Ubuntu 26.04.1 LTS, kernel `7.0.0-34-generic`, `amd64`, synchronized UTC time, successful metadata refresh, two phased deferrals, and no reboot requirement |
| `UBU-E03` | [`03-ubuntu-account-separation.txt`](03-ubuntu-account-separation.txt) | **Captured — reviewed** | Separate administrative and standard accounts, final group lists, unrestricted sudo for the administrative role, and explicit sudo denial for the standard role |
| `UBU-E04` | [`04-ubuntu-services-and-ssh.txt`](04-ubuntu-services-and-ssh.txt) | **Captured — reviewed** | Valid OpenSSH syntax, socket activation, TCP/22 listeners, and hardened effective SSH settings |
| `UBU-E05` | [`05-ubuntu-ufw-status.txt`](05-ubuntu-ufw-status.txt) | **Captured — reviewed** | Active UFW, logging, default-deny incoming policy, allowed outgoing traffic, disabled routed traffic, and TCP/22 limited to `10.10.10.2` |
| `UBU-E06` | [`06-ubuntu-services-and-auth-log.txt`](06-ubuntu-services-and-auth-log.txt) | **Captured — reviewed** | Running-service inventory and a sanitized successful public-key authentication event; all-listener diagnostic reviewed separately during collection |
| `UBU-E07` | [`07-ubuntu-ufw-independent-test.txt`](07-ubuntu-ufw-independent-test.txt) | **Captured — reviewed** | Allowed public-key SSH from `10.10.10.2` and UFW-blocked TCP/22 SYN packets from independent source `10.10.10.1` |
| `UBU-E08` | [`08-proxmox-ubuntu-snapshot.png`](08-proxmox-ubuntu-snapshot.png) | **Captured — reviewed** | The labeled `ubuntu-baseline-2026-09-29` recovery point exists for VM `101` and excludes RAM state |
| `UBU-E09` | [`09-ubuntu-rollback-validation.txt`](09-ubuntu-rollback-validation.txt) | **Captured — reviewed** | The post-snapshot marker is absent after rollback while IPv4 addressing, the default route, SSH, and UFW are restored |

Supporting recovery evidence:

| Artifact | Purpose |
|---|---|
| [`09a-ubuntu-pre-rollback-marker.txt`](09a-ubuntu-pre-rollback-marker.txt) | Records the controlled marker filename and SHA-256 before rollback so its later absence has a documented pre-change state |

## 4. Review notes and captions

### `UBU-E01` — Proxmox Ubuntu hardware

The screenshot shows VM `101`, two CPU cores, a 40 GiB main disk, the displayed memory values, and exactly one VirtIO network device attached to `vmbr1`. The NIC firewall checkbox is visible, but this does not by itself prove that a Proxmox firewall policy was configured or tested.

> **Caption:** Proxmox hardware view for Ubuntu VM `101`, showing its compute and disk resources and one intended VirtIO network attachment to internal bridge `vmbr1`. The virtual MAC address is redacted.

### `UBU-E02` — Platform and update state

The September 29 capture followed approved maintenance. It records a successful metadata refresh and a newer kernel than the earlier draft. `python3-distupgrade` and `ubuntu-release-upgrader-core` remained deferred by Ubuntu's phased rollout; the simulated upgrade reported zero eligible installations and no reboot requirement.

> **Caption:** Timestamped Ubuntu platform output showing Ubuntu 26.04.1 LTS on kernel `7.0.0-34-generic`, `amd64` architecture, synchronized UTC time, successful package-metadata refresh, two release-upgrader packages deferred by phased rollout, and no pending reboot.

### `UBU-E03` — Account separation and sudo authorization

Account and hostname values are replaced with stable role labels. The final administrative group list no longer contains `lxd`, after LXD was confirmed absent and the unused membership was removed. `sudo -l` results show the administrative role may run commands through sudo and the standard role may not.

> **Caption:** Sanitized Ubuntu account review showing separate Bash-capable administrative and standard roles, final group memberships without the unused `lxd` group, unrestricted sudo authorization for the administrative role, and explicit sudo denial for the standard role.

This validates the intended local sudo-role separation. It is not an exhaustive audit of every filesystem ACL, Linux capability, polkit rule, or application-specific permission.

### `UBU-E04` — OpenSSH configuration and runtime

The output records valid syntax, enabled and active `ssh.socket`, an active daemon, IPv4 and IPv6 TCP/22 listeners, public-key-only administrative authentication, denied root login, reduced authentication attempts, and disabled forwarding features. Separate controlled client checks established successful key-only access and rejected password attempts; those screenshots were not published because they contained identifiers.

> **Caption:** Timestamped Ubuntu output showing valid OpenSSH syntax, working socket activation, TCP/22 listeners, key-only administrative authentication, denied root login, and disabled password, keyboard-interactive, agent, TCP, stream-local, and X11 forwarding paths. The account is represented by a stable role label.

### `UBU-E05` — UFW policy

The saved status shows the host firewall configuration before independent testing. The rule permits SSH only when Ubuntu sees the source as `10.10.10.2`, the deliberately enabled Proxmox management endpoint.

> **Caption:** Timestamped UFW status showing active low-volume logging, default-deny inbound policy, allowed outbound traffic, disabled routed traffic, and TCP/22 permitted only from the temporary Proxmox management source at `10.10.10.2`.

### `UBU-E06` — Running services and authentication log

The publication copy retains the service inventory and security meaning of the SSH event while replacing its unique hostname, username, process ID, ephemeral client port, and key fingerprint. A separate `ss` diagnostic showed SSH as the only remotely listening server service; loopback DNS and chrony listeners and the expected DHCP client socket were reviewed but not duplicated in this artifact.

> **Caption:** Timestamped Ubuntu inventory of 20 running service units and a sanitized log event showing successful Ed25519 public-key authentication from authorized source `10.10.10.2`.

### `UBU-E07` — Independent UFW validation

The same SSH listener was tested from two sources. A fresh permitted management connection generated an `Accepted publickey` event from `10.10.10.2`. A connection attempt from the OPNsense LAN endpoint at `10.10.10.1` generated UFW block entries for IPv4 TCP destination port 22. The repeated block rows are expected SYN retransmissions while the client waited for a response.

Because the allowed and denied checks targeted an already confirmed listener on the same port, an additional temporary listener would not have improved attribution and was not created.

> **Caption:** Sanitized Ubuntu logs showing successful public-key SSH from permitted source `10.10.10.2` and repeated UFW blocks for IPv4 TCP/22 from independent source `10.10.10.1`. Host, account, process, MAC, ephemeral-port, and key identifiers are redacted.

### `UBU-E08` — Proxmox recovery point

The screenshot shows the full snapshot label, creation time, `RAM: No`, descriptive baseline note, and the later `NOW` state for VM `101`.

> **Caption:** Proxmox snapshot view showing the labeled `ubuntu-baseline-2026-09-29` recovery point for Ubuntu VM `101`, created without RAM state after updates, account separation, SSH hardening, service review, and UFW validation.

This artifact proves that the recovery point exists. It does not prove rollback by itself and must not be described as an independent backup.

### `UBU-E09` — Controlled rollback validation

The supporting pre-rollback file records a harmless marker and its SHA-256 after the snapshot. Following a powered-off rollback to the named snapshot, Ubuntu was started and reached through SSH. The final artifact records that the marker is absent and that `ens18`, the DHCP-derived default route, SSH, and UFW returned to their expected states.

> **Caption:** Timestamped post-rollback checks showing that the controlled post-snapshot marker is absent and that Ubuntu restored its expected IPv4 address and route, active SSH service, and source-restricted UFW policy.

The temporary SSH tunnel was then stopped, `10.10.10.2/24` was removed from Proxmox `vmbr1`, and a follow-up IPv4 query returned no address. That cleanup was confirmed during the collection session; it is not represented as an additional core artifact.

## 5. Sanitization record

Removed or replaced before publication:

- Real Ubuntu usernames and hostname
- Virtual MAC addresses and logged layer-2 identifiers
- SSH key fingerprints, process IDs, and ephemeral source ports
- Proxmox management and unrelated upstream addresses
- Unrelated device names or log entries

Intentionally retained because it supports the documented claims:

- VM ID `101`, bridge `vmbr1`, and interface `ens18`
- Lab-only addresses `10.10.10.1`, `10.10.10.2`, and `10.10.10.149`
- TCP/22, firewall action, timestamps, and UFW policy fields
- OS release, kernel, architecture, UIDs, shells, and role-based group relationships
- Snapshot label, marker checksum, and post-rollback health results

The raw `UBU-E07` collection file was not committed because it contained a real hostname, username, MAC addresses, process ID, ephemeral ports, and SSH fingerprint. The linked publication copy replaces those values while preserving the tested fields.

## 6. Integrity record

SHA-256 values for the reviewed publication files:

| File | SHA-256 |
|---|---|
| `01-proxmox-ubuntu-hardware.png` | `e7f7ac82fcc710cc4e846217d11f96db575916b7a82c719a65e08d056d0365bd` |
| `02-ubuntu-platform-and-updates.txt` | `494572b8f142abbde6a5e659e4ac31ccbddfee684546e9268d9e7d979ee54c37` |
| `03-ubuntu-account-separation.txt` | `6e4630c9f0336841b7a3cbf0eea66b37453ad793935e98d564b52370441039ab` |
| `04-ubuntu-services-and-ssh.txt` | `e0205c52ab854e941e70bc2c56913a7fb609299ee9c3efaef6475af0975e201e` |
| `05-ubuntu-ufw-status.txt` | `cacdda3a3dd3da99a7ee4a77fc3f84764760a4c67c327878ff822f7b141cda71` |
| `06-ubuntu-services-and-auth-log.txt` | `4dbda8450fc508adcec057496b69b21b6d40809588ad39836d5529844e7ecf42` |
| `07-ubuntu-ufw-independent-test.txt` | `824aed50a9c191991d6f3d97853fc9fb04ad292162aa22f1f9ebdcfc34c4278d` |
| `08-proxmox-ubuntu-snapshot.png` | `2b3b650f0afeea13d8a121852e3a367b6d5b31bae6a8a28d385b188980b3e993` |
| `09-ubuntu-rollback-validation.txt` | `c9f522472c4b885185b5ac762866b90fa73d6d33149ac2996fbb22ea4bf3fe73` |
| `09a-ubuntu-pre-rollback-marker.txt` | `b99aac8820a1270b385c073dccc4f11f8a15062e1596eb82d1269e0c69abcc34` |

## 7. Publication verification

- [x] All nine core artifacts are present under their planned filenames.
- [x] Captions describe only what the corresponding artifact visibly or textually supports.
- [x] VM `101` has one intended network adapter on `vmbr1` and none on `vmbr0`.
- [x] Administrative and standard accounts have different effective sudo results.
- [x] The unnecessary administrative `lxd` membership was removed before the final account capture.
- [x] Final patch state was recorded after metadata refresh, including phased deferrals and reboot state.
- [x] Effective SSH settings, runtime state, and listening sockets were reviewed.
- [x] UFW configuration and a permitted-versus-blocked IPv4 TCP/22 test were reviewed.
- [x] Running services and all TCP/UDP listeners were reviewed together during collection.
- [x] Authentication and firewall evidence is short, relevant, timestamped, and sanitized.
- [x] Snapshot existence and successful rollback are supported by separate artifacts.
- [x] The controlled marker has both a pre-rollback checksum record and a post-rollback absence check.
- [x] The latest temporary tunnel and Proxmox lab-side IPv4 address were removed after transfer.
- [x] No publication artifact contains credentials, private keys, password hashes, protected upstream addresses, MAC addresses, or unique machine identifiers.
- [x] The snapshot is explicitly distinguished from an independent backup.
- [x] IPv6 and broader network limitations remain visible.

The evidence pack is **Verified for the documented IPv4 baseline / Published**.

## 8. Remaining improvements outside this completed scope

- Validate or deliberately disable unused IPv6 paths.
- Test an independent backup and restore from separate storage.
- Test VM startup order and service recovery after a full Proxmox host restart.
- Repeat patch, listener, and firewall reviews periodically because these artifacts are point-in-time records.
- Expand network-segmentation tests before adding deliberately vulnerable targets.
