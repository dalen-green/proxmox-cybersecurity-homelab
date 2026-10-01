# Lessons Learned

> **Document status:** Living learning notes\
> **Last updated:** 2026-09-30\
> **Current work:** Windows endpoint setup and hardening, building on Proxmox, OPNsense, and Ubuntu

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

**Recorded result: Same-subnet limitation identified; Ubuntu host testing is recorded, and Windows testing remains.**

I learned that guests on the same `vmbr1` subnet can send traffic directly through the virtual switch. They do not normally send those local connections to their gateway.

That means putting two VMs behind OPNsense does not automatically filter their traffic to each other. UFW can filter traffic reaching Ubuntu. Separate subnets or additional routed interfaces may be useful later if an exercise needs OPNsense between groups of guests.

The Ubuntu source-specific test is now recorded under `LL-24`. Windows has joined the same bridge, so its own firewall still needs an independent test in LAB-04.

### `LL-13` — My IPv4 test says nothing conclusive about IPv6

**Recorded result: IPv6 review remains open.**

The OPNsense evidence was collected for IPv4. The saved interface view also shows WAN DHCPv6 state, the rules view includes an IPv6 allow rule, and Ubuntu's SSH output includes an IPv6 listener.

Those observations do not establish a working bypass, but they show why I cannot assume IPv6 is absent or protected by my IPv4 test. I still need to inspect the addresses, routes, and rules, then test an intentional IPv6 policy or a documented disabled configuration.

### `LL-19` — An empty Live View needs investigation

**Recorded result: Fresh test traffic produced matching log entries.**

At first, filtering OPNsense Live View for Ubuntu's lab address returned no entries. I learned that Live View shows recorded firewall events; opening the page does not generate traffic, and an empty filter is not a diagnosis.

I checked the current DHCP address, rule order, applied state, and logging, then kept auto-refresh on and generated another SSH attempt. Matching TCP/22 blocks appeared. Next time, I need to create a fresh event and look for it before deciding that the firewall or network is broken.

### `LL-20` — Temporary access needs a cleanup check every time

**Recorded result: LAB-02 and latest LAB-03 cleanup checks completed.**

My workstation was upstream, while the OPNsense web interface was on the lab side. I used a temporary `10.10.10.2/24` address on Proxmox's `vmbr1` and an SSH tunnel to reach it. The earlier setup record includes closing the tunnel, removing the address, and checking that the runtime IPv4 address was gone.

Later Ubuntu administration used the same source again because UFW permits SSH from it. After the final LAB-03 evidence transfer, I stopped the tunnel, removed `10.10.10.2/24`, and confirmed that `vmbr1` displayed no IPv4 address. The important lesson is that each temporary session needs its own cleanup check; an earlier successful removal cannot prove a later runtime state.

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

**Recorded result: Source-specific IPv4 UFW behavior tested.**

OPNsense handled the earlier connection going from Ubuntu toward an upstream destination. UFW runs on Ubuntu itself, so it can also filter connections from another guest on the same lab subnet.

The [UFW output](../projects/03-ubuntu-server-baseline/evidence/05-ubuntu-ufw-status.txt) shows an active firewall, default-deny incoming policy, and an SSH allow rule for `10.10.10.2`. That is the temporary Proxmox source used for forwarding my workstation connection.

The [independent UFW artifact](../projects/03-ubuntu-server-baseline/evidence/07-ubuntu-ufw-independent-test.txt) compares two sources against the same SSH listener. Ubuntu accepted a key-authenticated connection from permitted source `10.10.10.2` and logged UFW blocks for TCP/22 from the OPNsense LAN endpoint at `10.10.10.1`. Using one confirmed listening service made the difference easier to attribute to the source-specific UFW rule, so I did not need to create an extra temporary listener.

### `LL-25` — A baseline records a particular state, including unfinished updates

**Recorded result: Intended sudo-role review and final update capture completed.**

I created separate administrative and standard accounts, checked their groups, and reviewed their effective sudo results. The administrative role can deliberately elevate through sudo, while the standard role is explicitly denied. I also removed the administrative account's unused `lxd` membership after confirming LXD was not installed. This validates the intended local sudo separation, but it is not a universal audit of every Linux permission mechanism.

The final update record lists `python3-distupgrade` and `ubuntu-release-upgrader-core` as deferred by Ubuntu's phased rollout, with no reboot required at capture time. I kept that detail instead of forcing phased packages merely to make the pending count read zero. The September 29 result is still a point-in-time record; a later maintenance check may differ.

## Windows: learning how the guest sees its hardware

### `LL-26` — A missed boot prompt can look like a different problem

**Recorded result: Windows Setup launched after retrying the DVD prompt.**

