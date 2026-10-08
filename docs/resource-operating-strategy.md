# Resource Operating Strategy

> **Status:** Proposed operating plan; capacity calculations only\
> **Prepared:** 2026-10-08 (UTC)\
> **Platform:** Dell OptiPlex 7090 SFF, Intel Core i7, 32 GB installed RAM\
> **Live validation:** Not performed; no host or guest settings changed by this documentation update

## What I recommend starting with

The 32 GB host can support the next exercises if I run one selected group of guests at a time. I will keep OPNsense at 4 GiB, Windows 11 at 8 GiB, and the proposed Wazuh server at 8 GiB. Windows Server starts at the planned 6 GiB. Ubuntu is budgeted at its documented 4 GiB upper value until I verify its configuration.

My initial planning allowance is **4 GiB for Proxmox, virtualization overhead, and storage caching, plus a separate 4 GiB safety margin**. The saved Proxmox summary reports **31.08 GiB usable**, leaving a calculated 23.08 GiB guest budget. I would start with **at most 22 GiB of active guests**, rounded down for the proposed 2 GiB allocation increments. This is a planning ceiling, not a measured safe limit or a Proxmox-enforced reservation. Confirm current usable RAM before applying it.

Windows hardening and Active Directory administration fit comfortably on paper. Wazuh exercises should initially monitor Linux, a standalone Windows endpoint, or the domain controller in separate sessions. Running OPNsense, Windows 11, Windows Server, and Wazuh at their original sizes takes 26 GiB of guest RAM; adding Ubuntu takes 30 GiB. Neither is a routine 32 GB operating mode under this budget.

I would move to **64 GB before routinely combining the Windows endpoint, AD, Wazuh, and a clinical application VM**. Ballooning and small containers can improve efficiency, but they do not make that whole stack fit predictably today.

## What the repository actually establishes

This plan supplements the [architecture](architecture.md) and [roadmap](../ROADMAP.md). It does not promote any implementation or validation status.

