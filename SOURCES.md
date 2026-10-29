# Source audit

Results of checking every reference in this repo against Microsoft Learn on 2026-10-29. This file records what was checked, what was kept, what was removed, and why.

## How the check was done

1. **Reachability:** every cited URL was requested. All resolved (HTTP 200, after redirects).
2. **Anchors:** for each URL with a `#fragment`, the page HTML was searched for a matching `id`. Three anchors did not exist and were fixed (see below).
3. **Claims:** the primary Windows pages that carry the thresholds and definitions were read in full. Claims in the metric files were checked against them. A claim is **source-checked** only if the cited page supports it.

Not done: no live Windows machine was queried, and pages that were only reachability-checked are marked **partly checked** in the metric files.

## Primary Windows sources (read in full)

| Page | Used for | Result |
|---|---|---|
| [Troubleshoot issues using Performance Monitor](https://learn.microsoft.com/troubleshoot/windows-server/support-tools/troubleshoot-issues-performance-monitor) | CPU thresholds, % DPC Time, % Interrupt Time, CPU counter list, Handle Count | Supports the claims. Corrected: DPC and interrupt thresholds now use this page's values. |
| [Scenario guide: Troubleshoot performance problems in Windows](https://learn.microsoft.com/troubleshoot/windows-server/performance/troubleshoot-performance-problems-in-windows) | 18 core counters: disk latency, memory, CPU, network thresholds | Supports the claims. Corrected: disk latency thresholds replace the BizTalk values. |
| [Network-Related Performance Counters](https://learn.microsoft.com/windows-server/networking/technologies/network-subsystem/net-sub-performance-counters) | Network counter families, Output Queue Length, DPCs Queued/sec, RSC | Supports the claims. Corrected: the section anchors used in the original repo do not exist. |
| [Memory Performance Information](https://learn.microsoft.com/windows/win32/memory/memory-performance-information) | Memory counter to API structure mapping | Supports the claims. Used for commit, pool, process memory, and process and thread counts. |
| [Collecting Performance Data](https://learn.microsoft.com/windows/win32/perfctrs/collecting-performance-data) | `_Total` and `Process` counter behavior, `Process V2` | Supports the claims. Added: Process V2 pitfall. |
| [RAM, virtual memory, pagefile, and memory management](https://learn.microsoft.com/troubleshoot/windows-server/performance/ram-virtual-memory-pagefile-management) | Pages Output/sec, page-file usage | Supports the claims. This is a legacy 32-bit KB article, so it is used for counter meaning only. |
| [Cache and Memory Manager troubleshooting](https://learn.microsoft.com/windows-server/administration/performance-tuning/subsystem/cache-memory-management/troubleshoot) | Standby cache lifetime (1800 s), cache issues | Supports the claims. Scope noted: pre-Windows Server 2012. |
| [Profile the CPU and GPU with timing captures (PIX)](https://learn.microsoft.com/windows/pix/timing-captures/timing-captures-overview) | GPU engine utilization, memory, presentation counters, residency, demoted allocations, migrations | Supports the claims. Replaces the old `pix/articles/...` URL, which also resolves. |
| [GPU process memory counters report incorrect values (KB 4490156)](https://learn.microsoft.com/troubleshoot/windows-client/performance/gpu-process-memory-counters-report-wrong-value) | GPU process memory known issue | Corrected: the issue applies to Windows 10 version 1709 and later, not Windows 10 generally. |
| [Memory charts (WDDM)](https://learn.microsoft.com/windows-hardware/drivers/display/memory-charts) | Local versus system memory on adapter charts | Supports the concept. Counter-set names are not on this page. |

## Secondary and product-scoped sources (kept, labeled)

These are not Windows Server documentation. Each one is used only where no Windows primary source gives a value, and the metric file says what scope it applies to.

| Page | Where used | Scope |
|---|---|---|
| [System-Level Bottlenecks (BizTalk Server)](https://learn.microsoft.com/biztalk/technical-guides/system-level-bottlenecks) | Disk counter names, Avg. Disk Queue Length guidance, Pages/sec definition | BizTalk Server |
| [Storage and SQL Server capacity planning (SharePoint)](https://learn.microsoft.com/sharepoint/administration/storage-and-sql-server-capacity-planning-and-configuration) | Current Disk Queue Length (> 2 per disk), Disk Transfers/sec, spindle and LUN definitions | SharePoint Server |
| [Troubleshoot event ID 2115 (Operations Manager)](https://learn.microsoft.com/troubleshoot/system-center/scom/troubleshoot-event-2115-related-performance-problems) | Queue length limit (twice the spindles per LUN), disk latency tiers | Operations Manager. Its RAID formulas conflict with the gray-agent page below. |
| [Troubleshoot gray agent states (Operations Manager)](https://learn.microsoft.com/troubleshoot/system-center/scom/troubleshoot-gray-agent-states#troubleshooting-sql-server-performance) | Same queue guidance, different RAID 5 formula | Operations Manager. Recorded as a conflict. |
| [MSSQLSERVER_833](https://learn.microsoft.com/sql/relational-databases/errors-events/mssqlserver-833-database-engine-error?view=sql-server-ver17#more-information) | Warning that SQL Server queues can be long without a problem | SQL Server |
| [.NET performance counters: networking](https://learn.microsoft.com/dotnet/framework/debug-trace-profile/performance-counters#networking-performance-counters) | Pointer to .NET socket counters | .NET Framework. Reachable; content not read. |

## Removed

| Original reference | Claim it supported | Why removed | Replacement |
|---|---|---|---|
| Exchange 2013 performance counters | Processor Queue Length: less than 5 per processor | Not a Windows Server source. The Windows page gives no numeric threshold. | None. Entry says no numeric threshold. |
| Host Integration Server: useful performance counters | TCP connections as a user-load proxy | Not a Windows Server source. | None. |
| Azure Service Fabric monitor reference | Logical disk free space; network errors and discards | Not a Windows Server source. | Network-Related Performance Counters (Potential Network Problems section). |
| Azure Virtual Desktop insights glossary | Pages/sec, page faults, Avg. Disk sec/Transfer, disk and memory counter definitions | Not a Windows Server source. | Memory Performance Information; RAM, virtual memory article. Unsourced definitions are marked. |
| Azure Virtual Machines premium storage performance | Disk throughput, IOPS, % Idle Time | Not a Windows Server source. | None. Entries marked partly checked or unverified. |
| BizTalk: Checklist measuring performance on Hyper-V | Network utilization tiers (under 40%, 41–64%, 65–100%); Output Queue Length tiers (0, 1–2, above 2) | Tiers conflicted with the Windows page. | Scenario guide network thresholds (under 50%, 50–80%, over 80%); Output Queue Length from Network-Related Performance Counters. |
| BizTalk: Using the PAL tool | Paging file above 70%; network bandwidth | Thresholds come from the PAL report for BizTalk Server, not from Windows documentation. | Scenario guide and RAM article. |
| BizTalk: System-Level Bottlenecks (latency) | Disk latency: under 10 ms good, 15–25 ms fair, above 25 ms poor | Conflicts with the Windows scenario guide (under 15 ms healthy, over 25 ms warning, over 50 ms critical). | Scenario guide. The BizTalk values are kept only for the queue counter names. |
| SharePoint: % Disk Time above 90% busy | Disk busy threshold | Product-scoped; not a Windows value. | No threshold. Entry says so. |
| Windows Server: Memory consumption between identical environments | System\Processes as a load proxy | Scenario-specific, not a counter definition. | Memory Performance Information (ProcessCount and ThreadCount). |
| Azure VM memory issue troubleshooting | Working Set and Private Bytes pressure | Azure-specific. | Scenario guide (Working Set definition) and Memory Performance Information (Private Bytes). |
| Windows Client: Introduction to the page file | "System grows page file at 90% of commit limit" | The claim was not found in the source checked. | Removed. Claim not kept. |
| `pix/articles/timing-captures/pix-timing-captures` (older URL) | GPU counter list | Replaced by the current PIX timing-captures page. | PIX timing-captures overview. |

## Broken anchors fixed

| Original link | Problem | Fix |
|---|---|---|
| Network-Related Performance Counters `#resource-utilization` (5 uses) | Heading IDs on the page are `bkmk_ru`, so the anchor does not exist. | Link to the page; section named in text. |
| Network-Related Performance Counters `#potential-network-problems` | Same. | Same. |
| Network-Related Performance Counters `#receive-side-coalescing-rsc-performance` | Same. | Same. |

All other anchors used in this repo were found in the page HTML.
