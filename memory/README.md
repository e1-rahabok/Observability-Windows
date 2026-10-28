# Memory metrics

Available memory, commit charge, cache and standby lists, kernel pool, paging, and process memory for Windows.

Status: all entries unverified.

## Available memory

### Available MBytes (Memory)

- **Unit:** MB.
- **Formula:** Standby (cache) + free + zero page lists, in MB. Equivalent to the `MEMORYSTATUSEX.ullAvailPhys` basis.
- **Source:** `Memory\Available MBytes`; `Memory\Available Bytes` (bytes); WMI `Win32_PerfFormattedData_PerfOS_Memory` (`AvailableMBytes`).
- **Reference:** https://learn.microsoft.com/windows/win32/memory/memory-performance-information (associates counters with API data); https://learn.microsoft.com/troubleshoot/windows-server/performance/ram-virtual-memory-pagefile-management#monitoring-ram-and-virtual-memory-usage
- **Pitfall:** Microsoft's guidance: the counter stays above zero by design. The Virtual Memory Manager keeps a minimum free pool by trimming working sets. A low value alone does not prove a shortage. Pair with Pages/sec and % Committed Bytes In Use. Thresholds in the sources vary: 10% free is a warning (PAL); under 4 GB or 10% is healthy (scenario guide). Use the threshold set that matches the workload.

## Commit charge

### Committed Bytes, Commit Limit (Memory)

- **Unit:** bytes.
- **Formula:** Committed Bytes = virtual memory committed (RAM + page file reserved). Commit Limit = physical RAM + page file size.
- **Source:** `Memory\Committed Bytes`, `Memory\Commit Limit`.
- **Reference:** https://learn.microsoft.com/troubleshoot/windows-client/performance/introduction-to-the-page-file#system-committed-memory
- **Pitfall:** If Committed Bytes reaches Commit Limit, new allocations fail even with free RAM. The system grows system-managed page files at 90% of commit limit.

### % Committed Bytes In Use (Memory)

- **Unit:** percent.
- **Formula:** Committed Bytes / Commit Limit × 100.
- **Source:** `Memory\% Committed Bytes In Use`.
- **Reference:** https://learn.microsoft.com/troubleshoot/windows-server/performance/troubleshoot-performance-problems-in-windows#introduce-the-18-most-important-counters (warning 60–80%, critical 80–100%).
- **Pitfall:** Ratio is instantaneous, not averaged. Commit Limit changes when the page file grows.

## Cache and standby lists

### Cache Bytes, System Cache Resident Bytes (Memory)

- **Unit:** bytes.
- **Source:** `Memory\Cache Bytes`, `Memory\System Cache Resident Bytes`.
- **Reference:** https://learn.microsoft.com/windows-server/administration/performance-tuning/subsystem/cache-memory-management/troubleshoot
- **Pitfall:** Cache memory is reclaimable. High cache usage is not a fault unless Available MBytes is also low and Standby Cache Lifetime is short.

### Standby cache lifetime (Memory)

- **Unit:** seconds.
- **Formula:** Long-term average time a page stays in the standby list before reuse.
- **Source:** `Memory\Long-Term Average Standby Cache Lifetime (s)`.
- **Reference:** https://learn.microsoft.com/windows-server/administration/performance-tuning/subsystem/cache-memory-management/troubleshoot (threshold: below 1800 s suggests a cache problem on pre-2012 systems).
- **Pitfall:** Guidance applies to Windows Server 2008 R2 and earlier. Current Windows versions reduced this issue; use as a diagnostic signal only.

### Standby, Modified, Free, and Zero page lists (Memory)

- **Unit:** bytes.
- **Source:** `Memory\Standby Cache Normal Bytes`, `Standby Cache Reserve Bytes`, `Modified Page List Bytes`, `Free & Zero Page List Bytes`.
- **Reference:** https://learn.microsoft.com/windows/win32/memory/memory-performance-information
- **Pitfall:** These lists sum to Available MBytes. Use RAMMap to see per-process and per-file page-list usage.

### Windows Internals references

- **PFN database:** every physical page has a PFN entry tracking its state (active, standby, modified, free, zero). Reference: *Windows Internals* (memory management chapter).
- **Working set:** pages a process currently holds in physical RAM. Reference: *Windows Internals* and https://learn.microsoft.com/windows/win32/memory/address-windowing-extensions
- **Sysinternals RAMMap:** shows page-list usage. https://learn.microsoft.com/en-us/sysinternals/downloads/rammap

