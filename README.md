# Windows Metrics Reference

Formula, unit, source, and reference for Windows system metrics. Covers CPU, memory, disk, network, system, and GPU.

Scope excludes cgroups (no Windows equivalent). Windows job objects are noted only where they change a counter's meaning.

## Verification status

Each entry carries one of these **Status** values:

- **source-checked:** the Microsoft Learn page cited for the claim was read on 2026-10-29 and supports it. Threshold values are copied from that page.
- **partly checked:** the cited page resolves, but the specific claim (field name, option, or default) was not read.
- **unverified:** no primary Microsoft source was found for the claim. Treat it as an assumption.

No entry has been confirmed on a live Windows system yet. To verify on a live machine, use `Get-Counter`, `Get-CimInstance`, or Performance Monitor (`perfmon`), and record the Windows build (`[System.Environment]::OSVersion` or `winver`) with the result.

Source-by-source results, including links that were removed and why, are in [SOURCES.md](SOURCES.md). Terms are defined in [GLOSSARY.md](GLOSSARY.md).

## Primary sources

Start here before adding a new threshold. Prefer these over product-specific pages.

- [Troubleshoot issues using Performance Monitor](https://learn.microsoft.com/troubleshoot/windows-server/support-tools/troubleshoot-issues-performance-monitor): CPU counters and thresholds.
- [Scenario guide: Troubleshoot performance problems in Windows](https://learn.microsoft.com/troubleshoot/windows-server/performance/troubleshoot-performance-problems-in-windows): the 18 core counters with healthy, warning, and critical thresholds.
- [Network-Related Performance Counters](https://learn.microsoft.com/windows-server/networking/technologies/network-subsystem/net-sub-performance-counters): network counter families.
- [Memory Performance Information](https://learn.microsoft.com/windows/win32/memory/memory-performance-information): mapping of memory counters to API structures and Task Manager.
- [Collecting Performance Data](https://learn.microsoft.com/windows/win32/perfctrs/collecting-performance-data): how PDH computes `_Total` and `Process` values.
- [Performance Counters Portal](https://learn.microsoft.com/windows/win32/perfctrs/performance-counters-portal): index of counter objects.
- [Profile the CPU and GPU with timing captures (PIX)](https://learn.microsoft.com/windows/pix/timing-captures/timing-captures-overview): GPU counters and residency.
- [Win32 CIM provider classes](https://learn.microsoft.com/windows/win32/cimwin32prov/win32-operatingsystem): WMI fields such as `Win32_OperatingSystem`, `Win32_Process`, and `Win32_Processor`.

## Page format

Each metric entry gives:

- **Unit**
- **Formula**
- **Source** (counter set, WMI/CIM class, API, or tool)
- **Reference** (counter path, WMI/CIM class, or Microsoft Learn URL)
- **Pitfall**
- **Status** (`source-checked`, `partly checked`, or `unverified`, as defined above)

## Conventions

- **Δ**: difference between two samples of a cumulative counter.
- **Δt**: elapsed time between samples. Use the sample timestamp, not the requested interval.
- **Rate**: Δcounter / Δt.
- **Raw vs formatted**: PDH converts raw counters (for example `PERF_100NSEC_TIMER_INV`) into formatted percentages. Formatted values are used unless noted.
- **100-ns ticks**: Windows timer units used by CPU time fields (for example `Process.TotalProcessorTime`, `Win32_PerfRawData` timers). 10,000,000 ticks = 1 second.
- **Bytes and MB**: memory counters report bytes. `Available MBytes` reports MB. Confirm the unit in each entry.
- **Instances**: `_Total` aggregates all instances. `Process(*)` and `Network Interface(*)` return one sample per instance. Instance names can change when processes restart.
- **Counters are cumulative** unless the entry marks them as a rate, gauge, or average.

## Index

| Folder | Scope |
|---|---|
| [cpu/](cpu/README.md) | Processor time, DPC and interrupt time, queue length, scheduler, process CPU |
| [memory/](memory/README.md) | Available memory, commit charge, cache and standby lists, pool, paging, process memory |
| [disk/](disk/README.md) | Physical and logical disk latency, queue, throughput, free space |
| [network/](network/README.md) | Interface throughput, IP, TCP, UDP, output queue, errors |
| [system/](system/README.md) | OS identity, uptime, handles, threads, processes, time service, hardware errors |
| [gpu/](gpu/README.md) | GPU engine utilization, adapter memory, process GPU memory (WDDM) |

## Page format

Each metric entry gives:

- **Unit**
- **Formula**
- **Source** (counter set, WMI/CIM class, API, or tool)
- **Reference** (counter path, WMI/CIM class, or Microsoft Learn URL)
- **Pitfall**
- **Status** (always `unverified` for now)

## Sysinternals and Windows Internals

Internal structures (KPCR, KTHREAD, DPC queue, PFN database, standby lists) are used to explain counter behavior. They are not directly queryable through a stable public API. Each such reference cites *Windows Internals* (Russinovich, Solomon, Ionescu) or a Sysinternals tool that shows the structure.

- Sysinternals: https://learn.microsoft.com/en-us/sysinternals/
- Windows Internals resources: https://learn.microsoft.com/en-us/sysinternals/resources/windows-internals
