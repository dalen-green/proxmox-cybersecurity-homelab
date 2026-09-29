# Project 03: Ubuntu Server Security Baseline

> **Technical status:** Verified for the documented IPv4 baseline\
> **Portfolio status:** Review ready; 9 of 9 core evidence artifacts reviewed\
> **Platform:** Ubuntu Server VM on Proxmox VE\
> **Last reviewed:** 2026-09-29

## What I set out to learn

This is my first hands-on Linux server project. I used it to learn how to separate account roles, maintain an operating system, configure remote administration, review exposed services, apply a host firewall, and recover a VM after a controlled change.

A **security baseline** is a known starting state that I can describe, test, and return to. All nine planned artifacts are now reviewed. “Verified” applies only to the controls and IPv4 tests documented here; it is not a claim that the server is permanently secure or that every network path has been tested.

## Where Ubuntu sits in the lab

| Component | Recorded role or setting |
|---|---|
| Proxmox VM | VM `101`, with 2 virtual CPU cores and a 40 GiB main disk |
| Memory display | The hardware screenshot shows `2.00 GiB / 4.00 GiB`; I have not used it to claim one fixed memory value |
| Network adapter | One VirtIO adapter on internal bridge `vmbr1` |
| Guest address | `10.10.10.149/24`, assigned by OPNsense DHCP at the recorded checks |
| Gateway and DNS | OPNsense LAN at `10.10.10.1` |
| Guest platform | Ubuntu 26.04.1 LTS on kernel `7.0.0-34-generic` in the final platform capture |
| Host controls | Separate account roles, hardened OpenSSH, UFW, service review, and local logs |

`vmbr1` acts as the internal virtual switch. Ubuntu uses OPNsense for traffic leaving that subnet. The [LAB-02 evidence](../02-opnsense-segmentation/evidence/) records the DHCP lease, IPv4 egress, and a routed firewall test, so those results are referenced instead of duplicated.

During administration, I temporarily gave Proxmox the lab-side address `10.10.10.2/24` and forwarded a workstation SSH connection through it. Ubuntu therefore saw `10.10.10.2` as the client. After the final evidence transfer, I stopped the tunnel, removed that runtime address, and confirmed that `vmbr1` no longer displayed an IPv4 address. The UFW rule remains source-restricted to `10.10.10.2`, so the management path only works when that address and tunnel are deliberately restored.

## Baseline work completed

### Separated administrative and standard accounts

The final [account artifact](evidence/03-ubuntu-account-separation.txt) records two interactive Bash accounts with stable role labels. The administrative role has unrestricted `sudo` authorization, while the standard role is explicitly denied `sudo` access.

I also checked the administrative groups and removed its unnecessary `lxd` membership after confirming that LXD was not installed. The final group list no longer contains `lxd`. This completes the intended local sudo-role review, but it is not an exhaustive audit of every file permission, Linux capability, or application-specific authorization.

### Installed approved updates and recorded phased deferrals

The final [platform artifact](evidence/02-ubuntu-platform-and-updates.txt) records a successful package-metadata refresh, Ubuntu 26.04.1 LTS, kernel `7.0.0-34-generic`, `amd64` architecture, synchronized UTC time, and no pending reboot.

Two release-upgrader packages remained deferred by Ubuntu's phased rollout: `python3-distupgrade` and `ubuntu-release-upgrader-core`. A simulated upgrade reported that no eligible packages would be installed at that time. I did not force the phased packages merely to make the count display zero.

### Hardened and validated SSH

The [SSH artifact](evidence/04-ubuntu-services-and-ssh.txt) records valid OpenSSH syntax, active socket activation, TCP/22 listeners, public-key-only authentication for the administrative role, denied root login, reduced authentication attempts, and disabled password, keyboard-interactive, agent, TCP, stream-local, and X11 forwarding paths.

The service initially looked inconsistent because `ssh.service` was disabled for direct startup while active. Reviewing `ssh.socket` showed that systemd socket activation was enabled and holding the listener. A fresh key-only login succeeded after hardening, while password-based attempts were rejected.

### Applied and independently tested UFW

The [UFW policy artifact](evidence/05-ubuntu-ufw-status.txt) records:

- Active firewall status and low-volume logging
- Default-deny incoming policy
- Allowed outgoing traffic
- Disabled routed traffic
- TCP/22 permitted only from `10.10.10.2`

The [independent test artifact](evidence/07-ubuntu-ufw-independent-test.txt) then compares two sources against the same listening SSH service. Ubuntu accepted public-key SSH from permitted source `10.10.10.2` and logged repeated UFW blocks for TCP/22 from independent source `10.10.10.1`. Because both tests addressed the existing SSH listener, an extra temporary service or firewall rule was unnecessary.

### Reviewed services, listeners, and an authentication event

The [service and authentication artifact](evidence/06-ubuntu-services-and-auth-log.txt) records 20 running service units and one sanitized successful public-key login. A separate all-listener diagnostic was reviewed during collection.

SSH was the only remotely listening server service. DNS and chrony listeners were loopback-only, and the DHCP client listener on `ens18` was expected. Optional-looking services such as ModemManager, multipathd, udisks2, and upower did not expose listening network ports, so I did not disable them solely because their names were unfamiliar.

