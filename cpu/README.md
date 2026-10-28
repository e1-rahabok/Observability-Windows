# CPU metrics

Processor time, interrupt and DPC time, queueing, scheduler activity, process CPU, and throttling for Windows.

Status: all entries unverified.

## Processor utilization

### % Processor Time (Processor Information)

- **Unit:** percent, 0 to 100 per logical processor. `_Total` is the average across all logical processors.
- **Formula:** 100 × (1 − Δ idle time / Δ elapsed time). PDH derives this from raw timer counters.
- **Source:** Performance counter `Processor Information(_Total)\% Processor Time` (Windows Server 2008 R2 and later; the older `Processor` object is still present).
- **Reference:** https://learn.microsoft.com/windows/win32/perfctrs/collecting-performance-data (multiprocessor counter behavior); WMI `Win32_PerfFormattedData_Counters_ProcessorInformation`.
- **Pitfall:** `_Total` averages all logical processors. One saturated core on an 8-core host shows about 12.5%. Check per-processor instances before concluding the CPU is idle.

### % Privileged Time, % User Time (Processor Information)

- **Unit:** percent of total processor time.
- **Formula:** Privileged = kernel-mode time / elapsed time. User = user-mode time / elapsed time. % Privileged + % User ≈ % Processor Time.
- **Source:** `Processor Information(_Total)\% Privileged Time`, `\% User Time`.
- **Reference:** https://learn.microsoft.com/troubleshoot/windows-server/support-tools/troubleshoot-issues-performance-monitor#performance-counters-for-cpu-usage
- **Pitfall:** High % Privileged Time often points to drivers, interrupts, or syscalls, not the application. Check % DPC Time and % Interrupt Time first.

### % DPC Time, % Interrupt Time (Processor Information)

- **Unit:** percent of total processor time.
- **Formula:** Time spent in DPC (deferred procedure call) or ISR (interrupt service routine) / elapsed time. DPC and ISR time are subsets of % Privileged Time.
- **Source:** `Processor Information(_Total)\% DPC Time`, `\% Interrupt Time`.
- **Reference:** https://learn.microsoft.com/troubleshoot/windows-server/support-tools/troubleshoot-issues-performance-monitor#performance-counters-for-cpu-usage
- **Pitfall:** Microsoft's guidance flags % Interrupt Time above 30% as a possible hardware or driver issue. A short spike is normal. Use Process Monitor or ETW (`xperf`/WPR) to identify the driver.

### Interrupts/sec, DPCs Queued/sec (Processor Information)

- **Unit:** events per second.
- **Formula:** Rate of device interrupts received, or DPCs added to the logical processor's DPC queue: Δcount / Δt.
- **Source:** `Processor Information(_Total)\Interrupts/sec`, `\DPCs Queued/sec`.
- **Reference:** https://learn.microsoft.com/windows-server/networking/technologies/network-subsystem/net-sub-performance-counters (DPCs Queued/sec definition).
- **Pitfall:** DPCs Queued/sec counts additions, not the current queue length. Interrupt rate alone does not indicate a fault.

## Queueing and scheduling

### Processor Queue Length (System)

- **Unit:** threads (count, instantaneous).
- **Formula:** Number of threads in the ready queue waiting for a processor. Divide by logical processor count for a per-processor value.
- **Source:** `System\Processor Queue Length`.
- **Reference:** https://learn.microsoft.com/exchange/exchange-2013-performance-counters-exchange-2013-help#processor-and-process-counters (threshold guidance: less than 5 per processor); https://learn.microsoft.com/troubleshoot/windows-server/support-tools/troubleshoot-issues-performance-monitor#performance-counters-for-cpu-usage
- **Pitfall:** A sustained value above 5 per processor indicates CPU contention. The counter is a last-observed value, not an average.

### Context Switches/sec (System)

- **Unit:** switches per second.
- **Formula:** Δ context switches / Δt.
- **Source:** `System\Context Switches/sec`.
- **Reference:** https://learn.microsoft.com/troubleshoot/windows-server/support-tools/troubleshoot-issues-performance-monitor#performance-counters-for-cpu-usage
- **Pitfall:** No universal threshold. Compare against a baseline from the same workload.

