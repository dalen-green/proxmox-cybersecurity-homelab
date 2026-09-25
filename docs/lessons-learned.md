# Lessons Learned

> **Document status:** Living learning notes\
> **Last updated:** 2026-09-25\
> **Current work:** Proxmox foundation, OPNsense networking, and the Ubuntu baseline

## Why I am keeping these notes

This is my first time building an IT lab. I come from a microbiology background, and I am still learning Linux commands, networking terms, and how the different systems depend on each other. Getting a command to work is only part of the exercise; I also want to understand what changed and why it mattered.

These notes cover questions I had, mistakes I worked through, and limits I found in my testing. Some lessons came from a recorded problem. Others are design choices or things I still need to test, and I label them that way.

The [architecture](architecture.md) explains how the systems connect. The [security boundaries](security-boundaries.md) track the protections and unfinished checks. Each project has its own evidence folder so I can connect an explanation to a saved result.

## Proxmox: learning what the pieces do

### `LL-01` — A bridge's connections matter more than its name

**Recorded result: Mapping checked.**

I initially struggled to picture where `vmbr0` and `vmbr1` belonged. A diagram also needed correction. I learned that the numbers do not give the bridges built-in security roles.

I traced the actual connections: `vmbr0` has the physical Ethernet port, while `vmbr1` has no physical uplink. OPNsense connects to both, and Ubuntu connects to `vmbr1`. A bridge works like a virtual switch; OPNsense provides the routing and firewall rules between the networks.

Before adding another VM, I need to check which bridge its adapter uses. The name alone will not keep a guest on the intended network.

### `LL-04` — Disk space and RAM are separate allocations

**Recorded result: Initial confusion resolved.**

When I increased the OPNsense virtual disk to 32 GiB, I initially thought I was assigning all 32 GB of the computer's RAM to it. The similar units made the settings easy to mix up.

The VM actually has a 32 GiB virtual disk and 4096 MiB of RAM. The disk stores the operating system and files; RAM holds working data while the VM runs. I now keep CPU, RAM, and disk allocations in separate columns so I can see what each VM needs.

### `LL-05` — A checksum must match the exact file

**Recorded result: Upload hash calculated for the extracted ISO.**

The OPNsense download was compressed, but Proxmox needed the extracted ISO. I learned that the compressed archive and extracted image contain different bytes, so their checksums will differ.

I calculated SHA-256 for the exact ISO being uploaded. That helps check whether the uploaded bytes match my local file. It does not independently prove that the download came from the publisher; that requires the appropriate trusted publisher checksum or signature.

### `LL-06` — The browser's file path was a placeholder

**Recorded result: Hashing worked after using the real local path.**

PowerShell could not find the ISO when I used the path shown in the browser's upload field. The `C:\fakepath\` value was a browser privacy placeholder rather than its real location.

I made the file available locally, copied its actual path from File Explorer, and used PowerShell's `-LiteralPath` option. The OneDrive location added another detail to check: a visible file entry may still need downloading before a command can read all of its contents.

### `LL-14` — Containers save resources, but they change the exercise

**Recorded result: Design choice; containers remain optional.**

I considered LXC containers because the host has 32 GB of RAM. I learned that an LXC container shares the Proxmox Linux kernel, while a full VM has its own guest kernel.

I kept OPNsense and my first Ubuntu server as VMs. OPNsense needs FreeBSD, and Ubuntu gives me practice with a complete operating system. Windows and the planned assessment systems also remain VM projects. I may use unprivileged LXC containers later for lightweight support services; vulnerable applications are planned inside disposable VMs.

### `LL-15` — A virtual disk's size is not its current physical usage

**Recorded result: Storage behavior reviewed.**

Increasing the OPNsense virtual disk did not immediately consume that entire amount of physical storage. The `local-lvm` pool uses thin provisioning, which allocates storage as data is written.

This makes it easier to plan VM disks, but I still need to check the actual free space. Several guests can eventually use more storage than I expected from their initial sizes. Thin provisioning does not add physical capacity.

