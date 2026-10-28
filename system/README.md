# System metrics

OS identity, uptime, process and thread counts, handles, time synchronization, and hardware error reporting for Windows.

Status: all entries unverified.

## OS identity

### OS build and version (WMI)

- **Unit:** string.
- **Source:** WMI `Win32_OperatingSystem` (`Caption`, `Version`, `BuildNumber`, `OSArchitecture`).
- **Reference:** https://learn.microsoft.com/windows/win32/cimwin32prov/win32-operatingsystem
- **Pitfall:** `Version` is the major.minor.build string. Map build numbers to releases before comparing behavior across machines.

## Uptime

### System Up Time (System)

- **Unit:** seconds.
- **Formula:** Time since last boot = current time − `LastBootUpTime` (WMI).
- **Source:** `System\System Up Time`; WMI `Win32_OperatingSystem` (`LastBootUpTime`).
- **Reference:** https://learn.microsoft.com/windows/win32/cimwin32prov/win32-operatingsystem (LastBootUpTime); https://learn.microsoft.com/windows/win32/perfctrs/performance-counters-portal (System object).
- **Pitfall:** Fast Startup (hybrid shutdown) resets the counter differently from a full restart. A system that appears "up" for days may have hibernated.

## Processes, threads, and handles

### Processes, Threads (System)

- **Unit:** count (gauge).
- **Source:** `System\Processes`, `System\Threads`; WMI `Win32_PerfFormattedData_PerfOS_System`.
- **Reference:** https://learn.microsoft.com/troubleshoot/windows-server/performance/memory-consumption-between-identical-windows-server-environments#scenario-guide-compare-systems-using-the-performance-counters (System\Processes used as a load proxy).
- **Pitfall:** Count changes with services and scheduled tasks; compare under the same workload.

### Handle Count (Process, _Total)

- **Unit:** count (gauge).
- **Source:** `Process(_Total)\Handle Count`; per process `Process(<name>)\Handle Count`.
- **Reference:** https://learn.microsoft.com/troubleshoot/windows-server/support-tools/troubleshoot-issues-performance-monitor#performance-counters-for-cpu-usage (lists Handle Count as a process counter to monitor).
- **Pitfall:** Handle leaks grow without bound. A steady increase with stable workload indicates a leak. Sysinternals Handle shows the handle types.

### Sysinternals Handle

- **Unit:** count by handle type.
- **Source:** Sysinternals `handle.exe` (`-p <pid>` to filter).
- **Reference:** https://learn.microsoft.com/en-us/sysinternals/downloads/handle
- **Pitfall:** Requires administrator rights to list handles for all processes.

## Time synchronization

### Windows Time service status

- **Unit:** string / seconds (offset).
- **Source:** `w32tm /query /status` (Windows Time service); `w32tm /stripchart` for offset over time.
- **Reference:** https://learn.microsoft.com/windows-server/networking/windows-time-service/windows-time-service-tools-and-settings
- **Pitfall:** Time service runs independently of the performance counters. Offset values depend on the configured time source and network latency.

## Hardware and driver errors

### Hardware errors (WHEA)

- **Unit:** events (count per log entry).
- **Source:** Windows Event Log, channel `System`, provider `Microsoft-Windows-WHEA-Logger`.
- **Reference:** https://learn.microsoft.com/windows-hardware/drivers/whea/introduction-to-the-windows-hardware-error-architecture (WHEA overview).
- **Pitfall:** Corrected errors are logged even when the system recovers. Correlate with CPU, memory, or PCIe device before replacing hardware.

### Kernel crashes (bugcheck)

- **Unit:** event (count).
- **Source:** Event ID 1001 (BugCheck) in `System`; crash dump files in `%SystemRoot%\MEMORY.DMP` or `%SystemRoot%\Minidump`.
- **Reference:** https://learn.microsoft.com/windows-hardware/drivers/debugger/bug-check-code-reference2
- **Pitfall:** The bugcheck code is the first triage signal. Do not treat the event count as a reliability score without the codes.

## Event logs

### Error and critical events per hour

- **Unit:** events per hour.
- **Formula:** Count of events with Level 1 (Critical) or 2 (Error) in the window / hours.
- **Source:** `Get-WinEvent` with `-FilterHashtable @{LogName='System'; Level=1,2; StartTime=...}`.
- **Reference:** https://learn.microsoft.com/powershell/module/microsoft.powershell.diagnostics/get-winevent
- **Pitfall:** Event volume depends on logging configuration and sources. Compare trends only on the same host and log set.

## Windows Internals references

- **Processes and threads:** EPROCESS and ETHREAD structures hold per-process state; the counters above read them. Reference: *Windows Internals* (processes and threads chapter).
- **Handle table:** per-process handle table maps handles to kernel objects. Reference: *Windows Internals* (object manager chapter).
- **Sysinternals references:** Process Explorer (process tree and handles), Handle (handle listing). https://learn.microsoft.com/en-us/sysinternals/downloads/process-explorer
