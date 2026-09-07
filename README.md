# Proxmox Cybersecurity Home Lab

An evidence-driven cybersecurity lab built on a dedicated Dell OptiPlex 7090 SFF running Proxmox VE. OPNsense provides routing and firewall enforcement between the upstream private network and an isolated virtual lab network.

This portfolio emphasizes architecture, security reasoning, repeatable validation, troubleshooting, and clearly stated limitations. Verified, in-progress, and planned work are labeled separately so each technical claim reflects the evidence currently available.

AI tools were used to assist with documentation organization and editing. All configurations, testing, evidence collection, and technical conclusions were performed or independently verified by the repository owner.

## Start Here

| Resource | What it demonstrates |
|---|---|
| **[Project 01: Proxmox Foundation](projects/01-proxmox-foundation/)** | Bare-metal deployment, repository and update configuration, storage roles, Linux bridges, validation, security reasoning, and limitations. |
| **[Project 02: OPNsense Network Segmentation](projects/02-opnsense-segmentation/)** | Two-interface firewall deployment, IPv4 DHCP/DNS/NAT, ordered protected-destination policy, controlled SSH denial, and firewall-log correlation. |
| **[Lab Architecture](docs/architecture.md)** | Current topology, system roles, bridge and interface mapping, resource planning, data flows, and the distinction between deployed and planned systems. |
| **[Security Boundaries](docs/security-boundaries.md)** | Trust zones, protected assets, firewall principles, verified controls, required controls, validation methods, and residual risk. |
| **[Lessons Learned](docs/lessons-learned.md)** | Configuration mistakes, troubleshooting decisions, tested corrections, open improvements, and a reusable validation process. |
| **[Project Roadmap](ROADMAP.md)** | Technical and portfolio status, acceptance criteria, and the planned progression from infrastructure through monitoring to a healthcare integration capstone and authorized testing. |

## Current State

| Workstream | Status | Current result |
|---|---|---|
| `LAB-01` Proxmox foundation | **Verified** / **Published** | Proxmox is operational; repositories, updates, storage roles, `vmbr0`, and internal-only `vmbr1` are validated in the [sanitized evidence pack](projects/01-proxmox-foundation/evidence/). |
| `LAB-02` OPNsense segmentation | **Verified for IPv4** / **Published** | IPv4 DHCP, DNS, NAT, and outbound access work; a protected-destination rule was validated with a controlled SSH/TCP 22 attempt and matching [sanitized firewall evidence](projects/02-opnsense-segmentation/evidence/). |
| `LAB-03` Ubuntu Server baseline | **In progress** | Installation, account separation, updates, OpenSSH, and UFW enablement are complete; baseline audit and recovery validation remain. |
| `LAB-04`–`LAB-09` | **Planned** | Windows endpoint security, Active Directory, Wazuh, a simulated clinical laboratory capstone with independent recovery testing, Kali control validation, and vulnerable-target assessment are defined but not claimed as implemented. |

The [roadmap](ROADMAP.md) is the source of truth for detailed status and completion criteria.

## Planned Progression

`LAB-04` Windows 11 → `LAB-05` Windows Server / Active Directory → `LAB-06` Wazuh → `LAB-07` Secure Clinical Laboratory → `LAB-08` Kali Control Validation → `LAB-09` Vulnerable Systems and Web Applications.

The planned healthcare capstone will combine the earlier components to demonstrate fictional laboratory workflows, role-based access, risk assessment, control testing, monitoring, and downtime/recovery validation. It will use synthetic data only and will not claim Epic Beaker experience, production clinical validation, or regulatory compliance. Its design documents may be drafted before implementation; the next infrastructure milestone remains Windows 11.

## Capabilities Demonstrated

- Proxmox VE installation, administration, storage interpretation, and virtual networking
- Network segmentation using Linux bridges and a two-interface OPNsense firewall
- Firewall-rule design, traffic testing, and log correlation
- Linux server administration and host-firewall configuration
- Security architecture, trust-boundary analysis, and residual-risk documentation
- Evidence-driven troubleshooting, technical writing, and public-data sanitization

## Evidence and Safety

A control is described as **verified** only after implementation and a repeatable test or configuration review. Evidence is selected for what it proves rather than preserved as a click-by-click installation diary. Public material excludes credentials, keys, exact upstream addresses, MAC addresses, serial numbers, and unsanitized configuration exports.

All security testing is limited to systems I own or am explicitly authorized to test. Intentionally vulnerable systems will remain isolated behind OPNsense, with no router port forwarding or testing of third-party systems. Healthcare exercises will not use patient information or internal employer configurations.