### `LL-18` — A small path typo can affect every link

**Recorded result: Directory corrected.**

The documentation folder was originally named `docs.` with a trailing period. I learned that the period was part of the path, even though it was easy to overlook. It also created avoidable problems for links and Windows file handling.

The files were moved to `docs/`, and their relative links were checked. I now want to settle folder names before building many references around them.

## OPNsense: learning to follow a connection

### `LL-02` — Adapter numbering does not tell me which side is WAN

**Recorded result: Mapping checked in Proxmox and OPNsense.**

Proxmox and OPNsense use different names for the same virtual adapters. I had to match them rather than assume the first adapter was the upstream connection.

| Proxmox adapter | Bridge | OPNsense name | Role |
|---|---|---|---|
| `net1` | `vmbr0` | `vtnet1` | WAN, toward the home network |
| `net0` | `vmbr1` | `vtnet0` | LAN, toward the lab |

Checking the hardware view, interface roles, and working lab address helped me understand the connection from both sides.

### `LL-03` — A DHCP address can change

**Recorded result: Changed WAN lease identified.**

The home router assigned OPNsense a different WAN address through DHCP. I had been treating the earlier address as if it were a permanent part of the design.

I checked the current lease in the OPNsense console and kept the documentation centered on the interface's role. A DHCP reservation may be useful later, but the current address should always be checked when diagnosing access. The Proxmox VM console also gives me a way to inspect OPNsense without depending on its web interface.

### `LL-07` — A block rule needs the right position and an applied change

**Recorded result: The selected SSH flow was blocked and logged.**

The protected-destination block needed to come before the broader IPv4 LAN allow rule. For the quick interface rules used here, an earlier matching allow can stop the later block from taking effect on a new connection.

I placed the block above the allow rule, applied the changes, and generated another SSH attempt. The endpoint timeout and matching OPNsense log showed the tested result. Rule order and applying changes were both checks in this troubleshooting process; an empty log alone did not identify which setting was wrong.

