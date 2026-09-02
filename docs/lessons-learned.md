# Lessons Learned

> **Document status:** Living document  
> **Last updated:** 2026-09-02  
> **Lab phase:** Proxmox foundation, IPv4 network segmentation, and Ubuntu Server baseline  
> **Publication status:** Sanitized for a public portfolio

## 1. Purpose

This document records technical discoveries, configuration mistakes, troubleshooting methods, validation results, and design improvements identified while building the Proxmox cybersecurity home lab.

The objective is not to present the lab as a flawless installation. It is to demonstrate a repeatable professional process:

1. Define the expected result.
2. Observe the actual result.
3. Collect relevant evidence.
4. Identify the cause.
5. Apply a controlled correction.
6. Retest the behavior.
7. Record the lesson and any remaining work.

Current topology and system roles are documented in [architecture.md](architecture.md). Security requirements and remaining isolation controls are documented in [security-boundaries.md](security-boundaries.md).

## 2. Status Definitions

| Status | Meaning |
|---|---|
| **Resolved** | The immediate problem was corrected and normal operation was restored |
| **Validated** | The correction or security behavior was confirmed through a repeatable test |
| **In use** | The lesson has become an ongoing design or documentation practice |
| **Open improvement** | The issue is understood, but additional implementation or testing remains |
| **Design decision** | The lesson informed an architectural choice rather than correcting a failure |

## 3. Lessons Summary

| ID | Subject | Category | Status |
|---|---|---|---|
| `LL-01` | A bridge name does not define its security role | Proxmox networking | **Validated** |
| `LL-02` | OPNsense interfaces must be mapped to the correct virtual adapters | Firewall deployment | **Validated** |
| `LL-03` | DHCP-assigned WAN addresses can change | IP addressing | **Resolved** |
| `LL-04` | Virtual disk capacity and physical RAM are different resources | Resource management | **Resolved** |
| `LL-05` | A checksum must belong to the exact file being verified | File integrity | **Validated** |
| `LL-06` | Browser fake paths and cloud placeholders are not usable file paths | File handling | **Resolved** |
| `LL-07` | Firewall rule order can change the effective policy | Firewall administration | **Validated** |
| `LL-08` | Client source ports and service destination ports serve different purposes | TCP/IP | **Validated** |
| `LL-09` | A configured rule becomes defensible evidence only after testing and log review | Control validation | **In use** |
| `LL-10` | Closing an SSH client does not stop the SSH server | Linux administration | **Resolved** |
| `LL-11` | One successful denial test does not prove complete network isolation | Security assurance | **Open improvement** |
| `LL-12` | OPNsense does not normally inspect same-subnet traffic | Network segmentation | **Open improvement** |
| `LL-13` | IPv4 validation does not automatically validate IPv6 | Dual-stack security | **Open improvement** |
| `LL-14` | VMs and LXC containers provide different isolation guarantees | Virtualization | **Design decision** |
| `LL-15` | Thin provisioning improves flexibility but still requires capacity monitoring | Storage management | **In use** |
| `LL-16` | A snapshot is not an independent backup | Recovery planning | **Open improvement** |
| `LL-17` | Public documentation must separate verified work from planned work | Portfolio practice | **In use** |
| `LL-18` | Repository path names require exact review | Version control | **Open improvement** |

## 4. Detailed Lessons

### `LL-01` — A bridge name does not define its security role

**Category:** Proxmox networking  
**Status:** Validated

#### Situation

The relationship among `vmbr0`, `vmbr1`, the physical Ethernet interface, OPNsense, and the lab systems was initially difficult to visualize. A network diagram also required correction so that the bridges appeared in their proper locations.

#### Cause

Names such as `vmbr0` and `vmbr1` are identifiers, not built-in security classifications. A bridge's actual role is determined by its physical attachment, guest attachments, addressing, and routing.

#### Resolution and validation

- Confirmed that `vmbr0` is connected to the physical Ethernet interface.
- Confirmed that `vmbr1` has no physical uplink.
- Connected the OPNsense WAN adapter to `vmbr0`.
- Connected the OPNsense LAN adapter and Ubuntu Server to `vmbr1`.
- Verified that Ubuntu received its lab-network configuration from OPNsense.

#### Lesson

