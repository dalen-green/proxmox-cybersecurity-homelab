# Security Boundaries

> **Document status:** Living learning notes\
> **Last updated:** 2026-09-25\
> **Current work:** Proxmox foundation, OPNsense networking, and the Ubuntu baseline

## What I am trying to protect

I am building this lab to learn IT and cybersecurity without putting my everyday computers or personal information in the path of my experiments. I started with very little networking experience, so I am learning to ask a specific question about each connection: which system can reach which service, and what actually controls that access?

A **security boundary** is where I expect access to be limited. In this lab, OPNsense controls the normal routed path between networks, while Ubuntu's UFW firewall controls traffic at the Ubuntu server. I need evidence that each protection works before making a broad claim about isolation.

My protected assets include my home devices and data, Proxmox administration, the OPNsense configuration, and the lab's disks and recovery material. Testing is limited to systems I own or am explicitly authorized to test.

The [architecture](architecture.md) shows the connections. This document explains the protections I have configured, what the saved tests show, and what still needs work.

## How the boundaries fit together

| Area | What is there | What I am trying to allow or prevent |
|---|---|---|
| Home network | My workstation, router, and household devices | Keep lab experiments from gaining unnecessary access to these systems |
| Proxmox management | The services used to administer the host and VMs | Allow deliberate administration from my trusted workstation |
| OPNsense | Lab gateway, firewall, and network services | Control routed traffic and limit access to its own administration services |
| Lab network on `vmbr1` | Ubuntu now; other systems later | Allow the connections needed for each exercise |
| Internet | External services and networks | Permit needed outbound use without exposing management or lab services through router port forwarding |

Proxmox management and OPNsense WAN share `vmbr0` and the same physical Ethernet connection. They have different jobs, but they are not physically separate networks. OPNsense WAN is Proxmox `net1` / guest `vtnet1`; LAN is `net0` / `vtnet0` on `vmbr1`.

`vmbr1` has no physical uplink. That makes it an internal virtual switch, but traffic can still leave through OPNsense. A guest with an extra `vmbr0` adapter would have another path to the home network, so I check each VM's adapter assignment.

I also use a temporary Proxmox address, `10.10.10.2/24`, for some LAN-side administration. That is an exception to the normal design and is explained below. The internal bridge should not be described as permanently unreachable from the host.

## What I have checked so far

I use **configuration reviewed** for a saved setting and **tested** for an observed result. These are different kinds of evidence. A screenshot can show that a rule exists without proving that the traffic I care about matched it.

| Control | Recorded state | Evidence and limit |
|---|---|---|
| `SB-01`: No physical uplink on `vmbr1` | Configuration reviewed | LAB-01 bridge screenshot; does not establish absence of a temporary runtime address |
| `SB-02`: Ubuntu attached to `vmbr1` | Configuration reviewed | LAB-03 hardware screenshot and LAB-02 guest networking output |
| `SB-03`: Correct OPNsense adapter mapping | Configuration reviewed | Proxmox hardware and OPNsense interface screenshots agree |
| `SB-04`: IPv4 lab networking | Tested for the recorded connections | DHCP lease, route/DNS output, and successful HTTPS request support the documented routed/NAT setup |
| `SB-05`: Logged block above the broad IPv4 allow rule | Tested for one SSH flow | Ubuntu timeout and corresponding OPNsense TCP/22 block entries |
| `SB-06`: Comprehensive protection of upstream networks | Required before vulnerable targets | Review the full destination scope and test additional protocols; one tested destination is insufficient |
| `SB-07`: Restricted Proxmox and OPNsense management access | Required before vulnerable targets | Test unauthorized access from `vmbr1` and document deliberate administrative exceptions |
| `SB-08`: IPv6 boundary | Required before vulnerable targets | Review enabled IPv6 paths and test equivalent restrictions or a deliberate disabled configuration |
| `SB-09`: No unsolicited internet access to the lab | Documented configuration; external test pending | Setup notes record no home-router port forwarding; I have no published external denial test |
| `SB-10`: Ubuntu UFW | Active configuration reviewed; independent testing pending | September 25 output shows default-deny inbound policy and SSH allowed from `10.10.10.2` |
| `SB-11`: Tested clean recovery point | Planned | Ubuntu snapshot and rollback artifacts are still missing |
| `SB-12`: Centralized monitoring | Planned | Wazuh belongs to a later project |

