# Proxmox Cybersecurity Home Lab

I am building this lab as my first hands-on IT project. My background is in microbiology, and I want to connect my healthcare laboratory experience with practical IT and cybersecurity skills. I am learning the Linux commands, networking concepts, and troubleshooting methods as I go.

The lab runs on a dedicated Dell OptiPlex 7090 SFF with Proxmox VE. OPNsense provides the normal routed connection between the home network and an internal virtual lab network. Ubuntu is my first server-administration project.

My notes explain what I tried, why I chose a setting, what the evidence shows, and what I still need to test. I use AI tools to help organize and edit the documentation. The lab activities and saved results are my work; editing a write-up does not replace understanding or testing a configuration.

## Start with the projects

| Project | Current progress | What I am learning |
|---|---|---|
| [LAB-01: Proxmox Foundation](projects/01-proxmox-foundation/) | **Verified for its foundation scope / Published** | Installing a hypervisor, choosing update sources, understanding storage, and connecting virtual bridges |
| [LAB-02: OPNsense Segmentation](projects/02-opnsense-segmentation/) | **Verified for the recorded IPv4 setup and SSH test / Published** | Following a connection, ordering firewall rules, and comparing an endpoint result with a firewall log |
| [LAB-03: Ubuntu Server Baseline](projects/03-ubuntu-server-baseline/) | **In progress / Drafting** | Accounts, updates, hardened SSH, and active UFW; full service/log review, independent firewall tests, and recovery evidence remain |

LAB-03 has five of its nine core evidence artifacts reviewed. Its SSH hardening and UFW configuration are recorded; the overall baseline is still unfinished. The evidence reflects its capture dates rather than a live audit of the systems.

## How the lab fits together

| Notes | What they explain |
|---|---|
| [Architecture](docs/architecture.md) | What each system does, why WAN uses `vmbr0` and LAN uses `vmbr1`, and how temporary management access works |
| [Security boundaries](docs/security-boundaries.md) | Which connections the firewalls control, what the tests establish, and which protections still need checking |
| [Lessons learned](docs/lessons-learned.md) | The confusing parts, troubleshooting steps, and habits I am developing |
| [Roadmap](ROADMAP.md) | Completion criteria and the planned learning sequence |

## What comes next

The planned order is Windows 11, Windows Server / Active Directory, Wazuh monitoring, a simulated clinical laboratory, Kali control validation, and vulnerable-target assessment. These are LAB-04 through LAB-09; they have not been implemented.

The clinical laboratory project is where I want to bring my existing laboratory experience into the lab. It will use fictional workflows and synthetic records to practice access decisions, security controls, monitoring, and downtime recovery. Its design can be drafted before implementation. It does not represent Epic Beaker administration experience or a production LIS deployment.

## Evidence and testing scope

Each project links to selected screenshots or command output. I distinguish saved settings, recorded functional tests, and planned checks. For example, one blocked SSH attempt supports that tested result; it does not establish complete isolation of every network path.

Testing is limited to systems I own or am explicitly authorized to use. The documented setup has no home-router port forwarding for lab services. Broader protected-network, management-access, and IPv6 checks remain required before deliberately vulnerable targets are added. Public copies omit credentials, keys, upstream addresses, and unnecessary identifying details. Healthcare exercises will use synthetic data and fictional workflows.
