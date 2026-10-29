# Glossary

Terms used in the metric folders. Each definition gives its source. Where no Microsoft Learn page was checked, the source column says so.

Source values:

- **Learn:** a Microsoft Learn page, linked. The page was read on 2026-10-29.
- **Book:** *Windows Internals* (Russinovich, Solomon, Ionescu). Not checked against an online page.
- **Not checked:** a definition from general knowledge. Verify before relying on it.

## Counter model

| Term | Meaning | Source |
|---|---|---|
| Counter object | A group of related counters, for example `Processor`, `Memory`, `PhysicalDisk`, or `Process`. | [Scenario guide](https://learn.microsoft.com/troubleshoot/windows-server/performance/troubleshoot-performance-problems-in-windows) |
| Instance | A specific item inside a counter object, for example one processor, disk, or process. Written as `Object(instance)\Counter`. | [Scenario guide](https://learn.microsoft.com/troubleshoot/windows-server/performance/troubleshoot-performance-problems-in-windows) |
| `_Total` | The aggregate instance. For `Processor(_Total)\% Processor Time` it is the average over all processors. | [Collecting Performance Data](https://learn.microsoft.com/windows/win32/perfctrs/collecting-performance-data#understanding-multiple-processor-counters) |
| PDH | Performance Data Helper, the API that collects counters and computes formatted values. | [Collecting Performance Data](https://learn.microsoft.com/windows/win32/perfctrs/collecting-performance-data) |
| Raw and formatted value | PDH stores the raw value and computes a displayable (formatted) value from it. Rate and percent counters need two samples. | [Collecting Performance Data](https://learn.microsoft.com/windows/win32/perfctrs/collecting-performance-data) |
| Queue-length counter type | A counter type that reports the average length of a queue over time, computed from samples. | [Queue-length Algorithm Counter Types](https://learn.microsoft.com/windows/win32/wmisdk/queue-length-algorithm-counter-types) |
| `Process V2` | A Windows 11 counter set whose instance names include the process ID, avoiding mismatches when a process ends. | [Collecting Performance Data](https://learn.microsoft.com/windows/win32/perfctrs/collecting-performance-data#understanding-multiple-processor-counters) |
| Gauge | A counter that reports the current value, not a cumulative total. | Not checked (general counter terminology) |

## CPU and scheduling

| Term | Meaning | Source |
|---|---|---|
| Kernel mode | Operating system and driver code, which runs with full hardware access and a protected process (process ID 4, System). | [Scenario guide](https://learn.microsoft.com/troubleshoot/windows-server/performance/troubleshoot-performance-problems-in-windows#cpu) |
| User mode | Applications and services. Hardware access goes through the kernel. | [Scenario guide](https://learn.microsoft.com/troubleshoot/windows-server/performance/troubleshoot-performance-problems-in-windows#cpu) |
| Privileged time | Time a processor spends in kernel mode. | [Scenario guide](https://learn.microsoft.com/troubleshoot/windows-server/performance/troubleshoot-performance-problems-in-windows#cpu) |
| DPC | Deferred procedure call. Work that a driver queues to run after a hardware interrupt, to keep interrupt handling short. | Expansion from general knowledge. The Learn page that uses the term ([Troubleshoot issues using Performance Monitor](https://learn.microsoft.com/troubleshoot/windows-server/support-tools/troubleshoot-issues-performance-monitor#performance-counters-for-cpu-usage)) gives the counter name only. |
| ISR | Interrupt service routine. The driver code that runs first when a device raises an interrupt. | Expansion from general knowledge. The Learn page that uses the term ([Troubleshoot issues using Performance Monitor](https://learn.microsoft.com/troubleshoot/windows-server/support-tools/troubleshoot-issues-performance-monitor#performance-counters-for-cpu-usage)) gives the counter name only. |
| DPC queue | A per-logical-processor queue of DPCs waiting to run. DPCs Queued/sec counts additions to it. | [Network-Related Performance Counters](https://learn.microsoft.com/windows-server/networking/technologies/network-subsystem/net-sub-performance-counters) |
| Processor Queue Length | The number of threads that are ready to run and waiting for a processor. | [Troubleshoot issues using Performance Monitor](https://learn.microsoft.com/troubleshoot/windows-server/support-tools/troubleshoot-issues-performance-monitor#performance-counters-for-cpu-usage) |
| Context switch | The scheduler switching the processor from one thread to another. | [Troubleshoot issues using Performance Monitor](https://learn.microsoft.com/troubleshoot/windows-server/support-tools/troubleshoot-issues-performance-monitor#performance-counters-for-cpu-usage) (listed as a CPU counter) |
| KPCR | Kernel processor control region. Per-processor kernel state, including the current thread and DPC queue. | Book |
| EPROCESS, ETHREAD | Kernel structures for a process and a thread. Counters such as Thread Count are derived from them. | Book |

## Memory

| Term | Meaning | Source |
|---|---|---|
| Available MBytes | Memory available to all programs, including the kernel. | [Scenario guide](https://learn.microsoft.com/troubleshoot/windows-server/performance/troubleshoot-performance-problems-in-windows#introduce-the-18-most-important-counters) |
| Commit charge (Committed Bytes) | Virtual memory committed to processes. Backed by RAM or page file slots. | [Memory Performance Information](https://learn.microsoft.com/windows/win32/memory/memory-performance-information) (`CommitTotal`) |
| Commit limit | The most memory that can be committed. Allocations fail when the charge reaches this limit. | [Memory Performance Information](https://learn.microsoft.com/windows/win32/memory/memory-performance-information) (`CommitLimit`) |
| Working set | The RAM a process is using at a given time. Includes shared pages. | [Scenario guide](https://learn.microsoft.com/troubleshoot/windows-server/performance/troubleshoot-performance-problems-in-windows#introduce-the-18-most-important-counters) |
| Private bytes | Memory committed to a process and not shared with other processes. | [Memory Performance Information](https://learn.microsoft.com/windows/win32/memory/memory-performance-information) (`PrivateUsage`, shown as Commit Size) |
| Paged pool | Kernel memory that can be written to the page file. | [Scenario guide](https://learn.microsoft.com/troubleshoot/windows-server/performance/troubleshoot-performance-problems-in-windows#introduce-the-18-most-important-counters) |
| Nonpaged pool | Kernel memory that stays in RAM. Its limit is about 75% of RAM. | [Scenario guide](https://learn.microsoft.com/troubleshoot/windows-server/performance/troubleshoot-performance-problems-in-windows#introduce-the-18-most-important-counters) |
| Page file (pagefile.sys) | The disk file that holds pages moved out of RAM. | [RAM, virtual memory, pagefile, and memory management](https://learn.microsoft.com/troubleshoot/windows-server/performance/ram-virtual-memory-pagefile-management) |
| Paging | Moving pages between RAM and the page file (or other file-backed storage). | [RAM, virtual memory, pagefile, and memory management](https://learn.microsoft.com/troubleshoot/windows-server/performance/ram-virtual-memory-pagefile-management#monitoring-ram-and-virtual-memory-usage) |
| Hard page fault | A page fault that needs a read from disk. Pages/sec counts pages read or written to disk for these faults. | [Pages/sec definition in System-Level Bottlenecks (BizTalk Server)](https://learn.microsoft.com/biztalk/technical-guides/system-level-bottlenecks#high-level-system-bottlenecks) (product page, definition only) |
| Soft page fault | A page fault that resolves from memory without a disk read. | Not checked |
| Standby list | Pages kept as cache after leaving working sets. They can be reused without a disk read. | Book |
| Modified list | Pages changed in memory that must be written to disk before reuse. | Book |
| Free and zero lists | Pages ready for immediate use. Zero-list pages are cleared. | Book |
| PFN database | The kernel table with one entry per physical page, tracking its state. | Book |
| Cache Bytes | Memory used by the system file cache. It is reclaimable. | [Memory Performance Information](https://learn.microsoft.com/windows/win32/memory/memory-performance-information) (`SystemCache`) |
| Standby cache lifetime | The average time a page stays in the standby list before reuse. | [Cache and Memory Manager troubleshooting](https://learn.microsoft.com/windows-server/administration/performance-tuning/subsystem/cache-memory-management/troubleshoot) |
| Address Windowing Extensions (AWE) | The API that lets an application map physical memory into its address space. | [Address Windowing Extensions](https://learn.microsoft.com/windows/win32/memory/address-windowing-extensions) |

## Disk

| Term | Meaning | Source |
|---|---|---|
| PhysicalDisk | Counters for the physical disk, tracked by the partition manager. | [System-Level Bottlenecks (BizTalk Server)](https://learn.microsoft.com/biztalk/technical-guides/system-level-bottlenecks#performance-counters-for-measuring-disk-io-bottlenecks) |
| LogicalDisk | Counters for a volume, tracked by the volume manager. | [System-Level Bottlenecks (BizTalk Server)](https://learn.microsoft.com/biztalk/technical-guides/system-level-bottlenecks#performance-counters-for-measuring-disk-io-bottlenecks) |
| Avg. Disk sec/Transfer | The average seconds needed to complete one disk I/O. | [Troubleshoot event ID 2115 (Operations Manager)](https://learn.microsoft.com/troubleshoot/system-center/scom/troubleshoot-event-2115-related-performance-problems#resolution) |
| Avg. Disk Queue Length | The average number of requests in flight below the driver during the sample interval. Includes requests in service. | [System-Level Bottlenecks (BizTalk Server)](https://learn.microsoft.com/biztalk/technical-guides/system-level-bottlenecks#high-level-system-bottlenecks) (definition) |
| Current Disk Queue Length | The number of requests outstanding when the sample is taken. | [Storage and SQL Server capacity planning (SharePoint)](https://learn.microsoft.com/sharepoint/administration/storage-and-sql-server-capacity-planning-and-configuration#validate-and-monitor-storage-and-sql-server-performance) (definition) |
| Spindle | A rotating disk in a RAID set. Product guidance uses spindle count to set queue limits. | [Storage and SQL Server capacity planning (SharePoint)](https://learn.microsoft.com/sharepoint/administration/storage-and-sql-server-capacity-planning-and-configuration) |
| LUN | Logical unit number. A volume presented by a storage array to a host. | [Storage and SQL Server capacity planning (SharePoint)](https://learn.microsoft.com/sharepoint/administration/storage-and-sql-server-capacity-planning-and-configuration) |
| Storport | The Windows port driver that carries storage requests between the disk stack and the hardware adapter driver. | Book |

## Network

| Term | Meaning | Source |
|---|---|---|
| NDIS | Network Driver Interface Specification. The Windows model for network drivers. The output queue counter reports its queued requests. | [Network-Related Performance Counters](https://learn.microsoft.com/windows-server/networking/technologies/network-subsystem/net-sub-performance-counters) (names NDIS; definition not checked) |
| Output Queue Length | Packets waiting in the output queue. Above 2 causes delays, per Microsoft. | [Network-Related Performance Counters](https://learn.microsoft.com/windows-server/networking/technologies/network-subsystem/net-sub-performance-counters) |
| RSC | Receive Side Coalescing. Combines received TCP segments into larger packets to reduce per-packet CPU work. | [Network-Related Performance Counters](https://learn.microsoft.com/windows-server/networking/technologies/network-subsystem/net-sub-performance-counters) (section name; definition not checked) |
| Winsock | The Windows socket API layer. | Book |

## GPU

| Term | Meaning | Source |
|---|---|---|
| WDDM | Windows Display Driver Model. The display driver architecture used from Windows Vista on. | [WDDM architecture](https://learn.microsoft.com/windows-hardware/drivers/display/windows-vista-and-later-display-driver-model-architecture) (title only; content not checked) |
| DXGK | The graphics kernel that schedules GPU work and manages VRAM. | [PIX timing captures](https://learn.microsoft.com/windows/pix/timing-captures/timing-captures-overview#gpu-profiling) |
| VRAM / local video memory | Memory on the GPU board. Shown in red on adapter memory charts. | [Memory charts](https://learn.microsoft.com/windows-hardware/drivers/display/memory-charts) |
| Shared (system) memory for GPU | System memory the GPU can use. Shown in blue on adapter memory charts. | [Memory charts](https://learn.microsoft.com/windows-hardware/drivers/display/memory-charts) |
| Residency | Whether a GPU resource is in video memory. Operations include MakeResident and Evict. | [PIX timing captures](https://learn.microsoft.com/windows/pix/timing-captures/timing-captures-overview#gpu-profiling) |
| Demoted allocation | A resource the graphics kernel could not place in VRAM, so it sits in system memory. | [PIX timing captures](https://learn.microsoft.com/windows/pix/timing-captures/timing-captures-overview#gpu-profiling) |
| Allocation migration | Moving a demoted allocation back to VRAM. It suspends the GPU, so it is expensive. | [PIX timing captures](https://learn.microsoft.com/windows/pix/timing-captures/timing-captures-overview#gpu-profiling) |
| PIX | Microsoft's graphics and CPU profiler. Its timing captures record GPU and CPU activity. | [PIX timing captures](https://learn.microsoft.com/windows/pix/timing-captures/timing-captures-overview) |
| WPR / WPA | Windows Performance Recorder and Windows Performance Analyzer, part of the Windows Performance Toolkit. | [Windows Performance Toolkit](https://learn.microsoft.com/windows-hardware/test/wpt/) |
| GPUView | Tool that shows GPU queues and memory charts per adapter and process. | [Memory charts](https://learn.microsoft.com/windows-hardware/drivers/display/memory-charts) |

## System and reliability

| Term | Meaning | Source |
|---|---|---|
| WMI / CIM | Windows Management Instrumentation and its CIM classes, such as `Win32_OperatingSystem`, `Win32_Process`, and `Win32_Processor`. | [Win32 CIM provider classes](https://learn.microsoft.com/windows/win32/cimwin32prov/win32-operatingsystem) |
| WHEA | Windows Hardware Error Architecture. The framework that reports hardware errors to Windows. | [WHEA introduction](https://learn.microsoft.com/windows-hardware/drivers/whea/introduction-to-the-windows-hardware-error-architecture) (title only; content not checked) |
| Bugcheck (stop error) | The kernel's fatal-error report. Each bugcheck has a code that identifies the cause. | [Bug Check Code Reference](https://learn.microsoft.com/windows-hardware/drivers/debugger/bug-check-code-reference2) (title only; content not checked) |
| Fast Startup | A hybrid shutdown that saves kernel state, so a machine can appear to restart without a full boot. | Not checked |
| Handle | A reference to a kernel object, such as a file, thread, or registry key. | Book |
| Sysinternals | A set of diagnostic tools (Process Explorer, Handle, RAMMap, VMMap, Process Monitor, TCPView). | [Sysinternals](https://learn.microsoft.com/en-us/sysinternals/) |