The evidence is linked from the [Proxmox](../projects/01-proxmox-foundation/evidence/), [OPNsense](../projects/02-opnsense-segmentation/evidence/), and [Ubuntu](../projects/03-ubuntu-server-baseline/evidence/) evidence indexes. These are records of the captured state, not a live security assessment.

## What the OPNsense test means

I created a LAN rule that blocks and logs IPv4 traffic from the lab network to a selected protected upstream destination. Its protocol and port fields are set to `any`. I placed it above the broad IPv4 LAN allow rule.

The connection I used to test that rule was SSH, which uses TCP destination port 22. Ubuntu reported a timeout. The firewall log showed the matching LAN source, TCP destination port, block action, rule label, and closely matching time. The log is what connects the timeout to OPNsense's action; a timeout on its own could have several causes.

This gives me a specific result I can explain: the selected IPv4 SSH attempt was blocked by the intended rule. I still need broader tests before claiming that every home device, management service, protocol, or IPv6 path is protected.

The rules screenshot also shows a broad IPv4 allow rule and a separate IPv6 allow rule. “Approved internet use” currently describes my intended use of the lab. It does not mean the firewall enforces a narrow list of approved websites or repositories.

### What I learned about rule order

For the quick interface rules used here, the first matching rule takes effect. If the broad allow rule matches first, the later block does not get a chance to stop that new connection. OPNsense also has other rule categories and existing connection states, so this is not a claim that every rule on the firewall follows one simple list. The [official rule-processing explanation](https://docs.opnsense.org/manual/firewall.html#processing-order) describes those details.

When checking this policy, I need to confirm the rule's source, destination, address family, and position, apply changes, and generate a fresh test. Source ports are normally temporary client ports; destination port 22 identifies the SSH service in this test.

## What UFW adds on Ubuntu

UFW is Ubuntu's host firewall. The [saved status](../projects/03-ubuntu-server-baseline/evidence/05-ubuntu-ufw-status.txt) records:

| Setting | Recorded value | My interpretation |
|---|---|---|
| Status | Active | UFW is enabled |
| Incoming default | Deny | New inbound connections need an applicable allow rule, subject to UFW's supporting rules |
| Outgoing default | Allow | UFW is not enforcing a narrow outbound allowlist |
| Routed traffic | Disabled | The status reports forwarding disabled; Ubuntu is not intended to be the lab router |
| Logging | On, low | Selected firewall events can support later checks |
| Explicit SSH allow | TCP/22 from `10.10.10.2` | The documented management source is permitted; another lab address does not match this rule |

The LAB-03 notes record a successful fresh key-authenticated SSH connection after UFW activation. The published artifact itself shows the policy; independent blocked-traffic evidence has not been collected yet.

This distinction matters for the next test. A second lab VM should not automatically be able to SSH into Ubuntu under the current rule. I need to record a successful connection from the allowed management source and blocked activity from an independent unauthorized source. A temporary test listener can help show that UFW blocked a connection to a service that really was listening. Any temporary rule or service must be documented and removed afterward.

The [Ubuntu firewall documentation](https://ubuntu.com/server/docs/how-to/security/firewalls/) explains how source-specific rules work. I still need to review IPv6 behavior separately: the SSH artifact includes an IPv6 listener, and the IPv4 UFW allow rule does not prove IPv6 isolation.

### Why I need both firewalls

| Connection | Where I expect filtering |
|---|---|
| Ubuntu to another network through its gateway | OPNsense on the routed path; Ubuntu's host policy may also apply |
| Another `vmbr1` guest to Ubuntu | UFW on Ubuntu; ordinary same-subnet traffic bypasses OPNsense |
| Temporary Proxmox forwarding connection to Ubuntu | UFW on Ubuntu; Proxmox originates the lab-side connection |

The hardware screenshot has a Proxmox NIC firewall checkbox enabled. That alone does not show a configured and tested Proxmox firewall policy. I am not counting it as a separately validated protection.

## Administrative access and its temporary exception

Normal Proxmox administration comes from my workstation on the home network. The VM console provides another way to work on a guest when its networking or SSH is unavailable.

For LAN-side access, I have used SSH local forwarding through Proxmox with the temporary `10.10.10.2/24` address on `vmbr1`. Ubuntu sees Proxmox as the source of that connection. This is why its UFW SSH rule uses `10.10.10.2`, even though I type the SSH command on Windows.

The earlier OPNsense setup notes record closing the tunnel and removing that temporary address. Later Ubuntu work uses the path again. The latest evidence does not show whether it was removed after the September 25 session, so cleanup remains something to confirm. I should keep the address out of the permanent bridge configuration and verify removal after each use.

A source IP restriction also does not replace authentication. Ubuntu's recorded SSH settings require a public key for the administrative account, deny direct root login, and disable password and keyboard-interactive authentication. Its disabled forwarding settings apply to Ubuntu's SSH server; they do not describe forwarding configured on Proxmox.

No home-router port forwarding is documented for the lab. I use the OPNsense console or a deliberate LAN-side path for administration. Complete denial tests from unauthorized sources are still needed, including review of rules that allow access to OPNsense itself.

## Current behavior and intended policy

| Source and destination | Current evidence | Remaining work |
|---|---|---|
| Trusted workstation → Proxmox | Management access works in the foundation record | Document and test the exact permitted sources |
| Ubuntu → OPNsense DHCP/DNS and internet | Recorded IPv4 networking and HTTPS test succeeded | Review narrower outbound policy when exercises need it |
| Ubuntu → protected upstream test destination | The selected TCP/22 flow was blocked and logged | Expand testing to the required networks and services |
| Unauthorized lab endpoint → management services | Comprehensive denial is not documented | Complete `SB-06` and `SB-07` |
| Temporary Proxmox source → Ubuntu SSH | UFW permits `10.10.10.2`; successful login recorded in project notes | Confirm temporary-path cleanup and retain repeatable test evidence |
| Another lab VM → Ubuntu | Subject to UFW; independent results are pending | Test denial and correlate it with Ubuntu logs |
| Internet → lab | No router forwarding documented | An external denial test is not yet published; IPv6 also needs review |

NAT changes addresses for routed connections; it is not a substitute for a reviewed firewall policy. Likewise, assigning a private address does not by itself make a system inaccessible to other devices that have a route to it.

## Checks I still need to complete

| Check | Current position |
|---|---|
| `VAL-01`: Lab IPv4 configuration | Recorded successful DHCP/network setup |
| `VAL-02`: Outbound DNS/HTTPS | Recorded successful request |
| `VAL-03`: Protected-destination SSH denial | Recorded timeout matched to the firewall block |
| `VAL-04`: Firewall-log correlation | Recorded matching source, service, action, label, and time |
| `VAL-05`: Multiple protocols to protected networks | Not yet performed |
| `VAL-06`: Unauthorized management access | Not yet comprehensively tested |
| `VAL-07`: IPv6 behavior | Not yet tested |
| `VAL-08`: Independent UFW testing | Pending an independent endpoint and a test plan matching the source restriction |
| `VAL-09`: Ubuntu snapshot recovery | Not yet performed |

Before adding deliberately vulnerable targets, I plan to complete `SB-06`, `SB-07`, and `SB-08`, retain a private OPNsense configuration backup, and establish tested recovery points. I will keep targets off `vmbr0`, define the authorized source and destination before each exercise, limit unnecessary outbound access, and power targets off when they are not needed. Vulnerable applications will run inside disposable VMs. Unknown malware, ransomware deployment, and attacks on the host or third-party systems are outside this lab's scope.

## Limits I need to keep visible

Everything runs on one physical host. Losing the host or its storage could stop both the firewall and the guests. OPNsense also cannot protect against a compromised Proxmox administrator or hypervisor.

The lab currently has one internal subnet. It does not yet have separate user, server, monitoring, or vulnerable-target networks. Host firewalls help, and additional segmentation may become useful as the exercises grow.

Centralized monitoring, independent backups, and recovery tests remain planned. The existing logs and successful connection tests are useful evidence, but they do not establish a complete security baseline.

For public documentation, I remove credentials, keys, upstream addresses, hardware identifiers, and unrelated personal information while retaining the lab addresses and fields needed to explain a test. Future healthcare exercises will use synthetic data and fictional workflows.

## Change log

| Date | Change |
|---|---|
| 2026-09-25 | Rewrote the boundaries in a learning-focused voice; aligned SSH/UFW progress, clarified the temporary management path, and corrected the independent UFW test expectation. |
| 2026-09-07 | Limited the firewall validation claim to the recorded SSH flow while preserving the configured rule's broader protocol/port scope. |
| 2026-09-03 | Corrected WAN/LAN mapping and distinguished console access from network traffic. |
| 2026-09-02 | Created the security-boundary record. |