## Kernel pool

### Pool Paged Bytes, Pool Nonpaged Bytes (Memory)

- **Unit:** bytes.
- **Formula:** Kernel-mode allocations. Paged pool can be written to the page file; nonpaged pool cannot.
- **Source:** `Memory\Pool Paged Bytes`, `Memory\Pool Nonpaged Bytes`.
- **Reference:** https://learn.microsoft.com/troubleshoot/windows-server/performance/troubleshoot-performance-problems-in-windows#introduce-the-18-most-important-counters (nonpaged pool limit about 75% of RAM).
- **Pitfall:** Drivers allocate most pool memory. Process-level tools do not show it. Pool growth without workload growth suggests a driver leak. Use PoolMon (WDK) or Poolmon/Process Explorer.

## Paging

### Pages/sec, Pages Input/sec, Pages Output/sec (Memory)

- **Unit:** pages per second.
- **Formula:** Pages Input/sec = hard page faults read from disk. Pages Output/sec = pages written to page file. Pages/sec = Input + Output.
- **Source:** `Memory\Pages/sec`, `Memory\Pages Input/sec`, `Memory\Pages Output/sec`.
- **Reference:** https://learn.microsoft.com/troubleshoot/windows-server/performance/ram-virtual-memory-pagefile-management#monitoring-ram-and-virtual-memory-usage (Pages Output/sec is the best indicator of paging as a bottleneck); https://learn.microsoft.com/azure/virtual-desktop/insights-glossary#performance-counters
- **Pitfall:** Pages/sec includes cache reads (file I/O) and is not a pure fault count. Pages Output/sec is the more useful paging indicator.

### Page Faults/sec (Memory)

- **Unit:** faults per second.
- **Formula:** Δ(soft + hard faults) / Δt. Soft faults resolve from memory; hard faults read from disk.
- **Source:** `Memory\Page Faults/sec`.
- **Reference:** https://learn.microsoft.com/azure/virtual-desktop/insights-glossary#performance-counters
- **Pitfall:** High soft-fault rates are normal. Use Pages Input/sec to isolate disk-backed faults.

### % Usage, Paging File (PagingFile)

- **Unit:** percent.
- **Formula:** Page file in use / page file size × 100.
- **Source:** `Paging File(_Total)\% Usage`, `\% Usage Peak`.
- **Reference:** https://learn.microsoft.com/biztalk/technical-guides/using-the-performance-analysis-of-logs-pal-tool#understanding-the-pal-tool-report-for-microsoft-biztalk-server-2010 (PAL: check above 70%).
- **Pitfall:** High usage with low Pages Output/sec means the system used the page file but is not actively paging.

## Process memory

### Working Set, Private Bytes, Virtual Bytes (Process)

- **Unit:** bytes.
- **Formula:** Working Set = RAM in use by the process. Private Bytes = memory committed to the process and not shared. Virtual Bytes = virtual address space reserved.
- **Source:** `Process(<name>)\Working Set`, `\Private Bytes`, `\Virtual Bytes`; `Process(<name>)\Working Set - Private`.
- **Reference:** https://learn.microsoft.com/troubleshoot/windows-server/performance/troubleshoot-performance-problems-in-windows#introduce-the-18-most-important-counters; https://learn.microsoft.com/troubleshoot/azure/virtual-machines/windows/azure-windows-vm-memory-issue#troubleshoot-memory-pressure-issues
- **Pitfall:** Working Set includes shared pages (for example DLLs), so the sum across processes exceeds physical RAM. Private Bytes is the better leak indicator. A process that grows Private Bytes without releasing them over time is a leak candidate; a new process naturally grows at startup.

### Process memory via Win32 API

- **Unit:** bytes.
- **Source:** `GetProcessMemoryInfo` / `PROCESS_MEMORY_COUNTERS_EX` (`PrivateUsage`, `WorkingSetSize`).
- **Reference:** https://learn.microsoft.com/windows/win32/memory/memory-performance-information
- **Pitfall:** Same caveats as the counters above.

## Sysinternals references

- **RAMMap:** page-list breakdown, file cache, and metafile usage. https://learn.microsoft.com/en-us/sysinternals/downloads/rammap
- **VMMap:** per-process virtual address space, private bytes, and heap usage. https://learn.microsoft.com/en-us/sysinternals/downloads/vmmap
- **Process Explorer:** live memory per process. https://learn.microsoft.com/en-us/sysinternals/downloads/process-explorer
