# Disk metrics

Physical and logical disk utilization, latency, queueing, throughput, and free space for Windows.

Status: all entries unverified.

Use **PhysicalDisk** counters for hardware-level analysis. Use **LogicalDisk** counters for volume-level analysis (software RAID, Storage Spaces). Both objects expose the same counter names. Source: https://learn.microsoft.com/biztalk/technical-guides/system-level-bottlenecks#performance-counters-for-measuring-disk-io-bottlenecks

## Utilization and latency

### % Disk Time (PhysicalDisk)

- **Unit:** percent.
- **Formula:** Time the disk was busy servicing requests / elapsed time × 100.
- **Source:** `PhysicalDisk(_Total)\% Disk Time`.
- **Reference:** https://learn.microsoft.com/sharepoint/administration/storage-and-sql-server-capacity-planning-and-configuration#validate-and-monitor-storage-and-sql-server-performance (more than 90% is a busy disk).
- **Pitfall:** Can exceed 100% on some Windows versions (multiple requests in flight). Check queue length and latency alongside it.

### Avg. Disk sec/Read, Avg. Disk sec/Write (PhysicalDisk)

- **Unit:** seconds per operation (display in ms).
- **Formula:** Total read (write) time / read (write) operations.
- **Source:** `PhysicalDisk(_Total)\Avg. Disk sec/Read`, `\Avg. Disk sec/Write`.
- **Reference:** https://learn.microsoft.com/biztalk/technical-guides/system-level-bottlenecks#high-level-system-bottlenecks (less than 10 ms good, 15–25 ms fair, above 25 ms poor); https://learn.microsoft.com/azure/virtual-machines/premium-storage-performance#application-performance-requirements-checklist
- **Pitfall:** Thresholds in the sources are for spinning disks and mixed workloads. Flash and cloud disks have different targets.

### Avg. Disk sec/Transfer (LogicalDisk)

- **Unit:** seconds per operation.
- **Formula:** (Read time + write time) / total operations.
- **Source:** `LogicalDisk(<drive>)\Avg. Disk sec/Transfer`.
- **Reference:** https://learn.microsoft.com/azure/virtual-desktop/insights-glossary#performance-counters
- **Pitfall:** Value is a mean over the sample interval; a short burst can be hidden.

## Queueing

### Current Disk Queue Length (PhysicalDisk, LogicalDisk)

- **Unit:** requests (instantaneous).
- **Formula:** Requests outstanding at the moment of sampling.
- **Source:** `PhysicalDisk(_Total)\Current Disk Queue Length`, `LogicalDisk(<drive>)\Current Disk Queue Length`.
- **Reference:** https://learn.microsoft.com/sharepoint/administration/storage-and-sql-server-capacity-planning-and-configuration#validate-and-monitor-storage-and-sql-server-performance (above 2 per disk may indicate a bottleneck).
- **Pitfall:** Guidance is for spindle-based arrays. Value scales with spindle count. Not a reliable bottleneck signal for SQL Server, which issues asynchronous I/O (see MSSQLSERVER_833 notes).

### Avg. Disk Queue Length, Avg. Disk Read/Write Queue Length (PhysicalDisk)

- **Unit:** requests (average over interval).
- **Formula:** Time-weighted average of requests in flight below the driver.
- **Source:** `PhysicalDisk(_Total)\Avg. Disk Queue Length`, `\Avg. Disk Read Queue Length`, `\Avg. Disk Write Queue Length`.
- **Reference:** https://learn.microsoft.com/biztalk/technical-guides/system-level-bottlenecks#high-level-system-bottlenecks (threshold: no more than number of spindles plus two).
- **Pitfall:** Requests counted include those in service and in the controller, not only queued ones.

## Throughput

### Disk Bytes/sec, Disk Read Bytes/sec, Disk Write Bytes/sec (PhysicalDisk)

- **Unit:** bytes per second.
- **Formula:** Δbytes / Δt.
- **Source:** `PhysicalDisk(_Total)\Disk Bytes/sec`, `\Disk Read Bytes/sec`, `\Disk Write Bytes/sec`.
- **Reference:** https://learn.microsoft.com/azure/virtual-machines/premium-storage-performance#application-performance-requirements-checklist (throughput counters).
- **Pitfall:** Compare against the disk's rated throughput, not against a generic number.

### Disk Transfers/sec, Disk Reads/sec, Disk Writes/sec (PhysicalDisk, LogicalDisk)

- **Unit:** operations per second (IOPS).
- **Formula:** Δoperations / Δt.
- **Source:** `PhysicalDisk(_Total)\Disk Transfers/sec`, `\Disk Reads/sec`, `\Disk Writes/sec`.
- **Reference:** https://learn.microsoft.com/sharepoint/administration/storage-and-sql-server-capacity-planning-and-configuration#validate-and-monitor-storage-and-sql-server-performance
- **Pitfall:** IOPS depends on I/O size. Pair with Avg. Disk Bytes/Read and Avg. Disk Bytes/Write.

### Avg. Disk Bytes/Read, Avg. Disk Bytes/Write (LogicalDisk)

- **Unit:** bytes per operation.
- **Formula:** Total bytes / operation count over interval.
- **Source:** `LogicalDisk(<drive>)\Avg. Disk Bytes/Read`, `\Avg. Disk Bytes/Write`.
- **Reference:** https://learn.microsoft.com/sharepoint/administration/storage-and-sql-server-capacity-planning-and-configuration#validate-and-monitor-storage-and-sql-server-performance
- **Pitfall:** Large I/O sizes raise latency slightly. A small average can mean many small operations (metadata-heavy workloads).

## Volume capacity

### % Free Space, Free Megabytes (LogicalDisk)

- **Unit:** percent, MB.
- **Formula:** Free bytes / total bytes × 100.
- **Source:** `LogicalDisk(<drive>)\% Free Space`, `\Free Megabytes`; WMI `Win32_LogicalDisk` (`FreeSpace`, `Size`).
- **Reference:** https://learn.microsoft.com/azure/service-fabric/monitor-service-fabric-reference#performance-metrics (Logical Disk Free Space collected per node).
- **Pitfall:** NTFS reserves space; free space reported is available to users, not the raw volume total. Snapshots and shadow copies consume space without appearing as used files.

## Idle and busy

### % Idle Time (PhysicalDisk, LogicalDisk)

- **Unit:** percent.
- **Formula:** 100 − % Disk Time (for a single disk).
- **Source:** `PhysicalDisk(_Total)\% Idle Time`.
- **Reference:** https://learn.microsoft.com/azure/virtual-machines/premium-storage-performance (counter list).
- **Pitfall:** Derived from the same measurement as % Disk Time; not an independent signal.

## Windows Internals references

- **Storage stack:** I/O request packets (IRPs) pass through the filter drivers, the volume manager, and the port driver (storport). Queue counters sample this stack. Reference: *Windows Internals* (I/O system chapter).
- **Sysinternals Process Monitor:** per-file and per-process I/O trace. https://learn.microsoft.com/en-us/sysinternals/downloads/procmon
