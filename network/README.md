# Network metrics

Interface throughput, IP, TCP, UDP, output queue, errors, and RSC for Windows.

Status: all entries unverified.

Source for grouping: https://learn.microsoft.com/windows-server/networking/technologies/network-subsystem/net-sub-performance-counters

## Interface throughput

### Bytes Total/sec (Network Interface)

- **Unit:** bytes per second.
- **Formula:** Δ(bytes received + bytes sent) / Δt.
- **Source:** `Network Interface(<adapter>)\Bytes Total/sec`; `Bytes Received/sec`, `Bytes Sent/sec`.
- **Reference:** https://learn.microsoft.com/biztalk/technical-guides/checklist-measuring-performance-on-hyper-v#measuring-network-performance (utilization = bytes × 8 / adapter bandwidth).
- **Pitfall:** Counters include framing overhead. Utilization = (Bytes Total/sec × 8) / Current Bandwidth × 100. Thresholds in the source: under 40% healthy, 41–64% monitor, 65–100% critical.

### Current Bandwidth (Network Interface, WMI)

- **Unit:** bits per second.
- **Source:** WMI `Win32_NetworkAdapter` (`Speed`); `Network Interface(<adapter>)\Current Bandwidth`.
- **Reference:** https://learn.microsoft.com/biztalk/technical-guides/using-the-performance-analysis-of-logs-pal-tool#understanding-the-pal-tool-report-for-microsoft-biztalk-server-2010 (network utilization analysis uses current bandwidth).
- **Pitfall:** Speed reports link speed negotiated. A half-duplex fallback shows lower bandwidth and can cut throughput.

### Packets Received/sec, Packets Sent/sec (Network Interface)

- **Unit:** packets per second.
- **Source:** `Network Interface(<adapter>)\Packets Received/sec`, `\Packets Sent/sec`.
- **Reference:** https://learn.microsoft.com/windows-server/networking/technologies/network-subsystem/net-sub-performance-counters#resource-utilization
- **Pitfall:** Small-packet workloads saturate the CPU before bandwidth. Check Processor Information DPC time (see cpu/) alongside.

## Queueing

### Output Queue Length (Network Interface)

- **Unit:** packets (average over interval).
- **Formula:** Packets waiting in the NDIS output queue.
- **Source:** `Network Interface(<adapter>)\Output Queue Length`.
- **Reference:** https://learn.microsoft.com/windows-server/networking/technologies/network-subsystem/net-sub-performance-counters#resource-utilization (queue longer than 2 causes delays); https://learn.microsoft.com/biztalk/technical-guides/checklist-measuring-performance-on-hyper-v#measuring-network-performance (0 healthy, 1–2 caution, above 2 critical).
- **Pitfall:** The source says NDIS queues requests, so a healthy value is 0. A nonzero value is a bottleneck signal, not a normal state.

## Errors and discards

### Packets Received Errors, Packets Outbound Errors (Network Interface)

- **Unit:** packets (count).
- **Formula:** Δerrors / Δt.
- **Source:** `Network Interface(<adapter>)\Packets Received Errors`, `\Packets Outbound Errors`.
- **Reference:** https://learn.microsoft.com/azure/service-fabric/monitor-service-fabric-reference#performance-metrics
- **Pitfall:** Any sustained nonzero rate needs investigation at the driver or cabling layer.

### Packets Received Discarded, Packets Outbound Discarded (Network Interface)

- **Unit:** packets (count).
- **Source:** `Network Interface(<adapter>)\Packets Received Discarded`, `\Packets Outbound Discarded`.
- **Reference:** https://learn.microsoft.com/azure/service-fabric/monitor-service-fabric-reference#performance-metrics
- **Pitfall:** Discards happen when buffers are full or filters reject packets. Compare with Output Queue Length.

## IP

### Datagrams Received/sec, Datagrams Sent/sec (IPv4, IPv6)