Network documentation should show both the bridge name and its physical or virtual attachments. The absence of a physical port on `vmbr1` is what makes it internal-only; the name itself provides no isolation.

#### Continuing practice

Review every new VM's network-adapter assignment before its first boot, especially intentionally vulnerable systems.

---

### `LL-02` — OPNsense interfaces must be mapped to the correct virtual adapters

**Category:** Firewall deployment  
**Status:** Validated

#### Situation

OPNsense required one interface facing the upstream network and another facing the isolated lab. Reversing them could disrupt access or place the lab on the wrong side of the firewall.

#### Cause

Proxmox identifies the adapters as `net0` and `net1`, while OPNsense uses guest operating-system interface names. Interface numbering should not be accepted without checking the associated virtual adapter and MAC address.

#### Resolution and validation

- Verified `net0 → vmbr0 → OPNsense WAN`.
- Verified `net1 → vmbr1 → OPNsense LAN`.
- Confirmed that the WAN received an upstream private address.
- Confirmed that the LAN supplied the isolated lab subnet and related services.

#### Lesson

Interface mapping should be validated from both the hypervisor and guest perspectives. Functional addressing and traffic tests provide stronger evidence than interface names alone.

---

### `LL-03` — DHCP-assigned WAN addresses can change

**Category:** IP addressing  
**Status:** Resolved

#### Situation

The OPNsense WAN address changed, and the previously used management address stopped responding from the administrative workstation.

#### Cause

The home router supplied the OPNsense WAN address through DHCP. A dynamically assigned address can change after lease renewal, interface changes, or a new DHCP transaction.

#### Resolution and validation

The current WAN address was confirmed from the OPNsense console and then used for the appropriate management and connectivity checks.

#### Lesson

A dynamic address should be treated as temporary. Documentation should identify the interface by role rather than relying on one lease value. A DHCP reservation may later provide a predictable management address without manually configuring a conflicting static address.

---

### `LL-04` — Virtual disk capacity and physical RAM are different resources

**Category:** Resource management  
**Status:** Resolved

#### Situation

Increasing the OPNsense virtual disk to 32 GiB was initially interpreted as allocating all 32 GB of the host's physical RAM to the firewall.

#### Cause

Proxmox displays disk capacity and memory in different wizard sections, but both values may use similar-looking gigabyte units.

#### Resolution and validation

The OPNsense VM was configured with:

| Resource | Allocation |
|---|---:|
| Virtual disk | 32 GiB |
| RAM | 4096 MiB |

The virtual disk is stored on `local-lvm`; RAM is allocated from the host's physical memory while the VM runs.

#### Lesson

CPU, memory, disk capacity, and network bandwidth are separate virtual resources. Every VM plan should list them in separate columns to prevent accidental over-allocation.

---

### `LL-05` — A checksum must belong to the exact file being verified

**Category:** File integrity  
**Status:** Validated

#### Situation

Proxmox offered SHA-256 verification while uploading the extracted OPNsense ISO. The downloaded OPNsense package and extracted ISO were related but not byte-for-byte identical files.

#### Cause

Compression changes the byte representation of a file. A checksum published for an `.iso.bz2` archive will not match the checksum calculated from the extracted `.iso`.

#### Resolution and validation

SHA-256 was calculated against the exact extracted ISO selected for upload. The resulting value was supplied to the Proxmox upload process.

#### Lesson

A cryptographic hash verifies only the exact bytes from which it was calculated. File name, version, compression state, and publisher documentation must all match the verification method. A locally calculated upload hash checks transfer integrity; publisher-provided checksums or signatures are still needed when independently verifying authenticity.

---

### `LL-06` — Browser fake paths and cloud placeholders are not usable file paths

**Category:** File handling  
**Status:** Resolved

#### Situation

PowerShell could not find the ISO using the path displayed in the Proxmox browser upload dialog.

#### Cause

