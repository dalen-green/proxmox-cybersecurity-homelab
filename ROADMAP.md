# Cybersecurity Home Lab Roadmap

Last updated: 2026-09-03

This roadmap describes the planned development of a segmented cybersecurity home lab built on Proxmox VE. The lab is intended to demonstrate practical experience in virtualization, network security, Linux and Windows administration, identity management, vulnerability assessment, security monitoring, and recovery testing.

This is a public, high-level plan. Detailed task tracking, unsanitized configurations, credentials, and private network information are intentionally kept outside this repository.

## Status Definitions

| Status | Meaning |
|---|---|
| **Verified** | The control or system was implemented, tested, and produced repeatable results. |
| **In progress** | Implementation or validation is currently underway. |
| **Planned** | The milestone has been defined but work has not started. |
| **Optional** | A stretch goal that will be pursued after the core lab is operational. |

Portfolio documentation is tracked separately from technical implementation:

| Portfolio status | Meaning |
|---|---|
| **Published** | Sanitized documentation and evidence are available in this repository. |
| **Drafting** | Evidence exists, but the project write-up is not yet complete. |
| **Not started** | No public project write-up has been created. |

## Current Progress

| ID | Milestone | Lab status | Portfolio status |
|---|---|---|---|
| `LAB-01` | Proxmox foundation | **Verified** | **Published** |
| `LAB-02` | OPNsense network segmentation | **Verified for IPv4** | **Drafting** |
| `LAB-03` | Ubuntu Server security baseline | **In progress** | **Not started** |
| `LAB-04` | Windows 11 endpoint | **Planned** | **Not started** |
| `LAB-05` | Windows Server and Active Directory | **Planned** | **Not started** |
| `LAB-06` | Authorized vulnerability testing | **Planned** | **Not started** |
| `LAB-07` | Wazuh monitoring and detection | **Planned** | **Not started** |
| `LAB-08` | Backup and recovery validation | **Planned** | **Not started** |

## Roadmap

### `LAB-01` — Proxmox Foundation

**Objective:** Establish a stable virtualization platform for all later security projects.

Completed work:

- [x] Install Proxmox VE on the dedicated lab host.
- [x] Configure secure management access from the trusted home network.
- [x] Configure appropriate software repositories and install updates.
- [x] Review local storage roles for ISO images and virtual disks.
- [x] Create `vmbr1` as an isolated virtual bridge with no physical uplink.

Completion evidence:

- Sanitized platform and storage overview
- Diagram showing the host, bridges, and management path
- Selected screenshots or command output confirming bridge configuration
- Explanation of why the isolated bridge has no physical port

Repository location: [`projects/01-proxmox-foundation/`](projects/01-proxmox-foundation/)

Published evidence: [LAB-01 evidence pack](projects/01-proxmox-foundation/evidence/)

---

### `LAB-02` — OPNsense Network Segmentation

**Objective:** Make OPNsense the controlled path between the upstream network and the isolated cybersecurity network.

Completed work:

- [x] Deploy OPNsense as a two-interface virtual machine.
- [x] Connect the WAN interface to `vmbr0`.
- [x] Connect the LAN interface to isolated bridge `vmbr1`.
- [x] Configure the isolated IPv4 lab subnet, DHCP, DNS forwarding, and outbound connectivity.
- [x] Create and order firewall rules to enforce an IPv4 access restriction.
- [x] Validate the restriction from a lab endpoint.
- [x] Locate the matching blocked connection in the OPNsense firewall log.
- [x] Keep vulnerable systems off `vmbr0` and avoid router port forwarding.

Remaining enhancements:

- [ ] Validate IPv6 behavior and either enforce equivalent isolation or explicitly disable unused IPv6 paths.
- [ ] Document the final rule logic using sanitized names and addresses.
- [ ] Capture allowed and denied test results with explanatory captions.

Acceptance criteria:

- A lab endpoint receives an address from OPNsense.
- Approved DNS and internet traffic works through OPNsense.
- Prohibited traffic is denied by the intended rule.
- The firewall log records the denied test.
- No isolated target has a direct path through `vmbr0`.

Repository location: [`projects/02-opnsense-segmentation/`](projects/02-opnsense-segmentation/)

---

### `LAB-03` — Ubuntu Server Security Baseline

