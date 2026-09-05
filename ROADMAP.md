# Cybersecurity Home Lab Roadmap

Last updated: 2026-09-05

This roadmap describes the planned development of a segmented cybersecurity home lab built on Proxmox VE. The lab is intended to demonstrate practical experience in virtualization, network security, Linux and Windows administration, identity management, security monitoring, recovery testing, and authorized vulnerability assessment.

The planned progression connects those foundations to a simulated clinical laboratory environment: infrastructure → endpoint and identity administration → monitoring → healthcare integration → authorized attack and vulnerability validation. The healthcare project is an integration capstone, not a replacement for the foundational labs.

This is a public, high-level plan. Detailed task tracking, unsanitized configurations, credentials, and private network information are intentionally kept outside this repository. Healthcare scenarios will use synthetic data only, with no patient information or internal employer configurations.

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
| `LAB-04` | Windows 11 endpoint security | **Planned** | **Not started** |
| `LAB-05` | Windows Server and Active Directory | **Planned** | **Not started** |
| `LAB-06` | Wazuh monitoring and detection | **Planned** | **Not started** |
| `LAB-07` | Secure clinical laboratory environment | **Planned** | **Not started** |
| `LAB-08` | Kali attack and control validation | **Planned** | **Not started** |
| `LAB-09` | Vulnerable systems and web applications | **Planned** | **Not started** |

## Sequencing and Healthcare Focus

Windows endpoint administration and Active Directory come before centralized monitoring. Wazuh then provides a monitoring foundation for the healthcare capstone and later authorized testing. Kali and deliberately vulnerable targets are not prerequisites for the initial healthcare project.

The `LAB-07` charter, fictional workflow, proposed architecture, user roles, access matrix, and initial risk register may be drafted now without adding VMs. Its technical implementation follows the relevant foundation, endpoint, identity, and monitoring acceptance criteria in `LAB-01` through `LAB-06`. Planning documents must remain clearly labeled as proposed; they are not evidence of implemented controls.

The earlier standalone backup-and-recovery milestone is retained as an explicit workstream and acceptance requirement within `LAB-07`. An independent restore test remains required; a snapshot or application restart alone does not satisfy it.

Project paths below are naming targets where a folder does not yet exist. Only existing project directories are linked. Updating this roadmap does not create project write-ups, promote technical status, or mark evidence as published.

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

Planned write-up location (not yet created): `projects/02-opnsense-segmentation/`

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

Planned write-up location (not yet created): `projects/03-ubuntu-server-baseline/`

---

### `LAB-04` — Windows 11 Endpoint Security

**Objective:** Build a representative Windows workstation and establish a defensible endpoint-security baseline.

**Dependencies:** The Proxmox foundation and documented OPNsense lab-network controls. Domain joining follows `LAB-05` rather than blocking the initial endpoint baseline.

Planned work:

- [ ] Confirm licensing and domain-join prerequisites before installation.
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

Planned repository location (not yet created): `projects/04-windows-endpoint-security/`

---

### `LAB-05` — Windows Server and Active Directory

**Objective:** Build a small identity environment that demonstrates centralized authentication, authorization, DNS, and Group Policy administration.

**Dependencies:** The Proxmox and network foundations, plus the Windows endpoint from `LAB-04` for client-side validation.

Planned work:

- [ ] Install and patch Windows Server.
- [ ] Configure a static lab address and internal DNS.
- [ ] Install Active Directory Domain Services and promote the server to a domain controller.
- [ ] Design organizational units for users, computers, groups, and administrative roles.
- [ ] Create test users and security groups following least-privilege principles.
- [ ] Join the Windows 11 endpoint to the domain.
- [ ] Create and test baseline Group Policy Objects.
- [ ] Test group-based permissions on a selected resource using allowed and denied accounts.
- [ ] Document authentication, account-management, and policy events.
- [ ] Snapshot or back up the environment before major changes.

Acceptance criteria:

- Domain users can authenticate from the Windows endpoint.
- DNS resolves required internal records correctly.
- Group membership grants only the intended access to the tested resource.
- At least one security-focused Group Policy is applied and verified.
- Relevant domain-controller events are captured and explained.