| Item | Source and current evidence boundary |
|---|---|
| Host | The architecture records a dedicated 7090 SFF, Core i7, 32 GB RAM, local SSD storage, and one standalone Proxmox node. The [saved summary screenshot](../projects/01-proxmox-foundation/evidence/01-node-summary.png) shows **16 logical CPUs and 31.08 GiB total usable RAM**. Exact CPU model/physical-core count, DIMM population, and present RAM/storage state need a fresh inventory. |
| Historical load | That summary shows two running VMs, 7.74 GiB aggregate memory use, and 6% CPU at one moment. It is not a host-only baseline, workload peak, or tested multi-VM capacity limit. |
| Proxmox | The [saved version capture](../projects/01-proxmox-foundation/evidence/05-pveversion.txt) records PVE 9.2 components. It is a dated capture, not verification of the running version today. |
| OPNsense `100` | Recorded as 2 vCPU, 4 GiB, ballooning disabled. The earlier order-1/30-second startup note has not been validated through a full host restart. |
| Ubuntu `101` | The hardware record shows `2.00 GiB / 4.00 GiB`. A 2 GiB balloon minimum and 4 GiB maximum is a hypothesis to check with `qm config 101`, not a confirmed interpretation. |
| Windows `102` | [Open draft PR #5](https://github.com/dalen-green/proxmox-cybersecurity-homelab/pull/5) records an installed endpoint at 4 vCPU and 8 GiB with ballooning disabled. Its [reviewed project write-up](https://github.com/dalen-green/proxmox-cybersecurity-homelab/blob/b016974c1e8721b2dfa2a2bf603d2bf26190bb71/projects/04-windows-endpoint-security/README.md) still leaves hardening and runtime checks open. `main` has not yet incorporated that progress. |
| AD, Wazuh, clinical services | Planned. Their resource assignments below are proposals; no measured load or successful deployment is implied. Future VM IDs must come from the actual inventory. |

## Memory budget and starting assignments

### How the arithmetic works

All memory calculations below use **GiB**; Proxmox memory settings use MiB. Thus 2/4/6/8 GiB correspond to 2048/4096/6144/8192 MiB. The host's advertised 32 GB of installed DIMMs is treated as nominally 32 GiB for planning, but firmware and other reservations can make Linux `MemTotal` lower. Do not apply a disk-capacity GB-to-GiB conversion to the DIMM label.

| Budget item | Initial allowance | What it represents |
|---|---:|---|
| Host OS and PVE services | 2 GiB | Planning component, informed by Proxmox's host requirements; not a measurement |
| QEMU/device overhead and host storage caches | 2 GiB | Additional estimate across the selected guests; varies with guest count and workload |
| Total host-side allowance, `H` | **4 GiB** | Includes the two rows above; it is added only once |
| Separate safety margin, `R` | **4 GiB** | Space for bursts, estimation error, and responsive administration; not assigned to a VM |
| Nominal guest ceiling before reservations | 24 GiB | `32 - H - R`; this overstates the budget relative to the saved usable total |
| Ceiling using saved usable RAM | **23.08 GiB** | `31.08 - H - R`; confirm against a fresh inventory |
| Practical starting guest ceiling | **22 GiB** | Round down for the proposed 2 GiB allocation increments |

The initial admission calculation is:

```text
M = usable host MemTotal in GiB
G = sum of configured maximum RAM for all running VMs
    + memory limits of all running LXC containers
H = 4 GiB provisional host-side allowance
R = 4 GiB separate safety margin

Admit a proposed session only when G + H + R <= M.
Remaining capacity in the session tables = 31.08 - G - H.
```

The 31.08 GiB input is a saved observation, not a live reading. Recalculate if a fresh inventory differs. At that recorded capacity, 24 GiB of guests plus 4 GiB overhead leaves only 3.08 GiB, below the separate 4 GiB margin. Those combinations need a smaller validated configuration or a revised allowance supported by measurements. Do not quietly spend the safety margin to make the arithmetic work.

Proxmox [documents host memory separately from guest allocations](https://github.com/proxmox/pve-docs/blob/master/pve-system-requirements.adoc). My 4 GiB allowance is a lab planning choice, not a vendor sizing guarantee. The recorded VM store is `local-lvm`; this budget assumes the documented local-storage design, with no Ceph deployment or established ZFS pool. If storage changes, recalculate caching/ARC requirements from the actual configuration. Installed ZFS packages alone do not establish use of ZFS.

### Proposed guest assignments

| Guest | Starting vCPU | RAM budget | Memory policy | Why keep it this way |
|---|---:|---:|---|---|
| OPNsense `100` | 2 | 4 GiB | Fixed; retain ballooning disabled | Preserve the gateway and its network services. IDS/IPS or other heavy additions require a new budget. |
| Ubuntu baseline `101` | 2 | Up to 4 GiB | Verify current settings first; optional 2–4 GiB balloon range afterward | Keep the full Linux administration, firewall, boot, and recovery environment. Count 4 GiB even if it is idle or ballooned smaller. |
| Windows endpoint `102` | 4 | 8 GiB | Fixed initially | Retain room for updates, Defender, Sysmon, domain work, and a Wazuh agent. |
| Windows Server / DC | 2 initially; raise to the previously proposed 4 only if needed | 6 GiB | Fixed | Start with one small AD DS/DNS server. The 2-vCPU proposal is an alternative to the architecture's 4-vCPU starting estimate, not a recorded change. |
| Small DC trial | 2 | 4 GiB | Fixed, separate validation required | Optional AD DS/DNS-only configuration, preferably Server Core if it fits the learning goals. This is a lab sizing hypothesis, not a guaranteed sufficient allocation. No application/database roles on the DC. |
| Wazuh all-in-one | 4 | 8 GiB | Fixed | One dedicated Linux VM for central components. Do not count on balloon reclamation from its indexing workload. |
| Clinical application + small database | 2 initially | 4 GiB placeholder | Fixed VM initially | A tiny synthetic-data application; confirm the selected software's requirements before deployment. A larger database or interface engine changes the budget. |
| Kali | 2 | 4 GiB | Fixed; off outside its exercise | Preserve a separate OS boundary for authorized testing. |
| Benign support LXC | 1 initially | 0.5–1 GiB limit | Unprivileged; count the full limit | Optional documentation site or simple log relay. This is extra capacity if newly added, not a free service. |

Wazuh's [quickstart](https://documentation.wazuh.com/current/quickstart.html) recommends 4 vCPU, 8 GiB RAM, and 50 GB storage for its smallest 1–25-agent all-in-one tier. That supports the existing proposal; it does not establish performance on this host. The current quickstart lists Ubuntu 24.04 among supported choices, so use a supported OS in a new Wazuh VM rather than assuming the recorded Ubuntu 26.04.1 baseline is suitable for the central components. Check the selected release's requirements again at installation.

Agents run inside the endpoints and are included in their budgets. They do not each need an additional VM. Wazuh indexing, retention, scans, and event volume still need measurement even with only a few agents.

## Resource-allocation matrix

**Every row is calculated and untested.** These tables use the saved **31.08 GiB usable total** and `H = 4 GiB`. “Remaining” includes the separate 4 GiB safety margin and is not all available for another guest. A positive remainder alone does not qualify a session. A fresh `MemTotal` admission check and workload tests still apply.

Keys: **O** = OPNsense; **U** = Ubuntu; **W** = Windows 11; **D** = domain controller; **Z** = Wazuh; **C** = clinical application VM. Numbers in parentheses are GiB. Kali, vulnerable targets, and optional support services remain off in every row unless a new budget explicitly includes them.

### Windows, identity, and monitoring sessions

| Mode | Must run together | Other core guests off | Guest RAM | Host estimate | Budgeted total | Remaining | Proposed use |
|---|---|---|---:|---:|---:|---:|---|
| P0: Normal host boot | O(4) | U, W, D, Z, C | 4 | 4 | 8 | 23.08 | Minimal automatic startup |
| E1: Endpoint hardening | O(4), W(8) | U, D, Z, C | 12 | 4 | 16 | 15.08 | Local accounts, updates, Defender, logging |
| E2: Endpoint firewall test | O(4), U(4), W(8) | D, Z, C | 16 | 4 | 20 | 11.08 | Ubuntu supplies the independent test source |
| A1: AD administration | O(4), D(6), W(8) | U, Z, C | 18 | 4 | 22 | 9.08 | Domain join, fresh logon, DNS, GPO, permissions |
| M1: Linux monitoring | O(4), U(4), Z(8) | W, D, C | 16 | 4 | 20 | 11.08 | Linux source event through Wazuh alert |
| M2: Local Windows monitoring | O(4), W(8), Z(8) | U, D, C | 20 | 4 | 24 | 7.08 | Standalone/local-account work, with no AD/DNS dependency |
| M3: DC monitoring | O(4), D(6), Z(8) | U, W, C | 18 | 4 | 22 | 9.08 | Generate and investigate DC events from its console |
| M4: Reduced-memory AD + monitoring trial | O(4), D(4), W(6), Z(8) | U, C | 22 | 4 | 26 | 5.08 | Conditional only: validate both reduced Windows allocations first |
| X1: Original AD + monitoring sizes | O(4), D(6), W(8), Z(8) | U, C | 26 | 4 | 30 | 1.08 | Outside the proposed routine budget |
| X2: All five original guests | O(4), U(4), W(8), D(6), Z(8) | C | 30 | 4 | 34 | -2.92 | Exceeds recorded usable RAM after estimated overhead |

M2 is not an AD validation shortcut. Once Windows or Wazuh uses the DC for required DNS or authentication, keep the DC running or use a deliberately separate standalone test configuration. Cached domain sign-in does not prove AD availability. Do not change DNS or roll back a domain member casually just to fit a row.

M4 is an optional later trial, not the initial setup. First test a 4 GiB DC in the AD session, then separately test a 6 GiB endpoint through Windows updates, Defender, domain logon, and telemetry. Only then test them together with Wazuh. Keeping W at 8 GiB while reducing only D gives 24 GiB of guests and still misses the initial margin. If either reduction fails, use the separate monitoring sessions whose dependencies fit, or upgrade RAM. Retain 8 GiB as the starting endpoint allocation.

### Clinical laboratory sessions

These phases use synthetic orders, specimens, and results. They preserve the roadmap's distinction between directory permissions and actual application authorization.

| Mode | Must run together | Other core guests off | Guest RAM | Host estimate | Budgeted total | Remaining | What the session can establish |
|---|---|---|---:|---:|---:|---:|---|
| C1: Application and identity | O(4), D(6), W(8), C(4) | U, Z | 22 | 4 | 26 | 5.08 | Workflow, application roles, and implemented AD integration; no live centralized monitoring |
| C2: Application and monitoring | O(4), W(8), C(4), Z(8) | U, D | 24 | 4 | 28 | 3.08 | Below the initial margin; reduce a validated allocation before use; local accounts and independent DNS only |
| C3: Server-side integration | O(4), D(6), C(4), Z(8) | U, W | 22 | 4 | 26 | 5.08 | Synthetic server-side jobs/API events and telemetry; excludes Windows client workflow validation |
| X3: Complete clinical stack | O(4), D(6), W(8), C(4), Z(8) | U | 30 | 4 | 34 | -2.92 | Use a larger host budget before testing the entire workflow together |

C1, C2, and C3 do not collectively prove that the full stack works simultaneously. C2 would fall to 22 GiB with a validated 6 GiB Windows endpoint, or a selected application proven to fit 2 GiB; neither saving is assumed. If the app always requires AD, C2 is unavailable in that configuration. Run C3 for server-side checks, then schedule a full-stack test after an upgrade. Do not place the clinical app or database on the DC to avoid a VM.

For recovery, substitute an isolated restored application VM for the active C VM where possible. If source and restored guests both need to run, add the clone's full RAM and storage to the budget first. Prevent duplicate addresses and identities from appearing on the live lab segment. An independent backup and functional restore remain requirements; snapshots alone do not satisfy them.

## Reducing contention without weakening the lab

### Ballooning and KSM

Proxmox [describes ballooning](https://github.com/proxmox/pve-docs/blob/master/qm.adoc) as guest-driver cooperation that can reclaim RAM; Windows needs the VirtIO balloon driver. The QEMU Guest Agent is a separate service. Neither a checked GUI option nor an installed guest agent proves ballooning works.

Ubuntu is the first candidate for a controlled 2–4 GiB experiment after confirming its current configuration. Compare the same workload with a fixed allocation and with ballooning, including memory pressure inside Ubuntu. Avoid aggressively ballooning the firewall, DC, or Wazuh. Keep Windows fixed while establishing its baseline; test any later change separately.

Count **maximums**, not minimums, in every admission calculation. Ballooning can increase guest paging or memory pressure and cannot guarantee timely reclamation during simultaneous demand. Proxmox's [node balloon target](https://github.com/proxmox/pve-docs/blob/master/pvenode.adoc) is a controller setting, not reserved host RAM. Do not raise it to squeeze an oversized mode into memory.

[KSM](https://docs.kernel.org/admin-guide/mm/ksm.html) can deduplicate some identical pages, but savings depend on workload and can disappear as guests write. Record any savings as observations and budget zero credit for them. Leave KSM policy as a separate reviewed setting; this plan does not change the existing Windows “Allow KSM” choice. Host swap is also not added to usable RAM or treated as an operating-capacity upgrade.

### Where unprivileged LXC helps

An [unprivileged LXC container](https://github.com/proxmox/pve-docs/blob/master/pct.adoc) maps container root to an unprivileged host identity, but still shares the host's Linux kernel. It is useful for a trusted, small support service whose purpose is not OS security testing.

Start an optional static documentation site or basic log relay at 1 vCPU and a 512–1024 MiB RAM limit. Set and inspect its swap allowance as well; any swapping consumes host resources. Measure its peak before reducing the limit. Replacing a genuinely equivalent 2 GiB support VM with a 1 GiB container reduces the configured budget by 1 GiB; actual savings remain to be measured.

Keep OPNsense, Windows, AD, the Ubuntu baseline, Kali, and vulnerable targets as full VMs. Keep Wazuh in its own VM for predictable sizing and isolation. The clinical app should also start in a VM, particularly for application compromise, interface testing, and recovery exercises. Application containers can run inside that VM when useful. Do not enable privileged containers, broad host mounts, or unnecessary device access to chase memory savings.

### CPU, storage, and session discipline

The saved summary reports 16 logical CPUs. It does not establish the exact i7 model or physical-core count; verify those before selecting CPU limits. Under the proposed 2-vCPU DC configuration, A1 exposes 8 vCPUs and M4 exposes 12. They are scheduling opportunities, not dedicated physical cores, and 16 hardware threads are not equivalent to 16 physical cores. These counts suggest memory will constrain the proposed small sessions first, but that is an inference; scans and indexing can still saturate CPU or storage. More vCPUs do not automatically make a lightly loaded DC faster.

Use these operating habits first:

1. Pick one matrix row and check what is already running, including containers and restore clones. Fully shut down unrelated guests; a paused/suspended VM can still occupy RAM unless its memory is explicitly saved to disk and released.
2. Start infrastructure before clients and wait for services to become ready. Avoid a simultaneous boot storm.
3. Stagger Windows updates, Defender scans, Wazuh inventory/indexing bursts, package upgrades, and backups. Do not disable the protections being studied to improve a benchmark.
4. Watch host CPU, guest responsiveness, disk latency/I/O wait, and thin-pool space alongside RAM. Keep the initial Wazuh deployment and VM disks on the available SSD storage, subject to capacity checks.
5. Review Wazuh retention and event volume. Its planned 50 GB disk is a starting allocation, not an assurance that every logging configuration fits. Stop unnecessary test-event generation if indexing falls behind.

Before each larger session, inspect `local` free space and `local-lvm` **data and metadata** utilization. As a proposed lab guardrail, investigate either thin-pool percentage at 80% and postpone new VMs, snapshots, restores, or bulk updates until capacity is addressed. Thin provisioning does not create physical space. These storage thresholds are operating choices, not failure thresholds measured on this SSD.

If CPU or disk pressure persists, reduce concurrent work before changing scheduling weights. Once a specific contention problem is measured, consider reducing the CPU limit of the batch/test guest; keep gateway and identity services responsive. Avoid CPU pinning or speculative tuning before there is evidence to justify it.

## Startup dependencies and shutdown behavior

The default after a host restart should be **P0: OPNsense only**. Other guests remain manual-start while this is a 32 GB learning lab. This prevents a saved “start at boot” flag from recreating X2. Before a planned AD/monitoring session, start only its selected group.

| Guest/role | Proposed order | Initial delay after its start | Initial shutdown timeout | Default start at boot | Readiness check before dependents |
|---|---:|---:|---:|---|---|
| OPNsense `100` | 1 | 60 seconds | 180 seconds | Yes, after configuration review | LAN address, required DHCP/DNS service, permitted routed connectivity |
| AD DS/DNS | 2 | 120 seconds | 300 seconds | No | AD DS/DNS services, domain SRV lookup, time synchronization |
| Wazuh | 3 | 120 seconds | 300 seconds | No | Central services healthy, dashboard/API usable, ingestion available |
| Clinical app/database VM | 4 | 60 seconds | 300 seconds | No | Database and app ready; required identity service reachable |
| Ubuntu/Windows clients | 5 | 30 seconds between starts | 180/300 seconds | No | Guest network, time, sign-in, and any agent connection |
| Kali/disposable targets | 6 | 30 seconds between starts | 180 seconds | No | Start only for their separate authorized exercise |

All timings are starting hypotheses, not measured boot times. The proposed 60 seconds for OPNsense is a deliberate trial increase from the older 30-second note. Retain the old setting in the change record so it can be compared.

Proxmox's [startup documentation](https://github.com/proxmox/pve-docs/blob/master/qm.adoc) describes order, delay before subsequent starts, and reverse ordering at shutdown. These settings do not test service health or automatically start a dependency when I manually start one VM. “Boot order” inside a VM selects disks and other boot devices; it is a different setting.

A shutdown timeout is not a promise of a clean shutdown: Proxmox may force-stop a guest that has not exited in time. Verify graceful shutdown using the supported guest-agent or ACPI path before the host-restart test, and allow more time if actual Windows or database shutdowns require it.

The table gives a consistent single-host sequence, but the real dependencies matter more than the numbers:

- OPNsense provides the normal routed path and existing network services; same-subnet traffic on `vmbr1` does not require routing through it.
- A domain member needs AD DNS and a reachable DC for fresh domain authentication and policy. Following [Microsoft's DNS guidance](https://learn.microsoft.com/en-us/troubleshoot/windows-server/networking/best-practices-for-dns-client-settings), point domain clients at the AD DNS server. Configure its external forwarding deliberately; do not add a public/OPNsense resolver as an alternate client DNS server that lacks the AD zone. Avoid a forwarding loop.
- Wazuh is not intrinsically dependent on AD. It can start after OPNsense in a standalone session. If its configured DNS or authentication uses AD, the DC becomes a prerequisite. Keep infrastructure addressing and names stable without making Proxmox management depend on a lab guest.
- Start Wazuh before generating events intended for monitoring. Starting the collector later does not prove complete recovery of missed events; agent buffers and retention have limits. Shutdown ordering keeps it up after application/client guests, but the DC stops after Wazuh in this simple sequence, so DC shutdown-event coverage is not guaranteed.

For manual shutdown, stop clients and test systems, then the app, Wazuh, AD, and OPNsense last. Use graceful shutdown and investigate timeouts. If the exercise specifically measures DC shutdown telemetry, arrange Wazuh's DNS/authentication independence and an explicit alternate sequence for that test.

## How I will validate this in Proxmox

These are instructions to run on the actual lab during a maintenance session. They have not been executed as part of this repository update. Keep raw configuration output private and publish only reviewed, sanitized evidence.

### 1. Capture the starting configuration

In the Proxmox UI, review **Node → Summary**, **Node → Disks/storage**, each VM's **Hardware → Memory/Processors**, and **Options → Start at boot / Start-Shutdown order**. Check pending configuration changes as well as the running state. Record guest IDs before using commands for planned guests.

The following read-only commands give a starting inventory in the node shell:

```bash
date -Is
pveversion -v
lscpu
free -h
awk '/MemTotal/ {printf "Usable host RAM: %.2f GiB\n", $2/1048576}' /proc/meminfo
qm list
pct list
qm config 100
qm config 101
qm config 102
pvenode config get
pvesm status
lvs -a -o lv_name,lv_size,segtype,data_percent,metadata_percent
df -h
swapon --show
```

Run `qm config 102` only after confirming that ID still belongs to the Windows endpoint. Repeat `qm config` for each actual future VM and `pct config` for each container. If the installed version differs, use its local help before applying settings.

For Ubuntu, record `memory`, `balloon`, `shares`, and any pending values. `balloon: 0` disables the device; an absent entry should be interpreted using the installed Proxmox defaults, not assumed to mean either zero or a 2 GiB minimum. Compare the saved settings with guest `free -h` and the PVE runtime view. Check the balloon driver in Linux with `lsmod | grep virtio_balloon` or the appropriate driver/sysfs view; absence from `lsmod` alone does not rule out a built-in driver. Do not claim reclamation worked until a controlled comparison demonstrates it.

Review the DIMM slots privately with `dmidecode --type 17` or BIOS inventory before planning an upgrade. That output can contain serial numbers and asset information, so do not commit it unedited.

### 2. Establish host and workload measurements

First measure the host with optional guests stopped; if practical, include a brief host-only baseline using the trusted `vmbr0` management path. Then measure OPNsense-only P0. Follow with the chosen exercise, collecting approximately 10 minutes of settled idle behavior, a 30–60-minute representative task, and one restart/update or ingestion burst. These durations are a proposed sampling plan, not results.

Use the node Summary graphs and guest tools alongside these read-only samples:

```bash
vmstat 5 60
cat /proc/pressure/memory
cat /proc/pressure/cpu
cat /proc/pressure/io
journalctl -k -b --no-pager | grep -Ei 'out of memory|oom-kill|killed process'
```

`vmstat` runs for about five minutes; its first line reports averages since boot. Inspect subsequent samples for active `si`/`so` swap traffic and I/O wait. A small historical amount of allocated swap is different from ongoing swap activity. Record `MemAvailable` and pressure repeatedly during the task, not just at idle. Do not add host “used RAM,” guest “used RAM,” cache, and process RSS together as if they were disjoint totals.

Inside Windows, inspect Task Manager/Resource Monitor available memory, committed memory versus commit limit, paging, CPU, and disk response. In Linux, use `free`, `vmstat`, and service logs. Include Wazuh ingestion delay, indexer/service health, endpoint agent status, and timestamps. If available, use `iostat -xz` for disk latency; it may require installing `sysstat` separately.

The first workload tests should be E1/E2 and A1, then M1/M3, and M2 only when its standalone dependencies are satisfied. M4 and the clinical rows remain conditional until the smaller configurations and application requirements have their own evidence.

### 3. Apply the pass/review/stop criteria

| Observation | Proposed decision |
|---|---|
| Configured maximums fit the actual `M - H - R` budget; representative tasks pass; roughly 4 GiB or more host `MemAvailable` remains during peaks | Candidate for a validated operating mode; retain measured evidence and version/workload scope |
| `MemAvailable` falls below 4 GiB for about a minute, recurring memory pressure appears, guest paging grows, or log ingestion falls behind | Do not start another guest; investigate and reduce the session before calling it validated |
| `MemAvailable` falls below 2 GiB, sustained swap-in/out accompanies stalls, a host/guest OOM occurs, or essential services fail | Stop the workload and gracefully shut down an optional guest while management remains responsive; mark the mode failed |
| CPU is persistently saturated or disk latency/I/O wait harms sign-in, updates, or ingestion even with adequate RAM | Treat the mode as contention-limited; stagger tasks or reduce concurrent work, then retest |

These thresholds are conservative lab guardrails, not Proxmox guarantees. Low “free” memory caused by reclaimable cache is not by itself exhaustion, and acceptable host graphs do not rule out a balloon-starved guest. Use host and guest evidence together. Revise `H` upward if measurements justify it; never subtract active guest pages twice while estimating overhead.

### 4. Test a real host restart

1. Record all running guests, onboot flags, order/delay/timeout settings, pending changes, and the intended matrix mode. Ensure a recoverable baseline and access to the OptiPlex's physical console. Normal management must remain on `vmbr0` without relying on the DC or OPNsense.
2. In **VM → Options**, configure P0: OPNsense onboot enabled, optional guests disabled. Set the proposed startup order values after reviewing the existing settings. This document supplies proposed values, not an already-applied configuration.
3. Use a planned graceful **Proxmox node Reboot**, not just a guest restart or a power pull. Observe the node task log and guest shutdown results. After boot, confirm only OPNsense auto-started and measure readiness rather than assuming that 60 seconds was sufficient.
4. Verify Proxmox management, bridge membership, and absence of the temporary `10.10.10.2/24` host address on `vmbr1`. Test the required gateway, DHCP/DNS, and permitted routed path from a selected endpoint. Recheck the relevant firewall restrictions.
5. Separately, to validate automatic multi-guest ordering, temporarily enable onboot for **one admitted mode**, such as A1, and repeat a graceful full host restart. Capture start timestamps and service-ready times. During monitoring tests, confirm a fresh event reaches Wazuh after restart. Do not enable every guest just to test order.
6. For A1, verify an AD SRV lookup, time synchronization, fresh domain authentication, and `gpupdate /force` plus `gpresult /r` on Windows. A cached logon or ping alone is insufficient. For clinical sessions, also verify expected synthetic records and app authorization.
7. Restore P0's onboot flags after the multi-guest test and verify the resulting configuration. Record which automatic mode actually passed. If timing fails, increase the relevant delay and retest; a delay still is not a service-health gate.

Check the node task history and, where retained, `journalctl -u pve-guests -b`, `journalctl -u pve-guests -b -1`, and previous-boot kernel logs for order, timeouts, or OOM events. If previous-boot logs are unavailable, collect equivalent task/console evidence during the next test. Do not mark `ARC-VAL-08` passed until the full host-restart evidence exists and its scope is documented.

## Is a RAM upgrade worthwhile?

**Yes, 64 GB materially improves concurrent AD, monitoring, and clinical work. It is not required to finish the initial endpoint and AD exercises.** Dell's [7090 SFF memory specifications](https://www.dell.com/support/manuals/en-us/optiplex-7090-sff/d11_optiplex_7090_sff_ss/memory?guid=guid-e9d2ea83-38a4-431d-803c-96d63c1dbc34&lang=en-us) list four DDR4 UDIMM slots and up to 128 GB. This applies to the SFF, not the Micro model. Compatible speed and population depend on the CPU and Dell's supported configuration.

| Installed RAM | Illustrative guest combination | Guest RAM | Host allowance | Remaining | Interpretation |
|---|---|---:|---:|---:|---|
| 32 GB | O + W + D at original sizes | 18 GiB | 4 GiB | 9.08 GiB | Uses saved 31.08 GiB usable; suitable starting AD session on paper |
| 32 GB | O + U + W + D + Z | 30 GiB | 4 GiB | -2.92 GiB | Uses saved 31.08 GiB usable; does not fit this plan |
| 64 GB | O + U + W + D + Z | 30 GiB | 6 GiB | 28 GiB | Substantial room for bursts; still needs testing |
| 64 GB | All five above + C(4) + Kali(4) | 38 GiB | 6 GiB | 20 GiB | Capacity illustration only; Kali still runs only when needed |
| 128 GB | Same 38 GiB illustration | 38 GiB | 8 GiB | 82 GiB | More than the current plan calls for; CPU and SSD limits remain |

The 64/128 GB rows use nominal capacities because those upgrades have not happened. The larger host allowances are planning choices for more concurrent activity, not measurements or a rule that PVE overhead scales with installed RAM. Preserve at least the separate 4 GiB safety margin, and recalculate using actual `MemTotal` after an upgrade. Extra host RAM is not automatically added to the guests.

Check the current DIMM population first. If the existing 32 GB is 2×16 GB, a compatible additional pair may allow 4×16 GB; if it is 4×8 GB, modules need replacing. A supported 2×32 GB kit preserves expansion room. Confirm the exact CPU, module specifications, and [Dell population matrix](https://www.dell.com/support/manuals/en-us/optiplex-7090-sff/d11_optiplex_7090_sff_ss/memory-configuration-matrix?guid=guid-17d702af-3ef5-4e11-bfdb-fcaa1f41dd7d&lang=en-us), then run hardware memory diagnostics and repeat the workload tests. This is a capacity recommendation, not a priced shopping recommendation.

A larger SSD or separate NAS can address storage needs, but neither supplies RAM to this host. A second compute host would add capacity at the cost of another management and recovery path; it is unnecessary for the initial exercise groups.

## Measurement record to complete

No new measurements were collected for this plan. The saved screenshot supplies only the historical observations identified above. Add dated, sanitized evidence and leave unavailable values as **Not collected**. Do not convert estimates into results by copying numbers from the allocation tables.

| Check | Evidence to collect | Current result |
|---|---|---|
| `RES-01`: Host inventory | CPU model/cores/threads, DIMM population without serials, usable RAM, PVE version, storage types/free space | Partial historical evidence: 16 logical CPUs and 31.08 GiB usable; fresh inventory not collected |
| `RES-02`: Memory configuration | Current/pending memory, balloon, shares, onboot, startup, and guest-driver state; resolve Ubuntu's two values | Not collected |
| `RES-03`: Host allowance | Host-only/P0 baseline, cache/overhead observations, swap and memory-pressure samples | Not collected |
| `RES-04`: Operating modes | One record per tested mode with workload, duration, guest maximums, peak host/guest pressure, CPU/disk behavior, functional outcome | Not collected |
| `RES-05`: Restart dependencies | P0 and one selected multi-guest full-host reboot: task logs, readiness times, DNS/time/logon/policy/telemetry checks | Not collected; `ARC-VAL-08` remains open |
| `RES-06`: Optional optimizations | Repeatable fixed-vs-balloon comparison, equivalent support VM-vs-LXC workload, or before/after RAM-upgrade test | Not collected; optional |

For each `RES-04` run, record: **date and versions; mode; guest IDs/max RAM; usable host RAM; minimum host MemAvailable; peak guest memory/commit; swap activity and memory pressure; CPU/I/O symptoms; Wazuh ingestion delay if relevant; functional checks; pass/fail and limitations; evidence links**. Retain failed runs and the changes made before a retest.

## Reference notes

Vendor guidance was reviewed on 2026-10-08 UTC. The linked upstream documentation can change; compare it with the installed versions when implementing the plan. Allocations, margins, timings, guardrails, and mode choices in this document are proposed lab decisions. The repository and open PR establish only the saved configuration and progress explicitly attributed to them.

- [Proxmox system requirements](https://github.com/proxmox/pve-docs/blob/master/pve-system-requirements.adoc)
- [Proxmox VM memory and startup behavior](https://github.com/proxmox/pve-docs/blob/master/qm.adoc)
- [Proxmox node options](https://github.com/proxmox/pve-docs/blob/master/pvenode.adoc)
- [Proxmox container behavior](https://github.com/proxmox/pve-docs/blob/master/pct.adoc)
- [Linux Kernel Samepage Merging](https://docs.kernel.org/admin-guide/mm/ksm.html)
- [Wazuh quickstart and sizing](https://documentation.wazuh.com/current/quickstart.html)
- [Microsoft AD/DNS client recommendations](https://learn.microsoft.com/en-us/troubleshoot/windows-server/networking/best-practices-for-dns-client-settings)
- [Dell 7090 SFF memory specifications](https://www.dell.com/support/manuals/en-us/optiplex-7090-sff/d11_optiplex_7090_sff_ss/memory?guid=guid-e9d2ea83-38a4-431d-803c-96d63c1dbc34&lang=en-us)