### System Calls/sec (System)

- **Unit:** calls per second.
- **Formula:** Δ system calls / Δt.
- **Source:** `System\System Calls/sec`.
- **Reference:** https://learn.microsoft.com/troubleshoot/windows-server/support-tools/troubleshoot-issues-performance-monitor#performance-counters-for-cpu-usage
- **Pitfall:** High rates reflect application behavior, not a fault by themselves.

## Process CPU

### % Processor Time (Process)

- **Unit:** percent. Range is 0 to 100 × logical processor count.
- **Formula:** Σ over threads of CPU time / elapsed time × 100.
- **Source:** `Process(<name>)\% Processor Time`.
- **Reference:** https://learn.microsoft.com/windows/win32/perfctrs/collecting-performance-data (multiprocessor counter behavior, range 0 to 100 × ProcessorCount).
- **Pitfall:** Values above 100 are normal on multi-threaded processes. Instance names repeat across processes (`svchost#1`), and a restarted process can shift samples between instances. Use PID-based queries when accuracy matters.

### Thread Count, Handle Count (Process)

- **Unit:** count (gauge).
- **Source:** `Process(_Total)\Thread Count`, `Process(_Total)\Handle Count`; per-process via `Process(<name>)\Thread Count`.
- **Reference:** https://learn.microsoft.com/troubleshoot/windows-server/support-tools/troubleshoot-issues-performance-monitor (CPU counters list); WMI `Win32_Process` (`ThreadCount`, `HandleCount`).
- **Pitfall:** Handle and thread counts grow with workload. A steady climb without workload change indicates a leak.

### Process CPU time (100-ns ticks)

- **Unit:** 100-ns ticks (TimeSpan ticks). 10,000,000 ticks = 1 s.
- **Formula:** `UserProcessorTime`, `PrivilegedProcessorTime`, `TotalProcessorTime` are cumulative. Rate = Δticks / Δt / 10,000,000.
- **Source:** .NET `System.Diagnostics.Process` properties; WMI `Win32_Process` (`KernelModeTime`, `UserModeTime` in 100-ns units).
- **Reference:** https://learn.microsoft.com/windows/win32/cimwin32prov/win32-process (KernelModeTime, UserModeTime).
- **Pitfall:** Values are cumulative since process start. Divide by the process lifetime or Δt before comparing.

## Throttling and pressure

### Processor Frequency, % of Maximum Frequency (Processor Information)

- **Unit:** MHz (frequency), percent (of maximum).
- **Source:** `Processor Information(_Total)\Processor Frequency`, `\% of Maximum Frequency`.
- **Reference:** WMI `Win32_Processor` (`CurrentClockSpeed`, `MaxClockSpeed`).
- **Pitfall:** Frequency drops under power management. A low frequency with high utilization can mean power plan throttling, not a workload change.

## Physical CPU identity

### Logical processors and cores (Win32_Processor)

- **Unit:** count.
- **Source:** WMI `Win32_Processor` (`NumberOfCores`, `NumberOfLogicalProcessors`).
- **Reference:** https://learn.microsoft.com/windows/win32/cimwin32prov/win32-processor
- **Pitfall:** `NumberOfLogicalProcessors` counts hyperthreads. Core count and logical processor count differ on SMT systems. Processor groups (more than 64 logical processors) may need `Get-CimInstance` per group.

## Windows Internals references

- **KPCR (Kernel Processor Control Region):** per-processor kernel state, including the current thread and DPC queue. Reference: *Windows Internals* (kernel chapter).
- **KDPC / DPC queue:** each processor has its own DPC queue (see DPCs Queued/sec). Reference: *Windows Internals*.
- **Sysinternals:** Process Explorer (per-process CPU, threads); Process Monitor (driver-level activity). https://learn.microsoft.com/en-us/sysinternals/downloads/process-explorer
