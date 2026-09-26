# Project 03: Ubuntu Server Security Baseline

> **Technical status:** In progress\
> **Portfolio status:** Drafting; 6 of 9 core evidence artifacts reviewed\
> **Platform:** Ubuntu Server VM on Proxmox VE\
> **Last reviewed:** 2026-09-26

## What I am trying to learn

This is my first hands-on Linux server project. I am learning how to manage accounts, install updates, connect remotely, review services, and decide which incoming connections the server should accept.

A **security baseline** is the starting configuration I want to understand and be able to return to after an experiment. I have completed several setup steps, but the overall baseline is still in progress because I have not finished the remaining privilege review, independent firewall testing, or snapshot recovery exercise.

## Where Ubuntu sits in the lab

| Component | Recorded role or setting |
|---|---|
| Proxmox VM | `101`, with 2 virtual CPU cores and a 40 GiB main disk |
| Memory display | The hardware screenshot shows `2.00 GiB / 4.00 GiB`; a configuration review is still needed to explain the memory settings fully |
| Network adapter | One VirtIO adapter on `vmbr1` |
| Gateway and DNS | OPNsense LAN at `10.10.10.1` |
| Guest platform | Ubuntu 26.04.1 LTS, recorded in the September 8 platform output |
| Host controls | Accounts and permissions, OpenSSH, UFW, services, and local logs |

`vmbr1` is the internal lab switch. Ubuntu uses OPNsense for normal traffic to other networks. The [LAB-02 evidence](../02-opnsense-segmentation/evidence/) already records its DHCP lease and working IPv4 egress, so I reference those files here.

For workstation SSH access, the documented setup uses local forwarding through Proxmox with a temporary lab-side address of `10.10.10.2`. Ubuntu sees that Proxmox address as the source. This connection stays on the lab subnet and is separate from the routed outbound SSH connection blocked in LAB-02.

The [architecture](../../docs/architecture.md) and [security boundaries](../../docs/security-boundaries.md) explain that management exception. Earlier cleanup is recorded for LAB-02, but removal after the latest Ubuntu session has not been evidenced.

## What I have done so far

### Created separate accounts

I created an administrative account and a standard account. The saved group output shows the administrative account in the `sudo` group and the standard account outside it. `sudo` is how an authorized user deliberately runs a command with elevated privileges.

This gave me a first look at separating routine access from administration. It is a group-membership check, not a complete review of every permission. The administrative account also has other groups, including `lxd`, which should be considered during the remaining privilege and service review.

### Installed updates and recorded what remained

The September 8 evidence records Ubuntu 26.04.1 LTS, kernel `7.0.0-31-generic`, synchronized UTC time, and no pending reboot requirement. Two audit-library packages were deferred by Ubuntu's phased rollout.

I kept those pending packages in the record so the result is accurate. This describes the system at capture time; it is not a claim that the server has no pending updates today.

### Configured and hardened SSH

SSH lets me open an encrypted terminal session from another computer. I enrolled an Ed25519 public key for the administrative account before tightening the server settings.

The [SSH artifact](evidence/04-ubuntu-services-and-ssh.txt) records valid syntax, active listening sockets, public-key-only authentication for the administrative role, disabled password and keyboard-interactive authentication, denied root login, and disabled forwarding features on Ubuntu's SSH server.

The setup notes record keeping an existing session open while applying the changes, then successfully testing a fresh key-only login and rejected password attempts. Those client screenshots were not published because they contained identifiers. The committed text preserves the resulting server configuration and runtime state.

One confusing result was that `ssh.service` showed disabled for direct startup but active, while `ssh.socket` was enabled and active. I learned that socket activation can provide the listener and activate the SSH service. A single enabled/disabled line was not enough to explain the service's behavior.

### Enabled UFW with a specific management source

UFW is the firewall on Ubuntu itself. The September 25 [status output](evidence/05-ubuntu-ufw-status.txt) shows it active with low-volume logging, default-deny incoming policy, allowed outgoing traffic, and routed traffic reported as disabled.

Its explicit SSH rule allows TCP/22 from `10.10.10.2`, the temporary Proxmox management source. The setup notes record a fresh successful SSH connection after activation. The saved artifact shows the policy; I still need independent blocked-traffic evidence.

The source restriction matters: another lab VM with a different address should not automatically be able to SSH into Ubuntu. The planned test needs to distinguish allowed management access from denied access by an independent source.

### Reviewed running services, listeners, and one authentication event

The September 26 evidence records 20 running service units. I reviewed that inventory alongside a separate TCP/UDP listener diagnostic instead of disabling unfamiliar services simply because of their names. SSH was the only remotely listening server service. Local DNS and chrony listeners were bound to loopback, and the DHCP client listener on `ens18` was expected.

