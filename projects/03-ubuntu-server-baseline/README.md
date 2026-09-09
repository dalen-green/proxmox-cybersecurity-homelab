# Project 03: Ubuntu Server Security Baseline

> **Technical status:** In progress  
> **Portfolio status:** Drafting — evidence collection in progress  
> **Platform:** Ubuntu Server virtual machine on Proxmox VE  
> **Last reviewed:** 2026-09-09

## 1. Objective

Establish a documented Ubuntu Server baseline that demonstrates operating-system administration, account separation, patching, remote administration, host-firewall policy, service and port review, log analysis, and recoverability.

## 2. Current Scope

The current implementation state is:

- Installed Ubuntu Server as Proxmox VM `101`
- Attached the VM to the internal-only `vmbr1` lab network
- Created separate administrative and standard accounts with sudo membership limited to the administrative role
- Installed operating-system updates
- Configured OpenSSH access
- Hardened OpenSSH for the administrative role and validated an Ed25519 key-only login plus password-authentication rejection
- Enabled the UFW host firewall
- Verified IPv4 addressing, routing, DNS, and approved HTTPS egress through OPNsense during `LAB-02`

These statements remain provisional within this project until their LAB-03 evidence artifacts are captured and reviewed. The project does not yet claim a completed security baseline.

## 3. Architecture and Boundary

| Layer | Current role |
|---|---|
| Proxmox VM `101` | Full QEMU/KVM virtual machine for complete guest-operating-system administration |
| Proxmox `vmbr1` | Internal-only virtual bridge with no physical uplink |
| OPNsense LAN | Provides the VM's IPv4 gateway, DHCP, DNS forwarding, routed firewall path, and NAT |
| Ubuntu host controls | User/group permissions, OpenSSH configuration, UFW policy, services, sockets, and system logs |

The Ubuntu VM has one intended virtual network adapter on `vmbr1`. It must not receive an additional adapter on `vmbr0`, because that would bypass the documented OPNsense path.

## 4. Implementation and Validation Status

| Validation ID | Control or activity | Current status |
|---|---|---|
| `UBU-VAL-01` | Ubuntu uses `vmbr1` and obtains working IPv4 configuration through OPNsense | **Pass — supported by LAB-02 evidence** |
| `UBU-VAL-02` | Administrative and standard accounts have intentionally different privilege levels | **Pass — separate accounts captured; sudo limited to the administrative role** |
| `UBU-VAL-03` | Current Ubuntu version, kernel, time synchronization, and patch state are recorded | **Pass — evidence captured; two packages deferred by phased rollout** |
| `UBU-VAL-04` | OpenSSH is enabled, active, listening as expected, and reviewed using effective settings | **Pass — socket activation, hardened effective policy, key-only login, and password rejection validated** |
| `UBU-VAL-05` | UFW is active with documented defaults and explicit rules | **Configured — validation pending** |
| `UBU-VAL-06` | Running services, listening sockets, and relevant authentication logs are reviewed | **Not yet performed** |
| `UBU-VAL-07` | UFW allows the approved service and blocks a controlled unapproved service from an independent lab endpoint | **Not yet performed** |
| `UBU-VAL-08` | A labeled clean snapshot is created and a controlled rollback is functionally verified | **Not yet performed** |

## 5. Evidence

The evidence inventory, capture commands, draft captions, sanitization rules, and publication gates are maintained in [`evidence/README.md`](evidence/README.md).

Existing `LAB-02` artifacts are referenced for Ubuntu's DHCP lease and routed IPv4 egress. LAB-03 will not duplicate those files; its own evidence will focus on the guest operating system and host controls.

## 6. Current Limitations

- Two audit-library upgrades were pending at capture time because Ubuntu deliberately deferred them through its phased rollout; no reboot was pending.
- Running services, listening ports, and authentication logs have not yet been documented.
- UFW has not yet been tested from an independent lab endpoint.
- A clean snapshot and rollback test have not yet been completed.
- A Proxmox snapshot will not be described as an independent backup.

## 7. Publication Boundary

Until the remaining validation is complete, the accurate portfolio statement is:

> Deployed an Ubuntu Server VM on the isolated Proxmox lab network, separated administrative and standard user privileges, installed updates, hardened OpenSSH for key-only administrative access, and enabled UFW. Guest-level service, firewall, log, and recovery validation remains in progress.

The project remains **In progress / Drafting** until the evidence pack passes its publication checklist.

## 8. Next Actions

1. Capture UFW, service, and log evidence.
2. Review the results and correct any unexpected exposure before calling the baseline verified.
3. Validate UFW from an independent source on `vmbr1`.
4. Create a labeled snapshot, make one controlled change, roll back, and verify service health.
5. Complete sanitization and cross-document consistency review before publication.

## 9. Related Documentation

- [LAB-03 evidence plan](evidence/README.md)
- [LAB-02 OPNsense evidence](../02-opnsense-segmentation/evidence/)
- [Lab architecture](../../docs/architecture.md)
- [Security boundaries](../../docs/security-boundaries.md)
- [Lessons learned](../../docs/lessons-learned.md)
- [Project roadmap](../../ROADMAP.md)

## 10. Change Log

| Date | Change |
|---|---|
| 2026-09-09 | Hardened OpenSSH for public-key-only administrative access, denied root login, reduced authentication attempts, disabled forwarding features, completed lockout-safe positive and negative client tests, and added the reviewed consolidated SSH artifact. |
| 2026-09-09 | Confirmed Ubuntu's systemd SSH socket activation and successfully tested an Ed25519 key-only administrative login; retained SSH hardening and final evidence recapture as open work. |
| 2026-09-08 | Added reviewed VM-hardware, Ubuntu platform/update, and account-separation evidence; recorded the completed reboot and phased deferral of two audit-library packages; created a standard account and verified that sudo remains limited to the administrative role. |
| 2026-09-07 | Created the LAB-03 drafting outline and separated completed setup activities from pending guest, firewall, log, and recovery validation. |