- **Unit:** datagrams per second.
- **Source:** `IPv4\Datagrams Received/sec`, `IPv4\Datagrams Sent/sec`; same for `IPv6`.
- **Reference:** https://learn.microsoft.com/windows-server/networking/technologies/network-subsystem/net-sub-performance-counters#resource-utilization
- **Pitfall:** Counts include ICMP and multicast on that stack. Not a per-application measure.

## TCP

### Segments Received/sec, Segments Sent/sec (TCPv4, TCPv6)

- **Unit:** segments per second.
- **Source:** `TCPv4\Segments Received/sec`, `TCPv4\Segments Sent/sec`.
- **Reference:** https://learn.microsoft.com/windows-server/networking/technologies/network-subsystem/net-sub-performance-counters#resource-utilization
- **Pitfall:** Segment size varies with MSS and offload. Interpret rates with Bytes Total/sec.

### Segments Retransmitted/sec (TCPv4, TCPv6)

- **Unit:** segments per second.
- **Formula:** Δretransmitted segments / Δt. Retransmission ratio = Segments Retransmitted/sec / Segments Sent/sec.
- **Source:** `TCPv4\Segments Retransmitted/sec`.
- **Reference:** https://learn.microsoft.com/windows-server/networking/technologies/network-subsystem/net-sub-performance-counters#resource-utilization
- **Pitfall:** No universal threshold. Compare a ratio against the baseline for the same path. A rising ratio points to loss, congestion, or a faulty path.

### Connections Established (TCPv4, TCPv6)

- **Unit:** count (gauge).
- **Source:** `TCPv4\Connections Established`.
- **Reference:** https://learn.microsoft.com/host-integration-server/core/useful-performance-counters2 (use as user-load proxy).
- **Pitfall:** Counts currently established connections. TIME_WAIT and CLOSE_WAIT are not included. Use `Get-NetTCPConnection` for state breakdown.

## UDP

### Datagrams Received Errors (UDP)

- **Unit:** datagrams (count).
- **Formula:** Δerrors / Δt.
- **Source:** `UDPv4\Datagrams Received Errors`.
- **Reference:** https://learn.microsoft.com/windows-server/networking/technologies/network-subsystem/net-sub-performance-counters
- **Pitfall:** Errors often mean no listening socket on the port. Confirm with the application before treating it as a network fault.

## Receive Side Coalescing (RSC)

### RSC counters

- **Unit:** count or rate (per counter).
- **Source:** Counters listed under "Receive Side Coalescing (RSC) performance" in the Learn source.
- **Reference:** https://learn.microsoft.com/windows-server/networking/technologies/network-subsystem/net-sub-performance-counters (section 3).
- **Pitfall:** RSC applies only to supported NICs and VMs. Absence of RSC counters is normal on unsupported hardware.

## Per-process network

### Process network bytes (Windows Performance Counters)

- **Unit:** bytes.
- **Source:** `Process(<name>)\IO Data Bytes/sec`, `\IO Other Bytes/sec` (I/O, not network-only); `.NET` socket counters listed in https://learn.microsoft.com/dotnet/framework/debug-trace-profile/performance-counters#networking-performance-counters.
- **Reference:** https://learn.microsoft.com/biztalk/technical-guides/checklist-measuring-performance-on-hyper-v (process-level network counters are not isolated).
- **Pitfall:** `Process\IO Data Bytes/sec` includes disk and pipe I/O as well as sockets. Windows does not provide a per-process network counter in the base performance set. Use ETW (`Microsoft-Windows-TCPIP`) or Resource Monitor for per-process network use.

## Windows Internals references

- **TCP/IP stack:** tcpip.sys, NDIS miniport, and the Winsock layer. Reference: *Windows Internals* (networking chapter).
- **Sysinternals TCPView:** per-connection state and process owner. https://learn.microsoft.com/en-us/sysinternals/downloads/tcpview