The browser displayed a privacy placeholder beginning with `C:\fakepath\` rather than the file's actual Windows location. The ISO was also located in OneDrive, where a cloud-only placeholder might not yet have complete local bytes.

#### Resolution and validation

- Ensured the ISO was retained locally on the Windows system.
- Copied its actual path from File Explorer.
- Used PowerShell's `-LiteralPath` parameter to avoid path-parsing issues.
- Successfully calculated the SHA-256 hash.

#### Lesson

Browser file-selection paths should not be assumed to be real filesystem paths. Synced files should be made locally available before hashing, uploading, or using them with command-line tools.

---

### `LL-07` — Firewall rule order can change the effective policy

**Category:** Firewall administration  
**Status:** Validated

#### Situation

A specific blocking rule needed to operate alongside OPNsense's broader default LAN allow rule.

#### Cause

OPNsense evaluates interface rules from top to bottom. A broad allow rule placed before a narrower deny rule can match the traffic first, preventing the deny rule from being evaluated.

#### Resolution and validation

- Placed the specific SSH blocking rule above the general LAN allow rule.
- Repeated the SSH connection attempt from Ubuntu.
- Confirmed that the connection was denied.
- Located the corresponding block entry in the OPNsense firewall log.

#### Lesson

Firewall behavior depends on rule order as well as source, destination, protocol, and port. Every rule change should include a review of neighboring broader rules and a new functional test.

---

### `LL-08` — Client source ports and service destination ports serve different purposes

**Category:** TCP/IP  
**Status:** Validated

#### Situation

Creating the SSH rule required selecting values for both source and destination ports.

#### Cause

A client normally opens a temporary high-numbered source port. It connects to the well-known destination port on the server, such as TCP 22 for SSH. Selecting source port 22 would usually fail to match normal client traffic.

#### Resolution and validation

The rule left the client source port as `any` and identified SSH by destination port 22. The subsequent denial and firewall log confirmed that the rule matched the intended traffic.

#### Lesson

Firewall rules should describe the actual connection direction. For an outbound client request, the requested service is usually identified by the destination port, not the source port.

---

### `LL-09` — A configured rule becomes defensible evidence only after testing and log review

**Category:** Control validation  
**Status:** In use

#### Situation

The firewall interface showed that the SSH block rule existed, but configuration alone did not prove the rule matched the intended traffic.

#### Resolution and validation

The control was tested from Ubuntu, the observed denial was recorded, and the connection was correlated with an OPNsense firewall-log entry containing the expected source, destination, protocol, and action.

#### Lesson

A screenshot of a setting proves configuration state, not control effectiveness. Strong evidence combines:

1. The intended policy
2. The implemented configuration
3. A controlled test
4. The observed endpoint result
5. Supporting firewall or system logs

This validation pattern will be reused for UFW, Windows Firewall, Group Policy, Wazuh alerts, and recovery testing.

---

### `LL-10` — Closing an SSH client does not stop the SSH server

**Category:** Linux administration  
**Status:** Resolved

#### Situation

The Windows PowerShell window containing an SSH session was closed, creating concern that the Ubuntu server or SSH service had been damaged.

#### Cause

An SSH connection is a client session. Closing the client terminates that session, but the remote Ubuntu VM and its OpenSSH service continue running unless a shutdown or service-stop command was issued.

#### Resolution and validation

A new SSH session could be opened after confirming network reachability and the server's availability.

#### Lesson

The client, network session, remote service, guest operating system, and physical host are separate layers. Troubleshooting should identify which layer actually stopped before changing configuration.

---

### `LL-11` — One successful denial test does not prove complete network isolation

**Category:** Security assurance  
**Status:** Open improvement

#### Situation

The controlled SSH test proved that the selected IPv4 rule denied TCP destination port 22 and logged the result.

#### Limitation

That result does not prove that every other port, protocol, destination, management service, or address family is also blocked. A general LAN allow rule remains relevant to the final policy design.

#### Lesson

Security claims must remain no broader than the test evidence. The accurate current claim is that a representative IPv4 restriction was validated—not that comprehensive lab-to-home isolation has already been proven.

#### Remaining action

- Create or confirm a comprehensive logged block from the lab subnet to protected upstream and management networks.
- Test multiple representative protocols against a designated authorized test host.
- Verify that Proxmox and OPNsense management services are unreachable from the lab.
- Record both denied and permitted control tests.

This work corresponds to controls `SB-06` and `SB-07` in [security-boundaries.md](security-boundaries.md).

---

### `LL-12` — OPNsense does not normally inspect same-subnet traffic

**Category:** Network segmentation  
**Status:** Open improvement

#### Situation

Current and future lab systems attached to `vmbr1` share the same Layer 2 network and IPv4 subnet.

#### Cause

Hosts on the same subnet can resolve one another's MAC addresses and exchange frames through `vmbr1` without sending that traffic to their default gateway. OPNsense is therefore not in the ordinary path between those systems.

#### Lesson

Placing several systems behind a firewall does not automatically isolate those systems from each other. Per-host firewalls, separate subnets, VLANs, or additional routed interfaces are needed when east-west inspection or isolation becomes an exercise requirement.

#### Remaining action

Retain the simple single-subnet design for the current beginner phase, then reevaluate segmentation before deploying multiple vulnerable systems or distinct identity, monitoring, and target zones.

---

### `LL-13` — IPv4 validation does not automatically validate IPv6

**Category:** Dual-stack security  
**Status:** Open improvement

#### Situation

Current OPNsense routing and firewall evidence was collected using IPv4.

#### Cause

IPv4 and IPv6 use separate addressing, neighbor-discovery, routing, and firewall considerations. A correct IPv4 rule does not inherently control an available IPv6 path.

#### Lesson

Every security boundary must account for every enabled address family. Unused IPv6 should be disabled consistently, or equivalent IPv6 routing and firewall controls should be configured and tested.

#### Remaining action

Inspect IPv6 addresses and routes on Proxmox, OPNsense, and Ubuntu; then either validate equivalent isolation or document and implement a deliberate disabled state before introducing vulnerable targets.

---

### `LL-14` — VMs and LXC containers provide different isolation guarantees

**Category:** Virtualization  
**Status:** Design decision

#### Situation

LXC containers were considered as a way to run more systems within the host's 32 GB memory limit.

#### Analysis

LXC containers are efficient because they share the Proxmox host's Linux kernel. Full VMs provide independent guest kernels and stronger isolation but require more memory and storage.

#### Lesson

Resource density is only one design factor. The workload's operating system, kernel requirements, security risk, and learning objective determine the appropriate virtualization type.

#### Decision

- Keep OPNsense, Windows, the first Ubuntu baseline, Kali, Wazuh, and intentionally vulnerable full systems as VMs.
- Use unprivileged LXC containers only for lightweight benign support services.
- Run vulnerable application containers inside a disposable VM rather than directly against the Proxmox host kernel.

---

### `LL-15` — Thin provisioning improves flexibility but still requires capacity monitoring

**Category:** Storage management  
**Status:** In use

#### Situation

The OPNsense virtual disk was expanded to 32 GiB without immediately consuming the same amount of physical storage.

#### Cause

`local-lvm` uses thin provisioning. Virtual disks consume physical blocks as data is written rather than reserving their full advertised capacity immediately.

#### Lesson

Thin provisioning reduces initial storage consumption, but it does not create unlimited space. Several guests can collectively be promised more capacity than the host can physically provide, making storage monitoring and cleanup essential.

#### Continuing practice

Track both guest virtual-disk allocations and actual Proxmox storage usage before adding large Windows, Wazuh, or vulnerable-target disks.

---

### `LL-16` — A snapshot is not an independent backup

**Category:** Recovery planning  
**Status:** Open improvement

#### Situation

Proxmox snapshots are planned before controlled changes and security experiments.

#### Limitation

A snapshot is useful for short-term rollback, but it remains dependent on the same VM storage and Proxmox host. Loss or corruption of that storage can remove both the active state and its snapshots.

#### Lesson

Rollback and disaster recovery are different capabilities:

| Mechanism | Primary purpose | Independent of primary storage? |
|---|---|---|
| Snapshot | Rapidly reverse a controlled change | No |
| Proxmox backup stored elsewhere | Restore after guest or primary-storage loss | Yes, when stored independently |
| Sanitized configuration export | Rebuild an appliance configuration | Only if stored securely elsewhere |

#### Remaining action

Create and test the Ubuntu baseline snapshot first, then complete a separate backup-and-restore project using storage independent of the active VM disk.

---

### `LL-17` — Public documentation must separate verified work from planned work

**Category:** Portfolio practice  
**Status:** In use

#### Situation

The repository includes Windows, Active Directory, Kali, Wazuh, vulnerability testing, and recovery plans that have not yet been implemented.

#### Risk

Listing planned systems alongside operational systems without clear statuses could make the portfolio appear to claim experience that has not been demonstrated.

#### Lesson

Technical implementation status and portfolio-publication status should be tracked separately. A system becomes **Verified** only after testing; a project becomes **Published** only after its sanitized explanation and evidence are committed.

#### Continuing practice

- Use explicit statuses such as **Verified**, **In progress**, and **Planned**.
- Match résumé language to the available evidence.
- Record limitations and unfinished validation alongside accomplishments.
- Update [ROADMAP.md](../ROADMAP.md), architecture, security boundaries, and project write-ups when status changes.

---

### `LL-18` — Repository path names require exact review

**Category:** Version control  
**Status:** Open improvement

#### Situation

The documentation directory was created as `docs.` with a trailing period instead of the conventional `docs` name.

#### Cause

Repository paths are exact strings. A visually small punctuation difference becomes part of every file path and link. Trailing periods can also interact poorly with Windows path normalization and local cloning workflows.

#### Lesson

Repository structure should be reviewed before many files and cross-references depend on it. Names should use predictable lowercase paths, hyphens where needed, and no trailing punctuation or spaces.

#### Remaining action

Move the documentation files from `docs./` to `docs/`, verify relative links, and remove the incorrectly named directory in a dedicated repository-organization change.

## 5. Recurring Troubleshooting Pattern

The work completed so far produced a reusable troubleshooting model:

| Stage | Question | Example from this lab |
|---|---|---|
| Define | What should happen? | SSH from the lab to the designated upstream target should be denied |
| Observe | What actually happened? | The connection result was checked from Ubuntu |
| Localize | Which layer controls the behavior? | OPNsense LAN rule order and traffic matching |
| Inspect | What evidence is available? | Guest command output and OPNsense firewall logs |
| Correct | What is the smallest controlled change? | Move the specific deny rule above the general allow rule |
| Retest | Did behavior change as expected? | Repeat the SSH attempt and inspect the new log entry |
| Document | What should be retained? | Rule reasoning, test method, result, evidence, and remaining limitations |

This model avoids making multiple untracked changes at once and makes the final project evidence easier to explain during an interview.

## 6. Open Improvements Derived From These Lessons

- [ ] Rename the repository directory from `docs.` to `docs` and verify all links.
- [ ] Implement or confirm a comprehensive IPv4 block from the lab network to protected home and management networks.
- [ ] Verify that Proxmox and OPNsense management services cannot be reached from an unauthorized lab endpoint.
- [ ] Validate IPv6 isolation or implement a documented disabled state.
- [ ] Complete the Ubuntu service, port, user, UFW, and logging baseline.
- [ ] Create and test the Ubuntu snapshot and rollback procedure.
- [ ] Add a second endpoint and validate Ubuntu UFW externally.
- [ ] Configure and test Proxmox startup ordering for OPNsense-dependent guests.
- [ ] Establish an independent backup destination and perform a restore test.
- [ ] Reevaluate east-west segmentation before adding multiple vulnerable targets.

## 7. Interview-Relevant Takeaways

The most important professional lessons demonstrated so far are:

- I can distinguish a logical network diagram from the configuration that actually enforces it.
- I validate network controls with both endpoint behavior and firewall logs.
- I understand that firewall rules depend on direction, first-match order, address family, protocol, and port semantics.
- I can explain the difference between hypervisor networking, routed firewall traffic, and same-subnet switched traffic.
- I make virtualization decisions based on isolation and kernel requirements as well as resource efficiency.
- I document limitations and remaining work instead of overstating the scope of a successful test.
- I treat snapshots, configuration exports, and independent backups as different recovery mechanisms.

## 8. Update Standard

A new lesson should be added when any of the following occurs:

- A result differs from the expected design.
- A configuration mistake is identified and corrected.
- A test reveals that a control's scope is narrower than assumed.
- A design decision introduces a meaningful tradeoff.
- A recovery or monitoring exercise reveals a gap.
- A repeated task produces a more reliable method.

Each new entry should include the situation, cause or analysis, resolution when applicable, validation method, lasting lesson, and remaining action. Sensitive values and unnecessary home-network details must be removed before publication.

## 9. Change Log

| Date | Change |
|---|---|
| 2026-09-02 | Created the initial lessons-learned record from the Proxmox, OPNsense, Ubuntu, firewall-validation, and repository-documentation phases. |
