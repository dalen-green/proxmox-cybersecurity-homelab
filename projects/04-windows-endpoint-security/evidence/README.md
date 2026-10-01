# LAB-04 Evidence Pack and Capture Plan

> **Technical status:** In progress\
> **Portfolio status:** Drafting\
> **Evidence available:** Two reviewed screenshots; one complete evidence group and one partial group out of ten planned groups\
> **Last reviewed:** 2026-09-30

## What this pack supports so far

This pack supports the [Windows endpoint project](../README.md). The first screenshot records the created VM's hardware and bridge assignment. The second shows Windows reporting Ethernet Connected with a Public profile and identifies the signed-in account as local.

These are useful starting records. They do not yet prove patch state, account-role separation, Defender protection, host-firewall enforcement, or recovery. The plan below keeps configuration, runtime observations, and functional tests distinct, following the same approach as the Ubuntu evidence pack.

## Artifact index

Only files already present are linked. Filenames for future captures are planning targets, and an evidence group may contain more than one file when a test needs both configuration and results.

| ID | Artifact or planned group | Status | What it supports or needs to show |
|---|---|---|---|
| `WIN-E01` | [01-proxmox-windows-hardware.png](01-proxmox-windows-hardware.png) | **Captured — reviewed** | Created VM `102`, resources, UEFI/TPM configuration, and one VirtIO NIC on `vmbr1` |
| `WIN-E02` | `02-windows-platform-and-updates.txt` and supporting captures | **Planned** | Installed version/build, dated update/restart state, clock synchronization, drivers/guest agent, and Windows Secure Boot/TPM state |
| `WIN-E03` | `03-windows-account-separation.txt` and elevation capture | **Planned** | Administrative/standard memberships, standard-user sign-in, UAC review, and deliberate elevation behavior |
| `WIN-E04` | [04a-windows-network-status.png](04a-windows-network-status.png); planned `04-windows-ipv4-egress.txt` | **Partial — initial screenshot reviewed** | Connected/Public state is visible; Windows address, gateway, DNS, and functional egress still need capture |
| `WIN-E05` | `05-windows-defender-and-firewall.txt` and supporting captures | **Planned** | Defender settings, intelligence-update/scan results, firewall profiles, defaults, and reviewed exceptions |
| `WIN-E06` | `06-windows-services-ports-and-startup.txt` | **Planned** | Running services, TCP/UDP listeners, relevant connections, startup entries, and explanations |
| `WIN-E07` | `07-windows-audit-and-sysmon-events.txt` plus the selected Sysmon configuration | **Planned** | Audit settings, a controlled Windows event, Sysmon configuration provenance, and a matching benign process event |
| `WIN-E08` | `08-windows-firewall-independent-test.txt` and relevant log excerpt | **Planned** | A known listener, independent allowed/blocked comparison, Windows-side attribution, and cleanup |
| `WIN-E09` | `09-proxmox-windows-snapshot.png` | **Planned** | A clearly named snapshot created after baseline validation, with a useful description and RAM-state choice |
| `WIN-E10` | `10a-windows-pre-rollback-marker.txt` and `10-windows-rollback-validation.txt` | **Planned** | Marker recorded outside the VM before rollback, its later absence, and restored access/network/security checks |

## Reviewed captions and limits

### `WIN-E01` — Created VM hardware

> **Caption:** Proxmox hardware view for Windows VM `102`, showing four CPU cores with CPU type `host`, 8 GiB RAM with ballooning disabled, an 80 GiB SCSI disk, OVMF UEFI, an EFI disk with pre-enrolled keys, TPM 2.0, and one VirtIO network adapter on internal bridge `vmbr1`. The virtual MAC address is redacted.

This is the hardware view after VM creation, rather than a wizard preview. The main disk has discard, IO thread, and SSD emulation enabled. The Windows and VirtIO installation discs are attached in this setup-stage capture.

The image establishes configured firmware and TPM devices. Windows runtime checks are still needed before describing Secure Boot or TPM readiness as verified inside the guest. The visible NIC firewall option does not establish an applied or tested Proxmox firewall policy. An attached ISO filename is not proof of the installed Windows build.

### `WIN-E04` — Initial Windows network status

> **Caption:** Windows Settings after the VirtIO driver setup reports Ethernet Connected with a Public network profile. The account area identifies the generic lab account as a Local Account.

The screenshot does not show an IPv4 address, gateway, DNS server, route, or a controlled network test. Those checks remain part of this evidence group. It also does not show whether the local account is an administrator or prove that a separate standard account exists.

## How I plan to capture the remaining evidence

I will collect one group at a time and review its result before marking it complete. Screenshots will keep the relevant setting and context visible; text captures will include the command or a clear label and a timestamp with its time zone. A setup instruction by itself will not be treated as a result.

For account separation, I will record both roles and then test an administrative action from the standard account. For Windows Firewall, I will keep the test listener and network path consistent, compare fresh allowed and blocked connections, and locate the Windows-side log before attributing the result to the host firewall. For logging, I will connect the controlled action to the event's time, ID, and relevant fields.

For recovery, I will save the pre-rollback marker record outside the VM so the rollback does not remove the proof needed for comparison. Post-rollback checks will cover access, networking, and the selected baseline controls. This is a local snapshot test; independent backup restoration remains a separate project requirement.

## Relationship to earlier evidence

- [LAB-01](../../01-proxmox-foundation/evidence/README.md) records the Proxmox foundation and bridges.
- [LAB-02](../../02-opnsense-segmentation/evidence/README.md) records OPNsense networking and the tested Ubuntu-to-upstream IPv4 SSH restriction.
- [LAB-03](../../03-ubuntu-server-baseline/evidence/README.md) provides the model for account, host-firewall, and controlled rollback evidence.

I can reference those records for the shared infrastructure and testing method. Windows still needs its own guest configuration and test results. The existing OPNsense or UFW tests do not establish Windows Firewall behavior or Windows IPv6 protection.

## Sanitization and integrity

The two images are preserved byte-for-byte as supplied. The hardware image already has its virtual MAC address obscured. The Windows Settings image contains no password, recovery answer, email address, MAC address, or upstream address. Its generic `labadmin` role label is retained to explain the local-account observation.

Future public captures will omit passwords, recovery material, product/device identifiers, unnecessary unique host/account identifiers, and protected upstream addresses. Lab-only addresses, timestamps, account roles, event IDs, ports, actions, and policy details should remain where they are needed to explain the result. Raw event exports will be reviewed before publication.

| Artifact | SHA-256 |
|---|---|
| `01-proxmox-windows-hardware.png` | `1bdef411666750a09a4388a54dae980a3fcad0ec4d3e727906f686b90b70549a` |
| `04a-windows-network-status.png` | `464893f108caafc605469adad960b7f244ea64601b34262f4b184420c770f232` |

The collection sessions took place on September 29–30, 2026 (America/New_York). Neither image displays a capture timestamp, so this session context is not presented as an embedded timestamp or a time-correlated test.

## Completion checks

- [x] Review the two current images for legibility, claim scope, and public identifiers.
- [x] Link the present artifacts and label future filenames as planned.
- [ ] Complete `WIN-E02` through `WIN-E10`, including the remaining Windows network checks.
- [ ] Match each validation result in the project README to its supporting evidence.
- [ ] Record temporary test-rule/listener cleanup and recovery results.
- [ ] Check links, captions, integrity hashes, and publication wording before marking the baseline complete.

The project remains **In progress / Drafting** while these checks are open.
