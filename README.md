# Windows Metrics Reference

Formula, unit, source, and reference for Windows system metrics. Covers CPU, memory, disk, network, system, and GPU.

Scope excludes cgroups (no Windows equivalent). Windows job objects are noted only where they change a counter's meaning.

## Verification status

- **Status of every claim: unverified.** Entries were written from Microsoft Learn documentation and the *Windows Internals* book references. No live Windows system was queried.
- Each entry gives a **Reference** (counter path, WMI/CIM class, or Microsoft Learn URL) for verification.
- Verify on a live machine with `Get-Counter`, `Get-CimInstance`, or Performance Monitor (`perfmon`).
- Record the Windows build (`[System.Environment]::OSVersion` or `winver`) with any verification result.

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
