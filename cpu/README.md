# CPU metrics

Processor time, interrupt and DPC time, queueing, scheduler activity, process CPU, and throttling for Windows.

Status: source-checked against Microsoft Learn on 2026-10-29. Not yet verified on a live machine. Status values are defined in the [root README](../README.md#verification-status).

## Processor utilization

### % Processor Time (Processor Information)

- **Unit:** percent, 0 to 100 per logical processor. `_Total` is the average across all logical processors.
- **Formula:** 100 × (1 − Δ idle time / Δ elapsed time). PDH derives this from raw timer counters.
- **Source:** Performance counter `Processor Information(_Total)\% Processor Time`.
- **Reference:** [Troubleshoot issues using Performance Monitor](https://learn.microsoft.com/troubleshoot/windows-server/support-tools/troubleshoot-issues-performance-monitor#performance-counters-for-cpu-usage) (threshold table: 0–50% healthy, 50–80% warning, 80–100% critical; sustained use above 85% indicates a CPU bottleneck). [Collecting Performance Data](https://learn.microsoft.com/windows/win32/perfctrs/collecting-performance-data#understanding-multiple-processor-counters) (`_Total` is the average of all processors).
- **Status:** source-checked.
- **Pitfall:** `_Total` averages all logical processors. One saturated core on an 8-core host shows about 12.5%. Check per-processor instances before concluding the CPU is idle.

### % User Time, % Privileged Time (Processor Information)

- **Unit:** percent of total processor time.
- **Formula:** User = user-mode time / elapsed time. Privileged = kernel-mode time / elapsed time. Together they approximate % Processor Time (DPC and interrupt time are counted within Privileged time).
- **Source:** `Processor Information(_Total)\% User Time`, `\% Privileged Time`.
- **Reference:** [Scenario guide: Troubleshoot performance problems in Windows](https://learn.microsoft.com/troubleshoot/windows-server/performance/troubleshoot-performance-problems-in-windows#cpu) (User Time: under 50% healthy, 50–80% warning, over 80% critical. Privileged Time: under 30% healthy, 30–50% warning, over 50% critical).
- **Status:** source-checked.
- **Pitfall:** High Privileged Time usually reflects drivers, interrupts, or hardware I/O, not the application. Check % DPC Time and % Interrupt Time first.

### % DPC Time, % Interrupt Time (Processor Information)

- **Unit:** percent of total processor time.
- **Formula:** Time spent in DPC (deferred procedure call) or ISR (interrupt service routine) / elapsed time. DPC and interrupt time are part of Privileged Time.
- **Source:** `Processor Information(_Total)\% DPC Time`, `\% Interrupt Time`.
- **Reference:** [Troubleshoot issues using Performance Monitor](https://learn.microsoft.com/troubleshoot/windows-server/support-tools/troubleshoot-issues-performance-monitor#performance-counters-for-cpu-usage). % DPC Time: investigate when % Processor Time is above 85% and % DPC Time is above 15% for a sustained period. % Interrupt Time: above 30% suggests a hardware or driver problem; above 50% is very high.
- **Status:** source-checked.
- **Pitfall:** Short spikes are acceptable. The page says to investigate sustained levels. Use Process Monitor or ETW (WPR/WPA) to identify the driver.

### Interrupts/sec, DPCs Queued/sec (Processor Information)

- **Unit:** events per second.
- **Formula:** Interrupts/sec is the rate of device interrupts. DPCs Queued/sec is the rate at which DPCs are added to the logical processor's DPC queue: Δcount / Δt.
- **Source:** `Processor Information(_Total)\Interrupts/sec`, `\DPCs Queued/sec`.
- **Reference:** [Network-Related Performance Counters](https://learn.microsoft.com/windows-server/networking/technologies/network-subsystem/net-sub-performance-counters) (defines DPCs Queued/sec as a rate of additions, not the current queue length). Microsoft gives no numeric threshold for Interrupts/sec.
- **Status:** source-checked (definition). No threshold.
- **Pitfall:** DPCs Queued/sec counts additions, not the current queue length. Interrupt rate alone does not indicate a fault.

## Queueing and scheduling

### Processor Queue Length (System)

- **Unit:** threads (count).
- **Formula:** Threads waiting for a processor. Compare against the logical processor count.
- **Source:** `System\Processor Queue Length`.
- **Reference:** [Troubleshoot issues using Performance Monitor](https://learn.microsoft.com/troubleshoot/windows-server/support-tools/troubleshoot-issues-performance-monitor#performance-counters-for-cpu-usage). The page says: when there are two or more instances per processor and CPU usage is high, check processes for high CPU consumption, and also check Context Switches, % DPC Time, and % Interrupt Time. It gives no numeric threshold.
- **Status:** source-checked (no numeric threshold).
- **Pitfall:** A high value with high CPU usage indicates contention. Do not apply a fixed number from another product's documentation.

### Context Switches/sec, System Calls/sec (System)

- **Unit:** events per second.
- **Formula:** Δ count / Δt.
- **Source:** `System\Context Switches/sec`, `System\System Calls/sec`.
- **Reference:** [Troubleshoot issues using Performance Monitor](https://learn.microsoft.com/troubleshoot/windows-server/support-tools/troubleshoot-issues-performance-monitor#performance-counters-for-cpu-usage) (listed as CPU bottleneck counters; no threshold given).
- **Status:** source-checked (no threshold).
- **Pitfall:** No universal threshold. Compare against a baseline from the same workload.

## Process CPU

### % Processor Time (Process)

- **Unit:** percent. Range is 0 to 100 × logical processor count.
- **Formula:** Sum of processor usage by all threads of the process, as a percentage of elapsed time.
- **Source:** `Process(<name>)\% Processor Time`.
- **Reference:** [Collecting Performance Data](https://learn.microsoft.com/windows/win32/perfctrs/collecting-performance-data#understanding-multiple-processor-counters) (range 0 to 100 × ProcessorCount; PDH matches samples by instance name, so terminated processes can shift samples between instances; Windows 11 adds the `Process V2` counter set, which includes the process ID in the instance name).
- **Status:** source-checked.
- **Pitfall:** Values above 100 are normal on multi-threaded processes. Instance names repeat across processes (`svchost#1`), and a restarted process can shift samples between instances. Use the `Process V2` counter set on Windows 11, or PID-based queries, when accuracy matters.

### Thread Count, Handle Count (Process)

- **Unit:** count (gauge).
- **Source:** `Process(_Total)\Thread Count`, `Process(_Total)\Handle Count`; per process: `Process(<name>)\Thread Count`.
- **Reference:** [Troubleshoot issues using Performance Monitor](https://learn.microsoft.com/troubleshoot/windows-server/support-tools/troubleshoot-issues-performance-monitor#performance-counters-for-cpu-usage) (lists Thread Count and Handle Count as process counters to monitor). WMI `Win32_Process` (`ThreadCount`, `HandleCount`) is listed in the [Win32_Process class reference](https://learn.microsoft.com/windows/win32/cimwin32prov/win32-process); field names were not checked on this pass.
- **Status:** source-checked (counter names). Field names partly checked.
- **Pitfall:** Handle and thread counts grow with workload. A steady climb without workload change suggests a leak.

### Process CPU time (100-ns ticks)

- **Unit:** 100-ns ticks (TimeSpan ticks). 10,000,000 ticks = 1 s.
- **Formula:** `UserProcessorTime`, `PrivilegedProcessorTime`, `TotalProcessorTime` are cumulative. Rate = Δticks / Δt / 10,000,000.
- **Source:** .NET `System.Diagnostics.Process` properties; WMI `Win32_Process` (`KernelModeTime`, `UserModeTime` in 100-ns units).
- **Reference:** [Win32_Process class](https://learn.microsoft.com/windows/win32/cimwin32prov/win32-process) (KernelModeTime, UserModeTime). Field units were not checked on this pass.
- **Status:** partly checked.
- **Pitfall:** Values are cumulative since process start. Divide by the process lifetime or Δt before comparing.

## Throttling and pressure

### Processor Frequency, % of Maximum Frequency (Processor Information)

- **Unit:** MHz (frequency), percent (of maximum).
- **Source:** `Processor Information(_Total)\Processor Frequency`, `\% of Maximum Frequency`.
- **Reference:** [Win32_Processor class](https://learn.microsoft.com/windows/win32/cimwin32prov/win32-processor) (`CurrentClockSpeed`, `MaxClockSpeed`). The counter names were not checked on this pass.
- **Status:** partly checked.
- **Pitfall:** Frequency drops under power management. A low frequency with high utilization can mean power plan throttling, not a workload change.

## Physical CPU identity

### Logical processors and cores (Win32_Processor)

- **Unit:** count.
- **Source:** WMI `Win32_Processor` (`NumberOfCores`, `NumberOfLogicalProcessors`).
- **Reference:** [Win32_Processor class](https://learn.microsoft.com/windows/win32/cimwin32prov/win32-processor). Field names were not checked on this pass.
- **Status:** partly checked.
- **Pitfall:** `NumberOfLogicalProcessors` counts hyperthreads. Core count and logical processor count differ on SMT systems.

## Windows Internals references

- **KPCR (Kernel Processor Control Region):** per-processor kernel state, including the current thread and DPC queue. Reference: *Windows Internals* (kernel chapter). Not a public API.
- **KDPC / DPC queue:** each processor has its own DPC queue (see DPCs Queued/sec). Reference: *Windows Internals*.
- **Sysinternals:** [Process Explorer](https://learn.microsoft.com/en-us/sysinternals/downloads/process-explorer) (per-process CPU, threads); [Process Monitor](https://learn.microsoft.com/en-us/sysinternals/downloads/procmon) (driver-level activity).