Domain and operating-system permissions will be documented separately from the future simulated LIS application's permissions. Creating an AD group alone does not demonstrate that the application enforces that role.

Planned repository location (not yet created): `projects/05-windows-active-directory/`

---

### `LAB-06` — Wazuh Monitoring and Detection

**Objective:** Centralize endpoint telemetry and demonstrate detection, investigation, and tuning of security events before the healthcare integration and attack-validation projects.

**Dependencies:** The Ubuntu baseline, Windows endpoint, and identity environment from `LAB-03` through `LAB-05`.

Planned work:

- [ ] Deploy Wazuh using a resource-conscious architecture.
- [ ] Enroll the Ubuntu and Windows endpoints, including relevant domain-controller telemetry.
- [ ] Forward additional firewall or infrastructure logs where practical.
- [ ] Verify log ingestion, timestamps, and host health.
- [ ] Generate controlled events such as failed logins or test-account changes without requiring Kali or a vulnerable target.
- [ ] Investigate alerts using source logs and endpoint context.
- [ ] Tune one noisy rule without hiding meaningful activity.
- [ ] Document one detection use case from event generation through analyst conclusion.
- [ ] Record which VM combinations fit within the host's memory and storage limits.

Acceptance criteria:

- Both Linux and Windows telemetry are visible in the monitoring platform; collection may be validated in separate resource-conscious sessions.
- A controlled security event produces an expected alert.
- The alert can be traced back to its original log source.
- Investigation notes explain severity, evidence, conclusion, and recommended action.
- Any tuning change is tested for both false positives and missed detections.

Planned repository location (not yet created): `projects/06-wazuh-monitoring/`

---

### `LAB-07` — Secure Clinical Laboratory Environment

**Objective:** Integrate the earlier labs into a fictional clinical laboratory scenario demonstrating how workflow requirements become access decisions, security controls, monitoring, and downtime/recovery procedures.

**Dependencies:** The relevant foundation, endpoint, identity, and monitoring acceptance criteria in `LAB-01` through `LAB-06`. Planning may begin now; implementation and testing remain future work. Kali and deliberately vulnerable systems are not required for this capstone.

**Scope:** A small, explicitly simulated laboratory information system using synthetic orders, specimens, and results. This is not Epic Beaker, a production LIS, a clinically validated system, or a claim of regulatory compliance. No patient data, employer procedures, or internal employer configurations will be used.

Planning work:

- [ ] Write a project charter defining the fictional laboratory, scope, assumptions, exclusions, and success criteria.
- [ ] Model an order-to-result workflow and its data flows, assets, and trust boundaries.
- [ ] Define fictional technologist, supervisor, student, LIS analyst, IT administrator, security analyst, and vendor roles.
- [ ] Create an access-control matrix with permitted and prohibited actions and role-specific reasoning.
- [ ] Build a risk register linking threats to laboratory operational impact, planned controls, evidence, and residual risk.
- [ ] Design a phased architecture that reuses earlier lab components without overwriting their baseline evidence or assuming every VM runs concurrently.

Implementation and validation work:

- [ ] Deploy a minimal simulated LIS or result-tracking application with synthetic records and documented limitations.
- [ ] Implement selected group-based resource permissions and application-level roles; explicitly document whether and how the application uses directory identities.
- [ ] Separate routine laboratory actions, application configuration, server administration, and security-log review in the tested permission model.
- [ ] Define required traffic flows and implement selected host-firewall and routed-boundary controls.
- [ ] Start with the existing lab subnet; introduce separate subnets or VLANs only when needed and validated. Do not claim OPNsense inspects ordinary same-subnet traffic.
- [ ] Test permitted actions and denied actions, including a student attempting to modify a result.
- [ ] Record meaningful application events and forward selected logs to Wazuh; test at least one detection end to end.
- [ ] Simulate approved, restricted, time-limited vendor access to a designated lab resource, including activation, logging, revocation, and a denied retest.
- [ ] Record the limits of administrative separation: application roles do not eliminate the power of host or database administrators.

Downtime and independent recovery work:

- [ ] Define which systems need snapshots, private configuration exports, and independent backups.
- [ ] Define simulated recovery-time and recovery-point objectives before the exercise.
- [ ] Create a backup of at least one relevant VM or application dataset on storage independent of its active VM storage; document remaining shared failure risks.
- [ ] Create a fictional downtime workflow covering outage confirmation, escalation, synthetic accession/result tracking, and post-restoration reconciliation.
- [ ] Run a controlled application outage and record the downtime workflow using synthetic specimens only.
- [ ] Restore from the independent backup into an isolated recovery environment and validate accounts, services, and expected records.
- [ ] Reconcile synthetic downtime records, checking for missing or duplicate results and preserving an audit trail.
- [ ] Record recovery timing, the actual recoverable data point, functional test results, gaps, and lessons learned.
- [ ] Conduct a ransomware tabletop focused on laboratory continuity and recovery; do not deploy ransomware or unknown malware.

Acceptance criteria:

- The documentation connects laboratory workflows to risks, controls, and repeatable tests rather than listing installed tools alone.
- A role matrix is supported by both successful authorized tests and denied unauthorized tests.
- A selected application or security event is traceable from its source log through monitoring and an investigation note.
- At least one relevant system or dataset is restored from an independent backup and passes a documented functional check.
- The downtime exercise preserves synthetic specimen/result traceability and includes a documented reconciliation step.
- Recovery measurements are compared with the objectives; unmet objectives and residual risks are reported rather than hidden.
- Proposed features, tested controls, and tabletop-only decisions are clearly distinguished.

Planned evidence pack: charter and workflow, proposed/final architecture, access matrix, risk register, control/test matrix, captioned allow/deny evidence, monitoring investigation, independent restore results, downtime reconciliation, and tabletop notes. Earlier implementation evidence will be referenced rather than duplicated.

Planned repository location (not yet created): `projects/07-secure-clinical-laboratory/`

---

### `LAB-08` — Kali Attack and Control Validation

**Objective:** Use an authorized assessment workstation to test selected controls and explain what the firewall, endpoints, and monitoring platform actually observed.

**Dependencies:** The earlier endpoint, identity, and monitoring foundations, plus the required safety gates in [security-boundaries.md](docs/security-boundaries.md). The healthcare capstone is the preferred context, using disposable clones or explicitly designated synthetic test endpoints rather than risking its baseline.

Planned work:

- [ ] Deploy Kali Linux on `vmbr1` or a later restricted testing segment, never directly on `vmbr0`.
- [ ] Define exact source/target systems, permitted techniques, stopping conditions, and rules of engagement before testing.
- [ ] Complete the protected-network, management-access, IPv6, snapshot, and scope checks required before testing.
- [ ] Perform bounded host/service discovery against designated owned test endpoints.
- [ ] Generate non-destructive access and authentication tests against selected controls using test accounts.
- [ ] Correlate endpoint behavior with the logs from the enforcement point and Wazuh where applicable.
- [ ] Document which activities produced alerts, which did not, and why; do not assume every blocked connection or scan generates a Wazuh alert.
- [ ] Correct or tune one identified control or detection gap and retest.
- [ ] Restore changed test systems to their documented baseline.

Acceptance criteria:

- All traffic remains within the explicitly authorized scope.
- Evidence identifies the actual enforcement point; same-subnet traffic is not misattributed to OPNsense.
- At least one control is tested with both allowed and denied activity.
- A tested detection or visibility gap has a documented conclusion and retest.
- No deliberately vulnerable target is required for this initial control-validation lab; those targets are introduced in `LAB-09`.

Planned repository location (not yet created): `projects/08-kali-control-validation/`

---

### `LAB-09` — Vulnerable Systems and Web Applications

**Objective:** Perform a documented assessment against deliberately vulnerable systems owned and isolated within the lab, then demonstrate remediation or compensating controls and a retest.

**Dependencies:** The assessment workstation and rules-of-engagement practice from `LAB-08`, monitoring from `LAB-06`, and revalidation of the required safety gates before introducing any vulnerable target.

Planned work:

