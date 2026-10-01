# Project 04: Windows 11 Endpoint Security

> **Technical status:** In progress\
> **Portfolio status:** Drafting\
> **Platform:** Windows 11 Enterprise VM on Proxmox VE\
> **Last updated:** 2026-09-30

## What I set out to learn

After finishing the Ubuntu baseline, I wanted to practice the same ideas on a Windows workstation: separate everyday access from administration, maintain the operating system, understand the services it exposes, review its security controls, and recover from a controlled change.

This is my first Windows endpoint project in the lab. I have installed Windows, reached the desktop with a local account, and brought up Ethernet after working through the VirtIO drivers. The hardening and validation work is still underway. A working desktop gives me a starting point; I still need to show what protects it and test those settings.

This project follows [LAB-04 in the roadmap](../../ROADMAP.md#lab-04--windows-11-endpoint-security). The initial baseline uses local accounts. Joining a domain belongs to LAB-05, and forwarding logs to Wazuh belongs to LAB-06. Those later projects will build on this endpoint.

## Where Windows sits in the lab

| Setting | Recorded setup |
|---|---|
| Proxmox VM | `102`, named `windows-11-enterprise` |
| Processor | 4 virtual cores, one socket, CPU type `host` |
| Memory | 8 GiB; ballooning disabled |
| Main disk | 80 GiB on `local-lvm`, using VirtIO SCSI single |
| Disk options | Discard, IO thread, and SSD emulation enabled |
| Firmware | OVMF UEFI with a Q35 machine |
| EFI and TPM | EFI disk with pre-enrolled keys; virtual TPM 2.0 |
| Network | One VirtIO adapter on `vmbr1`; NIC firewall option enabled |
| Guest setup | Windows 11 Enterprise; exact installed version/build and update state still need a baseline capture |
| Current access | Proxmox VM console and a local setup account |

The [hardware screenshot](evidence/01-proxmox-windows-hardware.png) shows the created VM. Windows shares the internal bridge with Ubuntu and the OPNsense LAN interface. The intended routed path is Windows → `vmbr1` → OPNsense → `vmbr0` → the home router. The [architecture diagram](../../docs/architecture.md#how-the-network-fits-together) shows those connections.

The [Windows network screen](evidence/04a-windows-network-status.png) reports Ethernet **Connected** and a **Public** network profile. I still need to capture Windows' own IPv4 address, gateway, DNS settings, and a controlled DNS/HTTPS test. Ubuntu's earlier network results are useful background, but they do not prove the Windows configuration.

Guests on the same subnet can communicate directly through `vmbr1`. That is why reviewing Windows Firewall matters even though OPNsense already protects the routed path. The Proxmox NIC firewall checkbox and Windows' Public profile are settings, not evidence that a particular connection has been blocked.

## Progress so far

| Work | What I have observed | What still needs checking |
|---|---|---|
| Create the VM | Reviewed the created hardware, including one NIC on `vmbr1`, 8 GiB RAM, 80 GiB disk, UEFI, and TPM 2.0 | Confirm Secure Boot and TPM state from inside Windows |
| Boot the installer | Setup launched after retrying the DVD boot prompt with the console focused | No further boot troubleshooting is needed for this recorded issue |
| Make the virtual disk visible | Loaded `vioscsi\w11\amd64` from the attached VirtIO ISO; Setup then showed the 80 GB disk | Record the installed driver/device state in the baseline review |
| Install Windows | Completed setup and reached the desktop | Record the exact installed build and complete updates |
| Use a local account | The Settings screenshot identifies `labadmin` as a Local Account | Verify administrator membership, create/confirm the standard account, and test elevation behavior |
| Bring up Ethernet | Worked through the VirtIO guest-tools installer; the later Settings capture reports Ethernet Connected | Verify the guest-agent service and other devices, then capture the Windows network settings and functional tests |

The standard-account instructions have been prepared, but I have not yet recorded the `labuser` account-type result. Account separation therefore remains open. Defender, Windows Firewall policy, audit logging, Sysmon, and snapshot recovery have not yet been validated for this VM.

## The hardening plan

I am following the same pattern as the Ubuntu project: make one understandable change, check its effect, and keep enough evidence to explain the result later.

### 1. Separate everyday use from administration

- [ ] Confirm that the setup account has the intended local administrator membership.
- [ ] Create or confirm `labuser` as a **Standard User** with a separate password.
- [ ] Review local Administrators and Users membership, including the state of built-in accounts.
- [ ] Sign in as the standard user and request an administrative action. Record that it requires administrator credentials or is denied, then cancel the test.
- [ ] Review User Account Control and retain deliberate elevation for administrative changes.

The goal is to use the standard account for normal lab work and the administrative account when a task actually needs it. Merely giving two accounts different names will not establish that separation.

### 2. Establish the platform and maintenance baseline

- [ ] Record the Windows edition, version, build, capture time, time zone, and time synchronization.
- [ ] Install operating-system and security updates, complete required restarts, and record the resulting state.
- [ ] Review Device Manager and confirm the needed VirtIO drivers and QEMU Guest Agent service.
- [ ] Check Secure Boot and TPM readiness inside Windows, alongside the Proxmox configuration.
- [ ] Capture the Windows IPv4 address, gateway, DNS settings, and a successful DNS/HTTPS check through the intended lab path.

This will give me a dated maintenance record that I can compare with later checks. The installed version will come from Windows itself rather than the ISO filename.

### 3. Review endpoint protections and exposed services

- [ ] Review Microsoft Defender's real-time protection, security-intelligence updates, cloud-delivered protection, and tamper protection where available.
- [ ] Run a quick scan and record its result and time.
- [ ] Review Windows Firewall state and policy for the Domain, Private, and Public profiles, including the active profile and existing exceptions.
- [ ] Review remote-access and sharing settings; document any service that needs to be reachable and limit access to the required sources.
- [ ] Inspect running services, TCP/UDP listeners, active connections, and startup applications. Explain the expected entries before changing anything.

An enabled firewall or a clean scan will be evidence for that specific check. I will still need the connection test below to show how a selected firewall rule behaves.

### 4. Make selected activity visible in logs

- [ ] Review the audit settings needed for a selected authentication or account-management event.
- [ ] Generate a harmless event and locate it in Event Viewer, retaining its event ID, time, and relevant fields.
- [ ] Install Sysmon from Microsoft's official distribution with a small, documented configuration suited to this exercise.
- [ ] Save the configuration and its source/version or hash so the collected events can be explained and repeated.
- [ ] Generate a benign process event and find the corresponding Sysmon record, then note which channels should be collected by Wazuh in LAB-06.

I want to understand why an event appeared and what it tells me before adding centralized monitoring. Installing Sysmon alone will not count as a completed logging test.

### 5. Test Windows Firewall from another lab system

- [ ] Use an independent lab endpoint, such as Ubuntu, and record both systems' current IPv4 addresses.
- [ ] Establish a known test listener on Windows and confirm the connection with a narrowly scoped temporary allow rule.
- [ ] Remove that exception and repeat a fresh connection to the same address and port while the listener remains available.
- [ ] Compare the client result with Windows Firewall logging to identify the host firewall's decision.
- [ ] Remove the temporary listener and any test rule, then record the final policy.

Keeping the listener and network path consistent should help me distinguish a firewall block from an unavailable service. This will validate only the selected IPv4 flow. IPv6 and the wider management boundary remain separate checks in the [security-boundary notes](../../docs/security-boundaries.md).

### 6. Create and test a recovery point

- [ ] After the baseline checks pass, create a clearly named Proxmox snapshot and record what state it represents.
- [ ] Create a harmless post-snapshot marker and record its contents or hash outside the VM before rollback.
- [ ] Roll back in a controlled session and verify the marker is absent.
- [ ] Recheck sign-in, networking, the selected security settings, and logging after recovery.

This follows the controlled-marker approach used in LAB-03. The snapshot will still depend on Proxmox's local storage; an independent backup and restore remains a separate workstream.

## Validation results and acceptance criteria

| Check | Current result | Evidence needed to complete it |
|---|---|---|
| `WIN-VAL-01`: Intended VM hardware and bridge | **Configuration reviewed** | `WIN-E01` records the created VM; Windows runtime firmware checks belong to `WIN-E02` |
| `WIN-VAL-02`: Maintained Windows platform | **Pending** | Installed version, updates/restarts, synchronized time, drivers, Secure Boot, and TPM state |
| `WIN-VAL-03`: Deliberate administrative elevation | **Pending** | Account memberships, standard-user sign-in, and the elevation result |
| `WIN-VAL-04`: Windows uses the intended network path | **Partial** | Connected/Public screen captured; address, route, DNS, and functional egress tests remain |
| `WIN-VAL-05`: Endpoint protections and firewall policy | **Pending** | Defender state/scan and reviewed firewall profiles and rules |
| `WIN-VAL-06`: Expected services and listeners | **Pending** | Explained inventory of services, ports, connections, and startup items |
| `WIN-VAL-07`: Selected events can be found | **Pending** | Audit policy, a controlled Windows event, Sysmon configuration, and a matching process event |
| `WIN-VAL-08`: Independent host-firewall enforcement | **Pending** | Allowed/blocked comparison with Windows-side logging and cleanup |
| `WIN-VAL-09`: Baseline recovery works | **Pending** | Labeled snapshot, pre-change marker record, and post-rollback checks |

The initial endpoint baseline will be complete when these checks are recorded and reviewed. The [evidence index](evidence/README.md) maps each check to its planned artifacts. Domain joining and centralized log forwarding stay with LAB-05 and LAB-06.

## Problems I worked through

| What I saw | What I checked or changed | What I learned |
|---|---|---|
| DVD boot timed out and the VM tried PXE network boot | Reset the VM, focused the console, and responded to the DVD prompt; Windows Setup launched | The installation disc was reachable. A missed boot prompt could explain the fallback without changing the VM's network |
| Windows Setup showed no installation disk | Loaded the Windows 11 x64 VirtIO SCSI driver; the 80 GB disk appeared | A disk can exist in Proxmox while the guest still needs its controller driver |
| Initial setup offered no working network connection | Used the displayed offline setup option, created a local account, and worked through the VirtIO guest-tools installation; Ethernet was later shown as Connected | Storage and networking use different drivers, and a missing guest driver is worth checking before changing OPNsense |

## Skills I am practicing

So far, I can explain how I configured the Windows VM, checked its network attachment, recovered from a missed installation boot prompt, loaded a storage driver, and completed local-account setup. The next part is learning how to prove account separation, patch state, endpoint protection, logging, and recovery instead of relying on the defaults.

This workstation will later support the domain exercises and the simulated clinical laboratory. It currently represents a general Windows lab endpoint, with no clinical software or patient data.

## Related notes

- [Evidence index and capture plan](evidence/README.md)
- [Roadmap](../../ROADMAP.md)
- [Lab architecture](../../docs/architecture.md)
- [Security boundaries](../../docs/security-boundaries.md)
- [Lessons learned](../../docs/lessons-learned.md)
- [Ubuntu baseline and recovery example](../03-ubuntu-server-baseline/README.md)

## Change log

| Date | Change |
|---|---|
| 2026-09-30 | Drafted the Windows progress record and hardening plan, added the reviewed hardware and initial network captures, and kept remaining validation open. |
