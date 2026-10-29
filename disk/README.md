# Disk metrics

Physical and logical disk utilization, latency, queueing, throughput, and free space for Windows.

Status: source-checked against Microsoft Learn on 2026-10-29. Not yet verified on a live machine. Status values are defined in the [root README](../README.md#verification-status).

Use **PhysicalDisk** counters for hardware-level analysis. Use **LogicalDisk** counters for volume-level analysis (software RAID, Storage Spaces). Both objects expose the same counter names. The Windows primary guidance for disk counters is in the [scenario guide: Troubleshoot performance problems in Windows](https://learn.microsoft.com/troubleshoot/windows-server/performance/troubleshoot-performance-problems-in-windows#storage).

Thresholds in this folder come from Microsoft Learn. Where a source is product-specific (SharePoint, BizTalk, Operations Manager), the entry says so. Do not apply a product-specific threshold to general Windows hosts.

## Utilization and latency

### % Disk Time (PhysicalDisk)

- **Unit:** percent.
- **Formula:** Time the disk was busy servicing requests / elapsed time × 100.
- **Source:** `PhysicalDisk(_Total)\% Disk Time`.
- **Reference:** No primary Windows threshold. The [scenario guide](https://learn.microsoft.com/troubleshoot/windows-server/performance/troubleshoot-performance-problems-in-windows#storage) uses latency and queue counters as the primary signals.
- **Status:** partly checked (no threshold).
- **Pitfall:** Busy time does not show how much work is waiting. Check latency and queue length alongside it.

### Avg. Disk sec/Read, Avg. Disk sec/Write (PhysicalDisk, LogicalDisk)

- **Unit:** seconds per operation (display in ms).
- **Formula:** Total read (write) time / read (write) operations over the interval.
- **Source:** `PhysicalDisk(_Total)\Avg. Disk sec/Read`, `\Avg. Disk sec/Write`; `LogicalDisk(<drive>)\Avg. Disk sec/Read`, `\Avg. Disk sec/Write`.
- **Reference:** [Scenario guide: Troubleshoot performance problems in Windows](https://learn.microsoft.com/troubleshoot/windows-server/performance/troubleshoot-performance-problems-in-windows#storage) (healthy: under 15 ms; warning: over 25 ms; critical: over 50 ms. Short spikes are tolerated; investigate latency that lasts over one minute).
- **Status:** source-checked.
- **Pitfall:** Latency depends on the device. The guide's own example uses a 5,400 RPM spinning disk, where a single read takes about 11 ms. SSD and cloud disks have different targets.

### Avg. Disk sec/Transfer (LogicalDisk)

- **Unit:** seconds per operation.
- **Formula:** (Read time + write time) / total operations.
- **Source:** `LogicalDisk(<drive>)\Avg. Disk sec/Transfer`.
- **Reference:** Counter name listed in [System-Level Bottlenecks (BizTalk Server)](https://learn.microsoft.com/biztalk/technical-guides/system-level-bottlenecks#performance-counters-for-measuring-disk-io-bottlenecks). No Windows threshold was found.
- **Status:** partly checked.
- **Pitfall:** Value is a mean over the sample interval; a short burst can be hidden.

## Queueing

### Current Disk Queue Length (PhysicalDisk, LogicalDisk)

- **Unit:** requests (instantaneous).
- **Formula:** Requests outstanding at the moment of sampling.
- **Source:** `PhysicalDisk(_Total)\Current Disk Queue Length`, `LogicalDisk(<drive>)\Current Disk Queue Length`.
- **Reference:** No Windows primary threshold. SharePoint Server capacity guidance says values above 2 per disk may indicate a bottleneck ([Storage and SQL Server capacity planning](https://learn.microsoft.com/sharepoint/administration/storage-and-sql-server-capacity-planning-and-configuration#validate-and-monitor-storage-and-sql-server-performance)). This is SharePoint-scoped.
- **Status:** partly checked (product-scoped threshold only).
- **Pitfall:** The value scales with the number of spindles behind the disk. Not a reliable bottleneck signal for SQL Server, which issues asynchronous I/O and can show long queues without a problem ([MSSQLSERVER_833](https://learn.microsoft.com/sql/relational-databases/errors-events/mssqlserver-833-database-engine-error?view=sql-server-ver17#more-information)).

### Avg. Disk Queue Length, Avg. Disk Read/Write Queue Length (PhysicalDisk)

- **Unit:** requests (average over the interval).
- **Formula:** Average number of requests in flight below the driver during the sample interval. The count includes requests in service and in the controller, not only queued ones.
- **Source:** `PhysicalDisk(_Total)\Avg. Disk Queue Length`, `\Avg. Disk Read Queue Length`, `\Avg. Disk Write Queue Length`.
- **Reference:** Product-scoped guidance only:
  - [Troubleshoot event ID 2115-related performance problems (Operations Manager)](https://learn.microsoft.com/troubleshoot/system-center/scom/troubleshoot-event-2115-related-performance-problems#resolution): Avg. Disk Queue Length should not exceed twice the number of spindles on a LUN. This page gives the RAID 1, 10, and 5 formulas inconsistently with the page below.
  - [Troubleshoot gray agent states (Operations Manager)](https://learn.microsoft.com/troubleshoot/system-center/scom/troubleshoot-gray-agent-states#troubleshooting-sql-server-performance): gives the RAID 5 formula as disks × 2, which conflicts with the 2115 page.
  - [System-Level Bottlenecks (BizTalk Server)](https://learn.microsoft.com/biztalk/technical-guides/system-level-bottlenecks#high-level-system-bottlenecks): Avg. Disk Queue Length should not be higher than the number of spindles plus two.
- **Status:** partly checked (product-scoped; sources disagree).
- **Pitfall:** Use these only for a comparable storage layout, and record which product guidance you used. The formulas conflict, so do not present one as the Windows rule.

## Throughput

### Disk Bytes/sec, Disk Read Bytes/sec, Disk Write Bytes/sec (PhysicalDisk)

- **Unit:** bytes per second.
- **Formula:** Δbytes / Δt.
- **Source:** `PhysicalDisk(_Total)\Disk Bytes/sec`, `\Disk Read Bytes/sec`, `\Disk Write Bytes/sec`.
- **Reference:** Counter names listed in [System-Level Bottlenecks (BizTalk Server)](https://learn.microsoft.com/biztalk/technical-guides/system-level-bottlenecks#performance-counters-for-measuring-disk-io-bottlenecks). No threshold.
- **Status:** partly checked (no threshold).
- **Pitfall:** Compare against the disk's rated throughput, not against a generic number.

### Disk Transfers/sec, Disk Reads/sec, Disk Writes/sec (PhysicalDisk, LogicalDisk)

- **Unit:** operations per second (IOPS).
- **Formula:** Δoperations / Δt.
- **Source:** `PhysicalDisk(_Total)\Disk Transfers/sec`, `\Disk Reads/sec`, `\Disk Writes/sec`.
- **Reference:** No Windows threshold. Counter named in [Storage and SQL Server capacity planning](https://learn.microsoft.com/sharepoint/administration/storage-and-sql-server-capacity-planning-and-configuration#validate-and-monitor-storage-and-sql-server-performance) for growth trending.
- **Status:** partly checked (no threshold).
- **Pitfall:** IOPS depends on I/O size. Pair with Avg. Disk Bytes/Read and Avg. Disk Bytes/Write.

### Avg. Disk Bytes/Read, Avg. Disk Bytes/Write (LogicalDisk)

- **Unit:** bytes per operation.
- **Formula:** Total bytes / operation count over the interval.
- **Source:** `LogicalDisk(<drive>)\Avg. Disk Bytes/Read`, `\Avg. Disk Bytes/Write`.
- **Reference:** [Storage and SQL Server capacity planning](https://learn.microsoft.com/sharepoint/administration/storage-and-sql-server-capacity-planning-and-configuration#validate-and-monitor-storage-and-sql-server-performance) (larger operations can slightly raise latency). SharePoint-scoped.
- **Status:** partly checked.
- **Pitfall:** A small average can mean many small operations, which is common for metadata-heavy workloads.

## Volume capacity

### % Free Space, Free Megabytes (LogicalDisk)

- **Unit:** percent, MB.
- **Formula:** Free bytes / total bytes × 100.
- **Source:** `LogicalDisk(<drive>)\% Free Space`, `\Free Megabytes`; WMI `Win32_LogicalDisk` (`FreeSpace`, `Size`).
- **Reference:** Counter names only. No Microsoft threshold was checked. The [Win32_LogicalDisk class](https://learn.microsoft.com/windows/win32/cimwin32prov/win32-logicaldisk) is the WMI reference for the fields.
- **Status:** unverified (no threshold).
- **Pitfall:** Free space reported to users can differ from the raw volume total. Snapshots and shadow copies use space without appearing as files.

## Idle and busy

### % Idle Time (PhysicalDisk, LogicalDisk)

- **Unit:** percent.
- **Formula:** 100 − % Disk Time for a single disk.
- **Source:** `PhysicalDisk(_Total)\% Idle Time`.
- **Reference:** Counter name only. Not checked against a Microsoft threshold.
- **Status:** unverified.
- **Pitfall:** Derived from the same measurement as % Disk Time; not an independent signal.

## Windows Internals references

- **Storage stack:** I/O request packets (IRPs) pass through filter drivers, the volume manager, and the port driver (storport). Queue counters sample this stack. Reference: *Windows Internals* (I/O system chapter). Not a public API.
- **Sysinternals Process Monitor:** per-file and per-process I/O trace. [Sysinternals Process Monitor](https://learn.microsoft.com/en-us/sysinternals/downloads/procmon).