Optional-looking units such as ModemManager, multipathd, udisks2, and upower did not expose listening network ports, so I left them unchanged pending a dependency-based review. A fresh SSH event also confirmed successful Ed25519 public-key authentication from the authorized `10.10.10.2` management source. The publication copy keeps the event meaning and timestamp while replacing the hostname, account, process ID, ephemeral source port, and key fingerprint. Remaining group and effective-permission review is still open.

## What is checked and what is still open

| Validation | Recorded status |
|---|---|
| `UBU-VAL-01`: Ubuntu uses the lab IPv4 network | Supported by LAB-02 DHCP/egress evidence and LAB-03 VM hardware |
| `UBU-VAL-02`: Account separation | Separate accounts and sudo-group membership captured; full effective-permission review remains |
| `UBU-VAL-03`: Platform and update state | Captured September 8, including two phased deferrals |
| `UBU-VAL-04`: SSH configuration and access | Effective settings/runtime captured; key-login and password-rejection tests recorded in setup notes |
| `UBU-VAL-05`: Active UFW policy | Captured September 25; fresh allowed SSH connection recorded in setup notes |
| `UBU-VAL-06`: Running services, listening ports, and authentication-log review | Pass — service inventory and sanitized SSH event published; the all-listener diagnostic was reviewed during collection |
| `UBU-VAL-07`: Independent UFW allow/deny validation | Still needed |
| `UBU-VAL-08`: Snapshot and controlled rollback | Still needed |

The [evidence index](evidence/README.md) tracks the six captured artifacts and the three still needed. It also preserves the collection instructions and explains which tests have only been recorded in the setup notes.

## What the two firewalls taught me

OPNsense can filter traffic when it crosses the routed boundary between the lab and upstream networks. UFW can filter traffic at Ubuntu, including traffic from another guest on the same lab subnet. Those local guest-to-guest connections normally bypass OPNsense.

I also learned to separate a listening service from a reachable service. SSH can listen on an address while a firewall limits who can connect. The recorded SSH listener includes IPv6, so IPv6 review remains necessary; the IPv4 UFW rule does not settle that question.

The Proxmox hardware view has its NIC firewall checkbox enabled. That is a setting shown in the screenshot, but I have not demonstrated a separate Proxmox firewall policy from it.

## My next steps

1. Review remaining group memberships and effective permissions, including whether optional administrative groups are actually required.
2. Test a fresh allowed SSH connection from the documented management source and denied traffic from an independent lab endpoint. Use a temporary listening service to make the blocked-port test meaningful, then remove it.
3. Confirm cleanup of the temporary management address and tunnel after use.
4. Create a labeled clean snapshot after the baseline review, make one harmless change, roll back, and check networking, SSH, and UFW again.
5. Complete the evidence review before marking the whole project verified.

The snapshot exercise will demonstrate rollback. An independent backup and restore remains separate future work.

## What I can describe at this stage

I have deployed a Linux server, separated two account roles, recorded its update state, tightened SSH authentication, enabled a source-restricted host firewall, and reviewed its running services, network listeners, and one controlled authentication event. I am still learning to explain how those settings interact and to support them with repeatable tests.

The project remains **In progress / Drafting**. Its remaining tests and broader network limits are part of the result, not reasons to mark completed setup steps as unfinished.

## Related notes

- [Evidence and collection plan](evidence/README.md)
- [OPNsense project](../02-opnsense-segmentation/)
- [Lessons learned](../../docs/lessons-learned.md)
- [Lab architecture](../../docs/architecture.md)
- [Security boundaries](../../docs/security-boundaries.md)
- [Roadmap](../../ROADMAP.md)

## Change log

| Date | Change |
|---|---|
| 2026-09-26 | Added the sanitized running-service and SSH authentication artifact, documented the all-listener review, and advanced the evidence count to six of nine while leaving privilege, independent-firewall, and recovery validation open. |
| 2026-09-25 | Rewrote the project in a first-project voice; aligned claims with the five captured artifacts and clarified temporary access, memory display, and remaining tests. |
| 2026-09-25 | Captured active UFW policy with SSH restricted to the temporary Proxmox source; recorded a fresh successful SSH connection after activation. |
| 2026-09-09 | Hardened SSH, reviewed effective settings and socket activation, and recorded key-login and password-rejection tests. |
| 2026-09-08 | Added VM hardware, platform/update, and account-separation evidence. |
| 2026-09-07 | Created the Ubuntu project outline and evidence plan. |