**Objective:** Deploy and harden a Linux server while documenting its users, services, exposed ports, firewall policy, and logs.

Planned and ongoing work:

- [x] Install Ubuntu Server on the isolated network.
- [x] Create administrative and standard user accounts.
- [x] Install operating-system updates.
- [x] Configure OpenSSH access.
- [x] Enable the UFW host firewall.
- [ ] Document users, groups, services, listening ports, and important logs.
- [ ] Review SSH settings and move to key-based authentication after recovery access is confirmed.
- [ ] Record a repeatable baseline-audit command set or script.
- [ ] Create and label a clean Proxmox snapshot.
- [ ] Test one controlled change and rollback.

Acceptance criteria:

- Administrative tasks require deliberate privilege elevation.
- Only expected services and ports are exposed.
- UFW policy is documented and verified from another lab endpoint.
- Relevant authentication and service events can be located in system logs.
- The clean baseline can be restored successfully.

Repository location: [`projects/03-ubuntu-server-baseline/`](projects/03-ubuntu-server-baseline/)

---

### `LAB-04` — Windows 11 Endpoint

**Objective:** Build a representative Windows workstation and establish a defensible endpoint-security baseline.

Planned work:

- [ ] Install Windows 11 with virtual TPM and UEFI support.
- [ ] Create separate administrative and standard accounts.
- [ ] Apply operating-system and security updates.
- [ ] Review Microsoft Defender, Windows Firewall, and audit settings.
- [ ] Inspect active services, network connections, startup items, and Event Viewer logs.
- [ ] Install Sysmon using a documented configuration.
- [ ] Create a clean baseline snapshot.
- [ ] Join the endpoint to the lab domain after `LAB-05` is operational.

Acceptance criteria:

- The endpoint is patched and uses least-privilege daily access.
- Host firewall and security protections are enabled and documented.
- A selected security event can be generated and found in the relevant log.
- The endpoint can later report security telemetry to Wazuh.

Repository location: [`projects/04-windows-active-directory/`](projects/04-windows-active-directory/)

---

### `LAB-05` — Windows Server and Active Directory

**Objective:** Build a small identity environment that demonstrates centralized authentication, authorization, DNS, and Group Policy administration.

Planned work:

- [ ] Install and patch Windows Server.
- [ ] Configure a static lab address and internal DNS.
- [ ] Install Active Directory Domain Services and promote the server to a domain controller.
- [ ] Design organizational units for users, computers, groups, and administrative roles.
- [ ] Create test users and security groups following least-privilege principles.
- [ ] Join the Windows 11 endpoint to the domain.
- [ ] Create and test baseline Group Policy Objects.
- [ ] Document authentication, account-management, and policy events.
- [ ] Snapshot or back up the environment before major changes.

Acceptance criteria:

- Domain users can authenticate from the Windows endpoint.
- DNS resolves required internal records correctly.
- Group membership grants only the intended access.
- At least one security-focused Group Policy is applied and verified.
- Relevant domain-controller events are captured and explained.

Repository location: [`projects/04-windows-active-directory/`](projects/04-windows-active-directory/)

---

### `LAB-06` — Authorized Vulnerability Testing

**Objective:** Perform a documented assessment against intentionally vulnerable systems owned and isolated within the lab.

Planned work:

- [ ] Deploy Kali Linux on `vmbr1`.
- [ ] Deploy an intentionally vulnerable target on `vmbr1`.
- [ ] Define the authorized scope and rules of engagement before testing.
- [ ] Perform host discovery, service enumeration, and vulnerability identification.
- [ ] Manually validate at least one finding without causing unnecessary damage.
- [ ] Document risk, evidence, remediation, and limitations.
- [ ] Apply a remediation or compensating control and retest it.
- [ ] Restore the target to its clean snapshot after the exercise.

Acceptance criteria:

- All testing remains inside the explicitly authorized lab scope.
- Findings distinguish scanner output from manually validated evidence.
- At least one issue has a documented remediation and successful retest.
- The final report communicates technical risk in clear business language.

Repository location: [`projects/05-vulnerability-testing/`](projects/05-vulnerability-testing/)

---

### `LAB-07` — Wazuh Monitoring and Detection

**Objective:** Centralize endpoint telemetry and demonstrate detection, investigation, and tuning of security events.