The Windows VM displayed “Press any key to boot from CD or DVD,” then timed out and tried PXE network boot. I initially saw the later error rather than the earlier prompt.

I reset the VM, focused the console, and responded to the prompt. Setup then launched. That sequence helped me understand that the installation disc was reachable and that the network-boot fallback did not, by itself, show an OPNsense problem.

### `LL-27` — Virtual hardware still needs guest drivers

**Recorded result: The storage driver exposed the disk; Ethernet was shown as connected after guest-driver setup.**

Proxmox showed an 80 GiB disk, but Windows Setup initially showed no disk at all. I loaded the Windows 11 x64 VirtIO SCSI driver from the attached driver ISO, and the disk appeared. The virtual disk existed; Windows needed the driver for its controller.

Networking was a separate step. I used the offline option shown during setup, created a local account, and worked through the VirtIO guest-tools installer. The later [Windows Settings capture](../projects/04-windows-endpoint-security/evidence/04a-windows-network-status.png) reports Ethernet Connected. I still need a guest-agent and device review before describing the whole tools installation as verified.

### `LL-28` — Connected is a starting point for security checks

**Recorded result: Initial Windows network state captured; endpoint controls remain untested.**

The same Windows screen shows a Public network profile and a Local Account. Those labels are useful, but they do not tell me the account's privileges, the firewall's effective rules, or whether a standard account has been created.

I am carrying the Ubuntu lesson into Windows: record the setting, perform a specific check, and explain its limits. The [LAB-04 plan](../projects/04-windows-endpoint-security/README.md) starts with account-role separation and continues through updates, Defender, firewall testing, logs, and recovery.

## Recovery and documentation habits I am developing

### `LL-16` — A snapshot still depends on the original storage

**Recorded result: Snapshot creation and controlled rollback passed.**

I created the labeled `ubuntu-baseline-2026-09-29` [snapshot](../projects/03-ubuntu-server-baseline/evidence/08-proxmox-ubuntu-snapshot.png), added a harmless marker afterward, recorded its checksum, and rolled the powered-off VM back. The [post-rollback checks](../projects/03-ubuntu-server-baseline/evidence/09-ubuntu-rollback-validation.txt) show that the marker was absent while DHCP networking, the default route, SSH, and UFW returned to their expected states.

The test showed that the snapshot can undo this controlled change, but the snapshot still depends on the VM's original Proxmox storage. An independent backup needs a separate destination and its own restore test. Passing one does not prove the other.

### `LL-17` — Planned work should stay visibly planned

**Recorded result: Ongoing documentation practice.**

The current project folders cover Proxmox, OPNsense, Ubuntu, and the new Windows progress draft. Active Directory, Wazuh, the simulated clinical laboratory, Kali, and vulnerable targets remain learning goals in the roadmap.

I track the technical state separately from the write-up. Ubuntu now has all nine planned core artifacts reviewed and published, including source-specific UFW testing and controlled rollback. The project is technically verified for that documented IPv4 scope. IPv6, independent backup restoration, host-restart testing, and continuous compliance remain separate work rather than reasons to understate the completed baseline.

Windows has reached the desktop and reports Ethernet connectivity, but its security baseline remains in progress. The two initial screenshots support the setup observations; the remaining account, protection, logging, and recovery checks are still planned.

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

- Complete and validate the Windows account, maintenance, endpoint-protection, logging, and recovery baseline.
- Turn the individual Ubuntu checks into a repeatable baseline-audit script.
- Complete the protected-network, management-access, and IPv6 checks before introducing vulnerable targets.
- Test startup order and service recovery after a full host restart.
- Establish an independent backup on separate storage and perform a restore test.
- Repeat patch, listener, and firewall reviews over time instead of treating one baseline as permanent.

## Change log

| Date | Change |
|---|---|
| 2026-09-30 | Added Windows boot-prompt, VirtIO driver, and initial network-status lessons; kept the remaining hardening work visibly in progress. |
| 2026-09-29 | Recorded final LAB-03 sudo-role review, source-specific UFW testing, temporary-access cleanup, and controlled snapshot rollback. |
| 2026-09-28 | Updated LAB-03 progress and remaining practice to reflect the September 26 service/listener/authentication review. |
| 2026-09-25 | Rewrote the lessons as first-person learning notes, kept the original lesson IDs, and added evidence-backed Ubuntu lessons and current limitations. |
| 2026-09-07 | Added the LAB-02 troubleshooting, temporary-access cleanup, timestamp correlation, and evidence-redaction lessons. |
| 2026-09-03 | Corrected OPNsense adapter mapping and clarified the changing WAN lease. |
| 2026-09-02 | Created the initial lessons-learned record. |