- [ ] Deploy one deliberately vulnerable full operating system as a disposable VM or one vulnerable application inside a disposable VM.
- [ ] Keep targets on `vmbr1` or a later restricted segment, with no direct `vmbr0` adapter, external exposure, or unnecessary outbound access.
- [ ] Keep vulnerable-target environments separate from the healthcare baseline; document any deliberate test connection.
- [ ] Capture a clean snapshot and confirm the target's restoration process.
- [ ] Define the authorized scope and rules of engagement for each assessment.
- [ ] Perform service enumeration and vulnerability identification.
- [ ] Manually validate at least one finding without unnecessary damage.
- [ ] Distinguish scanner output, confirmed behavior, business impact, and assessment limitations.
- [ ] Apply a remediation or compensating control and retest it.
- [ ] Review available monitoring evidence, restore the target, and shut it down when not needed.

Acceptance criteria:

- The target's isolation and authorized scope are documented before assessment.
- At least one finding includes supporting evidence, risk reasoning, a remediation or compensating control, and a successful retest.
- The final report distinguishes observed facts from assumptions and scanner claims.
- Target cleanup and restoration are documented.

Planned repository location (not yet created): `projects/09-vulnerable-systems-web-apps/`

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
3. Complete the remaining protected-network, management-access, and IPv6 isolation review before authorized attack or vulnerability testing.
4. Optionally begin the `LAB-07` charter, fictional workflow, proposed architecture, roles, access matrix, and initial risk register without adding VMs or claiming implementation.

### Next

1. Build and validate the Windows 11 endpoint in `LAB-04`.
2. Deploy Windows Server and Active Directory in `LAB-05`; join the endpoint and validate group-based access and Group Policy.
3. Deploy Wazuh in `LAB-06` and validate selected Linux, Windows, and infrastructure telemetry using controlled events.

### Integration and Later Testing

1. Implement `LAB-07` incrementally, connecting laboratory workflows to identity, application permissions, network controls, and monitoring.
2. Complete the capstone's independent backup/restore, downtime reconciliation, and incident-response tabletop requirements.
3. Use Kali for authorized control validation in `LAB-08` after the safety gates are satisfied.
4. Introduce disposable vulnerable systems and web applications for assessment and remediation in `LAB-09`.
5. Refine the repository for résumé and interview use as each evidence pack is published.

## Optional Enhancements

- [ ] Compare an unprivileged LXC service with an equivalent Linux VM, including resource use and isolation tradeoffs.
- [ ] Add lightweight services such as a web server, DNS server, or log receiver using unprivileged LXC containers.
- [ ] Run vulnerable application containers only inside a disposable VM on `vmbr1` or a later restricted segment.
- [ ] Expand internal segmentation only after documenting required flows and a recovery path for administrative access.
- [ ] Add infrastructure diagrams created from version-controlled source.
- [ ] Add Markdown linting and secret scanning to the repository workflow.
- [x] Use the root README as a concise portfolio landing page that directs readers to detailed project evidence and design documentation.

## Safety and Ethics

All security testing documented in this repository is limited to systems I own and have explicitly designated as laboratory targets. Intentionally vulnerable systems remain isolated behind OPNsense on `vmbr1` or a later restricted lab segment. The lab does not use router port forwarding to expose vulnerable services, and it is not used to scan or test third-party systems without authorization.

Healthcare scenarios use synthetic data and fictional workflows only. A simulated LIS does not establish Epic Beaker administration experience, clinical validation, or regulatory compliance. Ransomware scenarios are tabletop exercises, not malware deployments. The existing [security boundaries](docs/security-boundaries.md) and their required pre-testing checks remain applicable throughout the roadmap.

## Revision Practice

This roadmap will be updated when a milestone changes state. A status changes to **Verified** only after its acceptance criteria have been tested, and a portfolio status changes to **Published** only after the supporting evidence has been sanitized and committed.

The 2026-09-05 revision preserves `LAB-01` through `LAB-03` and their recorded statuses, moves Wazuh ahead of Kali, adds the healthcare integration capstone as `LAB-07`, and separates initial control validation from later vulnerable-target assessment. Planned folder names align with the lab IDs; no existing project directory or evidence file was renamed.