The [OPNsense documentation](https://docs.opnsense.org/manual/firewall.html#processing-order) explains the other rule categories and state handling that I still need to keep in mind.

### `LL-08` — The source and destination ports have different jobs

**Recorded result: Test fields reviewed.**

In the firewall log, Ubuntu used a temporary source port while the destination port was 22. I learned that the client chooses a source port for its connection, while the destination port identifies the service it is requesting.

The rule I configured blocks any IPv4 protocol and port to the selected protected destination. SSH on TCP/22 was the particular connection I used to test it. I need to keep the rule's configured scope separate from the narrower scope of my test.

### `LL-09` — A setting and a working control are different evidence

**Recorded result: Used for the OPNsense test.**

Seeing a rule in the interface told me it was configured. I needed more information to explain whether it actually stopped my connection.

I compared the intended rule, the Ubuntu result, and the firewall log. The matching source, destination port, action, rule label, and time made the result much clearer. I want to use the same approach for later host-firewall and monitoring tests: explain what should happen, generate that event, and check the result at the system responsible for it.

### `LL-11` — One blocked connection leaves other questions open

**Recorded result: Broader testing still needed.**

The SSH test showed that one selected IPv4 connection was blocked. It did not test every home-network destination, management service, protocol, or IPv6 route.

I now describe the successful test specifically instead of calling the whole environment fully isolated. The remaining work is recorded under `SB-06` and `SB-07` in the [security boundaries](security-boundaries.md): review the protected networks, check management access, and test additional representative traffic.

### `LL-12` — Two guests can communicate without going through OPNsense

**Recorded result: Architectural limitation identified; independent host testing remains.**

I learned that guests on the same `vmbr1` subnet can send traffic directly through the virtual switch. They do not normally send those local connections to their gateway.

That means putting two VMs behind OPNsense does not automatically filter their traffic to each other. UFW can filter traffic reaching Ubuntu. Separate subnets or additional routed interfaces may be useful later if an exercise needs OPNsense between groups of guests.

### `LL-13` — My IPv4 test says nothing conclusive about IPv6

**Recorded result: IPv6 review remains open.**

The OPNsense evidence was collected for IPv4. The saved interface view also shows WAN DHCPv6 state, the rules view includes an IPv6 allow rule, and Ubuntu's SSH output includes an IPv6 listener.

Those observations do not establish a working bypass, but they show why I cannot assume IPv6 is absent or protected by my IPv4 test. I still need to inspect the addresses, routes, and rules, then test an intentional IPv6 policy or a documented disabled configuration.

### `LL-19` — An empty Live View needs investigation

**Recorded result: Fresh test traffic produced matching log entries.**

At first, filtering OPNsense Live View for Ubuntu's lab address returned no entries. I learned that Live View shows recorded firewall events; opening the page does not generate traffic, and an empty filter is not a diagnosis.

I checked the current DHCP address, rule order, applied state, and logging, then kept auto-refresh on and generated another SSH attempt. Matching TCP/22 blocks appeared. Next time, I need to create a fresh event and look for it before deciding that the firewall or network is broken.

### `LL-20` — Temporary access needs a cleanup check every time

**Recorded result: LAB-02 cleanup recorded; latest LAB-03 cleanup not evidenced.**

My workstation was upstream, while the OPNsense web interface was on the lab side. I used a temporary `10.10.10.2/24` address on Proxmox's `vmbr1` and an SSH tunnel to reach it. The earlier setup record includes closing the tunnel, removing the address, and checking that the runtime IPv4 address was gone.

Later Ubuntu administration uses the same source again, and UFW now permits SSH from it. The earlier removal does not prove it was removed after the later session. I need a fresh cleanup check after each use and should keep the temporary address out of permanent bridge configuration.

### `LL-21` — Timestamps help connect events across systems

**Recorded result: Endpoint and firewall times correlated.**

Ubuntu used UTC even though I was in a different local time zone. The final SSH attempt was recorded at `2026-09-07T02:55:56Z`, and the associated firewall sequence started at `02:55:57`.

The close times, together with the connection fields, helped me match the two records. The `Z` identifies UTC. I need to retain the time zone when collecting evidence so a difference in clock display does not look like a different event.

## Ubuntu: learning how administration and host protection work

### `LL-10` — Closing PowerShell closes a connection

**Recorded result: Client and server roles clarified.**

I was concerned that closing the PowerShell window might stop or damage the Ubuntu server. I learned that the SSH client session is separate from the remote VM and its SSH service.

Closing the client disconnects that session. Closing a window that carries an SSH tunnel also closes that temporary route. The VM can still be running normally, so reconnecting may require restoring the tunnel first. I now need to distinguish the client, tunnel, remote service, and VM when diagnosing access.

### `LL-23` — SSH can work even when ssh.service says disabled

**Recorded result: Socket activation and effective SSH settings captured.**

The initial SSH checks were confusing because `ssh.service` was disabled for direct startup but active, while `ssh.socket` was enabled and active. I learned that systemd can hold the listening socket and activate the SSH service through it. Looking at one status line would have given me an incomplete picture.

The final [SSH artifact](../projects/03-ubuntu-server-baseline/evidence/04-ubuntu-services-and-ssh.txt) records valid configuration syntax, TCP/22 listeners, and public-key-only access for the administrative account. The setup notes also record a successful Ed25519 key login and rejected password attempts. Those client screenshots are not published; the committed text shows the resulting server settings.

### `LL-24` — UFW and OPNsense protect different parts of the path

**Recorded result: UFW policy captured; independent denial test still needed.**

OPNsense handled the earlier connection going from Ubuntu toward an upstream destination. UFW runs on Ubuntu itself, so it can also filter connections from another guest on the same lab subnet.

The [UFW output](../projects/03-ubuntu-server-baseline/evidence/05-ubuntu-ufw-status.txt) shows an active firewall, default-deny incoming policy, and an SSH allow rule for `10.10.10.2`. That is the temporary Proxmox source used for forwarding my workstation connection.

A second lab VM with a different address should not automatically pass that rule. The next test needs an allowed management connection and independent blocked traffic, with logs showing which firewall acted. I have not completed that test yet.

### `LL-25` — A baseline records a particular state, including unfinished updates

**Recorded result: Accounts and update state captured; broader review remains.**

I created separate administrative and standard accounts and checked their groups. The recorded standard account is outside the sudo group, while the administrative account belongs to it. This is a useful first privilege check, but group membership alone is not a complete audit of effective permissions.

The update record also lists two audit-library packages deferred by Ubuntu's phased rollout, with no reboot required at capture time. I kept that detail instead of saying the system had no pending updates. Those September 8 results describe that capture; a later maintenance check may produce different results.

## Recovery and documentation habits I am developing

### `LL-16` — A snapshot still depends on the original storage

**Recorded result: Recovery work remains planned.**

I plan to use a Proxmox snapshot before making a controlled change, then test whether rollback restores the expected state. The snapshot and rollback evidence have not been collected yet.

A snapshot would help undo an experiment, but it still depends on the VM's storage. An independent backup needs a separate destination and a restore test. I want to demonstrate both eventually, and I need to avoid using the terms interchangeably.

### `LL-17` — Planned work should stay visibly planned

**Recorded result: Ongoing documentation practice.**

The roadmap includes Windows, Active Directory, Wazuh, a simulated clinical laboratory, Kali, and vulnerable targets. Those are learning goals; the current project folders cover Proxmox, OPNsense, and Ubuntu.

I track the technical state separately from the write-up. Ubuntu can have working SSH and UFW while the overall project is still in progress. I need to update the overview when individual steps are completed without suggesting that the remaining service, firewall, and recovery tests are finished too.

### `LL-22` — Redaction should leave enough detail to explain the result

**Recorded result: Published evidence uses redactions and role labels.**

The raw captures included addresses and identifiers that were not needed publicly. I removed or obscured those values while keeping the useful context: bridge names, lab-only addresses, timestamps, rule actions, and destination ports.

The LAB-02 record includes text and OCR-assisted publication checks. Those checks belong to that earlier evidence review; editing these notes does not constitute a new full image-sanitization audit. For new captures, I still need to inspect the actual file and use consistent replacements across related evidence.

## The troubleshooting process I am practicing

| Question | Example from this lab |
|---|---|
| What am I trying to make happen? | Block a connection from Ubuntu to the selected upstream destination |
| What did I observe? | The connection timed out |
| Where should the decision happen? | OPNsense, because this test crosses the routed boundary |
| What else can explain the result? | An unreachable service can also produce a timeout |
| What evidence can narrow it down? | A matching firewall block with the expected connection fields and time |
| What should I change or record? | Apply the needed correction, retest, and keep the result with its limitations |

I am trying to make one understandable change at a time and record why it helped. That should make it easier to return to the lab after a break and explain the work in my own words.

## What I still need to practice

- Complete the Ubuntu service, listening-port, group/permission, and authentication-log review.
- Test UFW from an independent lab endpoint using the current source restriction.
- Create and test the Ubuntu snapshot and rollback process.
- Confirm removal of the latest temporary management address and tunnel.
- Complete the protected-network, management-access, and IPv6 checks before introducing vulnerable targets.
- Test startup order after a host restart and establish an independent backup/restore process.

## Change log

| Date | Change |
|---|---|
| 2026-09-25 | Rewrote the lessons as first-person learning notes, kept the original lesson IDs, and added evidence-backed Ubuntu lessons and current limitations. |
| 2026-09-07 | Added the LAB-02 troubleshooting, temporary-access cleanup, timestamp correlation, and evidence-redaction lessons. |
| 2026-09-03 | Corrected OPNsense adapter mapping and clarified the changing WAN lease. |
| 2026-09-02 | Created the initial lessons-learned record. |