### Created and tested a recovery point

The [snapshot artifact](evidence/08-proxmox-ubuntu-snapshot.png) shows the labeled `ubuntu-baseline-2026-09-29` snapshot for VM `101`, created without RAM state after the reviewed baseline work.

Afterward, I created a harmless marker file and recorded its checksum in the [pre-rollback record](evidence/09a-ubuntu-pre-rollback-marker.txt). I powered off the VM, rolled it back to the named snapshot, started it, and logged in again. The [post-rollback artifact](evidence/09-ubuntu-rollback-validation.txt) shows that the marker was absent while DHCP networking, the default route, SSH, and the expected UFW policy were restored.

This demonstrates rollback to the recorded VM state. The snapshot remains on the same Proxmox storage and is **not** an independent backup.

## Validation results

| Validation | Result |
|---|---|
| `UBU-VAL-01`: Ubuntu uses the intended lab IPv4 network | **Pass** — VM hardware plus LAB-02 DHCP and egress evidence |
| `UBU-VAL-02`: Administrative and standard roles are separated | **Pass** — only the administrative role has effective sudo authorization; unnecessary `lxd` membership was removed |
| `UBU-VAL-03`: Platform and update state are documented | **Pass** — final September 29 capture records refreshed metadata, phased deferrals, and no reboot requirement |
| `UBU-VAL-04`: SSH configuration and access are hardened | **Pass** — effective settings, listeners, socket activation, and controlled login tests reviewed |
| `UBU-VAL-05`: UFW policy is active and source-restricted | **Pass** — defaults, logging, and the TCP/22 source rule are captured |
| `UBU-VAL-06`: Running services, listeners, and authentication logs are reviewed | **Pass** — service inventory, listener diagnostic, and sanitized login event reviewed |
| `UBU-VAL-07`: UFW distinguishes permitted and independent sources | **Pass for the recorded IPv4 TCP/22 test** — allowed management login and blocked independent-source SYN packets recorded |
| `UBU-VAL-08`: A controlled snapshot rollback restores the baseline | **Pass** — pre-change marker proof plus post-rollback network, SSH, and UFW checks |

The [evidence index](evidence/README.md) provides the final captions, sanitization notes, hashes, and limitations for all nine core artifacts and the supporting pre-rollback record.

## What the two firewalls taught me

OPNsense filters traffic when it crosses a routed boundary. UFW filters traffic at Ubuntu itself, including direct traffic from another system on the same `vmbr1` subnet. That same-subnet traffic does not normally traverse OPNsense's routed LAN rules.

I also learned to separate a **listening service** from a **reachable service**. SSH can listen on TCP/22 while UFW limits which source may complete a connection. The independent test used one known listener so that the different results could be attributed to the source-specific UFW policy.

The Proxmox hardware view shows its NIC firewall checkbox enabled, but I have not demonstrated a separate Proxmox firewall policy. Ubuntu's SSH artifact also shows an IPv6 listener, while this project's firewall validation is IPv4-only.

## Limits and future improvements

- Review IPv6 addressing and UFW behavior, then enforce an equivalent policy or document a deliberate disabled configuration.
- Create an independent backup on separate storage and perform a restore test; the Proxmox snapshot does not satisfy that requirement.
- Test VM startup order and service recovery after a full host restart.
- Revisit optional services only after checking dependencies and actual system use.
- Turn the individual evidence commands into a small repeatable baseline-audit script.
- Repeat update and exposure checks over time because these artifacts describe specific capture dates, not continuous compliance.

## What I can now explain

I can explain how I deployed a Linux VM on an isolated virtual network, separated routine and administrative access, maintained its packages, hardened key-based SSH, applied a source-specific host firewall, reviewed active services and listeners, validated an allowed and denied connection, and proved that a controlled snapshot rollback restored the expected state.

The project is **Verified for the documented IPv4 baseline / Review ready**. IPv6, broader management-boundary testing, independent backup restoration, and continuous monitoring remain outside this completed scope.

## Related notes

- [Evidence pack](evidence/README.md)
- [OPNsense project](../02-opnsense-segmentation/)
- [Lessons learned](../../docs/lessons-learned.md)
- [Lab architecture](../../docs/architecture.md)
- [Security boundaries](../../docs/security-boundaries.md)
- [Roadmap](../../ROADMAP.md)

## Change log

| Date | Change |
|---|---|
| 2026-09-29 | Completed the local sudo-role review, independent IPv4 UFW test, temporary-path cleanup, labeled snapshot, and controlled rollback; advanced the pack to nine of nine reviewed core artifacts. |
| 2026-09-26 | Added the sanitized running-service and SSH authentication artifact and completed `UBU-VAL-06`. |
| 2026-09-25 | Captured active UFW policy with SSH restricted to the temporary Proxmox source. |
| 2026-09-09 | Hardened SSH, reviewed socket activation, and recorded key-login and password-rejection tests. |
| 2026-09-08 | Added VM hardware, platform/update, and account-separation evidence. |
| 2026-09-07 | Created the Ubuntu project outline and evidence plan. |
