# Memory metrics

Available memory, commit charge, cache and standby lists, kernel pool, paging, and process memory for Windows.

Status: source-checked against Microsoft Learn on 2026-10-29. Not yet verified on a live machine. Status values are defined in the [root README](../README.md#verification-status).

## Available memory

### Available MBytes (Memory)

- **Unit:** MB.
- **Formula:** Memory available to all programs, including the kernel. Includes standby (cache) and free pages.
- **Source:** `Memory\Available MBytes`; `Memory\Available Bytes` (bytes).
- **Reference:** [Scenario guide: Troubleshoot performance problems in Windows](https://learn.microsoft.com/troubleshoot/windows-server/performance/troubleshoot-performance-problems-in-windows#introduce-the-18-most-important-counters) (healthy: more than 10% or at least 4 GB free; warning: under 10%; critical: under 1% or under 500 MB). [Memory Performance Information](https://learn.microsoft.com/windows/win32/memory/memory-performance-information) (maps `Available KB` to `PERFORMANCE_INFORMATION.PhysicalAvailable`).
- **Status:** source-checked.
- **Pitfall:** The counter does not reach zero by design, because the Memory Manager keeps a minimum pool of pages free by trimming pages to the page file. A low value is a signal, not proof of a shortage. Pair with Pages Output/sec and % Committed Bytes In Use.

## Commit charge

### Committed Bytes, Commit Limit (Memory)

- **Unit:** bytes.
- **Formula:** Committed Bytes = virtual memory committed to processes (RAM pages or page-file slots). Commit Limit = RAM plus page-file capacity.
- **Source:** `Memory\Committed Bytes`, `Memory\Commit Limit`.
- **Reference:** [Memory Performance Information](https://learn.microsoft.com/windows/win32/memory/memory-performance-information) (`CommitTotal` and `CommitLimit` in `PERFORMANCE_INFORMATION`). [RAM, virtual memory, pagefile, and memory management](https://learn.microsoft.com/troubleshoot/windows-server/performance/ram-virtual-memory-pagefile-management#monitoring-ram-and-virtual-memory-usage) (Committed Bytes grows past available RAM and paging increases).
- **Status:** source-checked.
- **Pitfall:** If Committed Bytes reaches Commit Limit, new allocations fail even when RAM is free. Commit Limit changes when the page file grows.

### % Committed Bytes In Use (Memory)

- **Unit:** percent.
- **Formula:** Committed Bytes / Commit Limit × 100.
- **Source:** `Memory\% Committed Bytes In Use`.
- **Reference:** [Scenario guide: Troubleshoot performance problems in Windows](https://learn.microsoft.com/troubleshoot/windows-server/performance/troubleshoot-performance-problems-in-windows#introduce-the-18-most-important-counters) (healthy: 0–50%; warning: 60–80%; critical: 80–100%).
- **Status:** source-checked.
- **Pitfall:** The ratio is instantaneous, not averaged. Commit Limit changes when the page file grows.

## Cache and standby lists

### Cache Bytes, System Cache Resident Bytes (Memory)

- **Unit:** bytes.
- **Source:** `Memory\Cache Bytes`, `Memory\System Cache Resident Bytes`.
- **Reference:** [Troubleshoot Cache and Memory Manager Performance Issues](https://learn.microsoft.com/windows-server/administration/performance-tuning/subsystem/cache-memory-management/troubleshoot) (use RAMMap when Available MBytes is low and the system cache uses a large share of memory). [Memory Performance Information](https://learn.microsoft.com/windows/win32/memory/memory-performance-information) (`SystemCache` maps to the Cache counter).
- **Status:** source-checked.
- **Pitfall:** Cache memory is reclaimable. High cache usage is not a fault unless Available MBytes is also low.

### Standby cache lifetime (Memory)

- **Unit:** seconds.
- **Formula:** Long-term average time a page stays in the standby list before reuse.
- **Source:** `Memory\Long-Term Average Standby Cache Lifetime (s)`.
- **Reference:** [Troubleshoot Cache and Memory Manager Performance Issues](https://learn.microsoft.com/windows-server/administration/performance-tuning/subsystem/cache-memory-management/troubleshoot) (the page lists a value under 1800 seconds as a sign of a cache problem, and describes the issues as applying before Windows Server 2012).
- **Status:** source-checked.
- **Pitfall:** The guidance targets pre-2012 systems. Current Windows versions reduced this issue; use the value as a diagnostic signal only.

### Standby, Modified, Free, and Zero page lists (Memory)

- **Unit:** bytes.
- **Source:** `Memory\Standby Cache Normal Bytes`, `Memory\Standby Cache Reserve Bytes`, `Memory\Modified Page List Bytes`, `Memory\Free & Zero Page List Bytes`.
- **Reference:** [Memory Performance Information](https://learn.microsoft.com/windows/win32/memory/memory-performance-information) (maps these page-list counters to the Task Manager Physical Memory fields). Counter names were not checked against this page on this pass.
- **Status:** partly checked.
- **Pitfall:** Together these lists make up most of Available MBytes. Use RAMMap to see per-process and per-file page-list usage.

## Kernel pool

### Pool Paged Bytes, Pool Nonpaged Bytes (Memory)

- **Unit:** bytes.
- **Formula:** Kernel-mode allocations. Paged pool can be written to the page file; nonpaged pool cannot.
- **Source:** `Memory\Pool Paged Bytes`, `Memory\Pool Nonpaged Bytes`.
- **Reference:** [Scenario guide: Troubleshoot performance problems in Windows](https://learn.microsoft.com/troubleshoot/windows-server/performance/troubleshoot-performance-problems-in-windows#introduce-the-18-most-important-counters) (nonpaged pool limit is about 75% of RAM; pool counts directly affect Available MBytes). [Memory Performance Information](https://learn.microsoft.com/windows/win32/memory/memory-performance-information) (`KernelPaged`, `KernelNonpaged`).
- **Status:** source-checked.
- **Pitfall:** Drivers allocate most pool memory, and process-level tools do not show who owns it. Pool growth without workload growth suggests a driver leak. Use PoolMon from the Windows Driver Kit to see pool tags.

## Paging

### Pages/sec, Pages Input/sec, Pages Output/sec (Memory)

- **Unit:** pages per second.
- **Formula:** Pages/sec = pages read from or written to disk to resolve hard page faults. Pages Output/sec = pages written to the page file.
- **Source:** `Memory\Pages/sec`, `Memory\Pages Input/sec`, `Memory\Pages Output/sec`.
- **Reference:** [RAM, virtual memory, pagefile, and memory management](https://learn.microsoft.com/troubleshoot/windows-server/performance/ram-virtual-memory-pagefile-management#monitoring-ram-and-virtual-memory-usage) (Pages Output/sec is the best counter to monitor when paging is suspected as the bottleneck; a high Pages/sec does not by itself mean a RAM shortage). The article is a legacy 32-bit KB article, so use its guidance for counter meaning, not for current thresholds.
- **Status:** source-checked.
- **Pitfall:** Pages/sec includes cache reads (file I/O), so it is not a pure fault count. Pages Output/sec is the more useful paging indicator.

### Paging File % Usage (PagingFile)

- **Unit:** percent.
- **Formula:** Page file in use / page file size × 100.
- **Source:** `Paging File(_Total)\% Usage`, `\% Usage Peak`.
- **Reference:** [RAM, virtual memory, pagefile, and memory management](https://learn.microsoft.com/troubleshoot/windows-server/performance/ram-virtual-memory-pagefile-management#monitoring-ram-and-virtual-memory-usage) (keep the page file no more than 50–75% used for volatile workloads; at 100% the page file is full and the system stops working).
- **Status:** source-checked.
- **Pitfall:** High usage with low Pages Output/sec means the page file holds data but the system is not actively paging.

## Process memory

### Working Set, Private Bytes, Virtual Bytes (Process)

- **Unit:** bytes.
- **Formula:** Working Set = RAM currently used by the process. Private Bytes = memory committed to the process and not shared. Virtual Bytes = virtual address space in use.
- **Source:** `Process(<name>)\Working Set`, `\Private Bytes`, `\Virtual Bytes`; `Process(<name>)\Working Set - Private` (not supported on Windows Server 2003 and XP).
- **Reference:** [Scenario guide: Troubleshoot performance problems in Windows](https://learn.microsoft.com/troubleshoot/windows-server/performance/troubleshoot-performance-problems-in-windows) (Working Set is the RAM a process uses at a given time). [Memory Performance Information](https://learn.microsoft.com/windows/win32/memory/memory-performance-information) (Process object counters and their `PROCESS_MEMORY_COUNTERS_EX` fields, including `Private Bytes` as `PrivateUsage`).
- **Status:** source-checked.
- **Pitfall:** Working Set includes shared pages (for example DLLs), so the sum across processes can exceed physical RAM. Private Bytes is the better leak indicator. A process that keeps growing Private Bytes without releasing them is a leak candidate. A new process grows at startup.

### Process memory via Win32 API

- **Unit:** bytes.
- **Source:** `GetProcessMemoryInfo` / `PROCESS_MEMORY_COUNTERS_EX` (`PrivateUsage`, `WorkingSetSize`).
- **Reference:** [Memory Performance Information](https://learn.microsoft.com/windows/win32/memory/memory-performance-information) (structure fields).
- **Status:** source-checked.
- **Pitfall:** Same caveats as the counters above.

## Sysinternals and Windows Internals references

- **RAMMap:** page-list breakdown, file cache, and metafile usage. [Sysinternals RAMMap](https://learn.microsoft.com/en-us/sysinternals/downloads/rammap).
- **VMMap:** per-process virtual address space, private bytes, and heap usage. [Sysinternals VMMap](https://learn.microsoft.com/en-us/sysinternals/downloads/vmmap).
- **Process Explorer:** live memory per process. [Sysinternals Process Explorer](https://learn.microsoft.com/en-us/sysinternals/downloads/process-explorer).
- **PFN database:** every physical page has a PFN entry tracking its state (active, standby, modified, free, zero). Reference: *Windows Internals* (memory management chapter). Not a public API.
- **Working set:** Address Windowing Extensions is the API for direct physical-memory control; see [Address Windowing Extensions](https://learn.microsoft.com/windows/win32/memory/address-windowing-extensions).
