# System metrics

OS identity, uptime, process and thread counts, handles, time synchronization, and hardware error reporting for Windows.

Status: mixed. Counter names for processes, threads, and handles are source-checked against Microsoft Learn on 2026-10-29. Other entries are partly checked or unverified, as marked. None has been verified on a live machine. Status values are defined in the [root README](../README.md#verification-status).

## OS identity

### OS build and version (WMI)

- **Unit:** string.
- **Source:** WMI `Win32_OperatingSystem` (`Caption`, `Version`, `BuildNumber`, `OSArchitecture`).
- **Reference:** [Win32_OperatingSystem class](https://learn.microsoft.com/windows/win32/cimwin32prov/win32-operatingsystem). Field names were not checked on this pass.
- **Status:** partly checked.
- **Pitfall:** `Version` is the major.minor.build string. Map build numbers to releases before comparing behavior across machines.

## Uptime

### System Up Time (System)

- **Unit:** seconds.
- **Formula:** Time since last boot = current time − `LastBootUpTime` (WMI).
- **Source:** `System\System Up Time`; WMI `Win32_OperatingSystem` (`LastBootUpTime`).
- **Reference:** [Win32_OperatingSystem class](https://learn.microsoft.com/windows/win32/cimwin32prov/win32-operatingsystem) (`LastBootUpTime`); [Performance counters portal](https://learn.microsoft.com/windows/win32/perfctrs/performance-counters-portal) (System object).
- **Status:** partly checked.
- **Pitfall:** Fast Startup (hybrid shutdown) and hibernation mean "uptime" does not always match the last full restart. Confirm the boot type before treating uptime as a stability signal.

## Processes, threads, and handles

### Processes, Threads (Objects, Process _Total)

- **Unit:** count (gauge).
- **Source:** `Objects\Processes` (Process count); `Process(_Total)\Thread Count`.
- **Reference:** [Memory Performance Information](https://learn.microsoft.com/windows/win32/memory/memory-performance-information) (table maps `Processes (Objects object)` to `PERFORMANCE_INFORMATION.ProcessCount` and `Thread Count (Process(_Total) object)` to `ThreadCount`).
- **Status:** source-checked.
- **Pitfall:** Counts change with services and scheduled tasks. Compare them under the same workload.

### Handle Count (Process, _Total)

- **Unit:** count (gauge).
- **Source:** `Process(_Total)\Handle Count`; per process: `Process(<name>)\Handle Count`.
- **Reference:** [Troubleshoot issues using Performance Monitor](https://learn.microsoft.com/troubleshoot/windows-server/support-tools/troubleshoot-issues-performance-monitor#performance-counters-for-cpu-usage) (lists Handle Count among the process counters to monitor). [Memory Performance Information](https://learn.microsoft.com/windows/win32/memory/memory-performance-information) (`PERFORMANCE_INFORMATION.HandleCount` for the system total).
- **Status:** source-checked.
- **Pitfall:** A steady increase with a stable workload suggests a handle leak. Use Sysinternals Handle to see which handle types are growing.

### Sysinternals Handle

- **Unit:** count by handle type.
- **Source:** Sysinternals `handle.exe` (`-p <pid>` to filter).
- **Reference:** [Sysinternals Handle](https://learn.microsoft.com/en-us/sysinternals/downloads/handle).
- **Status:** partly checked (link resolves; options not read).
- **Pitfall:** Listing handles for all processes requires an elevated (administrator) session.

## Time synchronization

### Windows Time service status

- **Unit:** string / seconds (offset).
- **Source:** `w32tm /query /status` (Windows Time service); `w32tm /stripchart` for offset over time.
- **Reference:** [Windows Time service tools and settings](https://learn.microsoft.com/windows-server/networking/windows-time-service/windows-time-service-tools-and-settings). Command options were not checked on this pass.
- **Status:** partly checked.
- **Pitfall:** Offset values depend on the configured time source and network latency. The time service does not write to the performance counters.

## Hardware and driver errors

### Hardware errors (WHEA)

- **Unit:** events (count per log entry).
- **Source:** Windows Event Log, channel `System`, provider `Microsoft-Windows-WHEA-Logger`.
- **Reference:** [Introduction to the Windows Hardware Error Architecture](https://learn.microsoft.com/windows-hardware/drivers/whea/introduction-to-the-windows-hardware-error-architecture). The event provider name was not checked on this pass.
- **Status:** partly checked.
- **Pitfall:** Corrected errors are logged even when the system recovers. Correlate them with CPU, memory, or PCIe device data before replacing hardware.

### Kernel crashes (bugcheck)

- **Unit:** event (count).
- **Source:** Bugcheck event in the `System` log. Dump files are written under `%SystemRoot%` (for example `MEMORY.DMP` or `Minidump`). The event ID and dump paths were not checked on this pass.
- **Reference:** [Bug Check Code Reference](https://learn.microsoft.com/windows-hardware/drivers/debugger/bug-check-code-reference2).
- **Status:** partly checked.
- **Pitfall:** The bugcheck code is the first triage signal. Do not treat the event count as a reliability score without the codes.

## Event logs

### Error and critical events per hour

- **Unit:** events per hour.
- **Formula:** Count of events with Level 1 (Critical) or 2 (Error) in the window / hours.
- **Source:** `Get-WinEvent` with `-FilterHashtable @{LogName='System'; Level=1,2; StartTime=...}`.
- **Reference:** [Get-WinEvent](https://learn.microsoft.com/powershell/module/microsoft.powershell.diagnostics/get-winevent). Parameter behavior was not checked on this pass.
- **Status:** partly checked.
- **Pitfall:** Event volume depends on logging configuration and sources. Compare trends only on the same host and log set.

## Windows Internals references

- **Processes and threads:** EPROCESS and ETHREAD structures hold per-process state; the counters above read them. Reference: *Windows Internals* (processes and threads chapter). Not a public API.
- **Handle table:** per-process handle table maps handles to kernel objects. Reference: *Windows Internals* (object manager chapter). Not a public API.
- **Sysinternals Process Explorer:** process tree and handles. [Sysinternals Process Explorer](https://learn.microsoft.com/en-us/sysinternals/downloads/process-explorer).