Planned work:

- [ ] Deploy Wazuh using a resource-conscious architecture.
- [ ] Enroll the Ubuntu and Windows endpoints.
- [ ] Forward additional firewall or infrastructure logs where practical.
- [ ] Verify log ingestion and host health.
- [ ] Generate controlled events such as failed logins, account changes, or authorized network scans.
- [ ] Investigate alerts using source logs and endpoint context.
- [ ] Tune one noisy rule without hiding meaningful activity.
- [ ] Document one detection use case from event generation through analyst conclusion.

Acceptance criteria:

- Both Linux and Windows telemetry are visible in the monitoring platform.
- A controlled security event produces an expected alert.
- The alert can be traced back to its original log source.
- Investigation notes explain severity, evidence, conclusion, and recommended action.
- Any tuning change is tested for both false positives and missed detections.

Repository location: [`projects/06-wazuh-monitoring/`](projects/06-wazuh-monitoring/)

---

### `LAB-08` — Backup and Recovery Validation

**Objective:** Prove that important lab systems can be recovered instead of assuming that snapshots or backups will work.

Planned work:

- [ ] Define which systems require snapshots, configuration exports, and full backups.
- [ ] Document the difference between a snapshot and an independent backup.
- [ ] Store sensitive configuration exports outside the public repository.
- [ ] Create a Proxmox backup of at least one non-production lab VM.
- [ ] Restore the VM or restore it as an isolated clone.
- [ ] Verify boot, networking, accounts, and a representative service after recovery.
- [ ] Record recovery time, problems encountered, and lessons learned.

Acceptance criteria:

- At least one VM is recovered from an independent backup.
- Recovered services pass a documented functional test.
- Recovery evidence includes timestamps and validation results.
- The process identifies realistic recovery-point and recovery-time expectations.

Repository location: [`projects/07-backup-recovery/`](projects/07-backup-recovery/)

## Portfolio Completion Standard

A milestone is marked **Published** only after its project folder contains:

- [ ] A clear objective and scope
- [ ] A sanitized environment description
- [ ] An architecture or data-flow diagram when useful
- [ ] An implementation summary focused on decisions, not every click
- [ ] Security reasoning for the major configuration choices
- [ ] Repeatable validation steps and expected results
- [ ] Selected evidence with captions explaining what each item proves
- [ ] At least one problem encountered and its resolution
- [ ] Skills demonstrated and remaining improvements
- [ ] A final check for credentials, keys, public IP addresses, serial numbers, and other sensitive data

## Priority Order

### Now

1. Finish `LAB-03`, including the Ubuntu baseline review and snapshot validation.
2. Publish sanitized documentation for `LAB-02` and `LAB-03`.
3. Complete the remaining IPv6 isolation review for `LAB-02`.

### Next

1. Build the Windows 11 endpoint.
2. Deploy Windows Server and Active Directory.
3. Join the endpoint to the domain and validate Group Policy.
4. Conduct the first authorized vulnerability assessment.

### Later

1. Deploy Wazuh and enroll the Linux and Windows endpoints.
2. Build and document detection-and-investigation exercises.
3. Complete an independent backup and restore test.
4. Refine the repository for résumé and interview use.

## Optional Enhancements

- [ ] Compare an unprivileged LXC service with an equivalent Linux VM, including resource use and isolation tradeoffs.
- [ ] Add lightweight services such as a web server, DNS server, or log receiver using unprivileged LXC containers.
- [ ] Run vulnerable application containers only inside a disposable VM on `vmbr1`.
- [ ] Add infrastructure diagrams created from version-controlled source.
- [ ] Add Markdown linting and secret scanning to the repository workflow.
- [x] Use the root README as a concise portfolio landing page that directs readers to detailed project evidence and design documentation.

## Safety and Ethics

All security testing documented in this repository is limited to systems I own and have explicitly designated as laboratory targets. Intentionally vulnerable systems remain isolated behind OPNsense on `vmbr1`. The lab does not use router port forwarding to expose vulnerable services, and it is not used to scan or test third-party systems without authorization.

## Revision Practice

This roadmap will be updated when a milestone changes state. A status changes to **Verified** only after its acceptance criteria have been tested, and a portfolio status changes to **Published** only after the supporting evidence has been sanitized and committed.
