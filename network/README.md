# Network metrics

Interface throughput, IP, TCP, UDP, output queue, errors, and RSC for Windows.

Status: source-checked against Microsoft Learn on 2026-10-29. Not yet verified on a live machine. Status values are defined in the [root README](../README.md#verification-status).

Source for grouping: [Network-Related Performance Counters](https://learn.microsoft.com/windows-server/networking/technologies/network-subsystem/net-sub-performance-counters) (sections: Resource Utilization, Potential Network Problems, Receive Side Coalescing).

## Interface throughput

### Bytes Total/sec (Network Interface)

- **Unit:** bytes per second.
- **Formula:** Δ(bytes received + bytes sent) / Δt. Utilization = (Bytes Total/sec × 8) / link speed in bits per second × 100.
- **Source:** `Network Interface(<adapter>)\Bytes Total/sec`; `Bytes Received/sec`, `Bytes Sent/sec`.
- **Reference:** [Scenario guide: Troubleshoot performance problems in Windows](https://learn.microsoft.com/troubleshoot/windows-server/performance/troubleshoot-performance-problems-in-windows#network) (healthy: under 50% of link speed; warning: 50–80%; critical: over 80%).
- **Status:** source-checked.
- **Pitfall:** Network speed is in bits and counters are in bytes. A 1 Gb/s adapter can carry about 125 MB/s. The guide's own example uses this conversion.

### Current Bandwidth (Network Interface, WMI)

- **Unit:** bits per second.
- **Source:** WMI `Win32_NetworkAdapter` (`Speed`); `Network Interface(<adapter>)\Current Bandwidth`.
- **Reference:** [Win32_NetworkAdapter class](https://learn.microsoft.com/windows/win32/cimwin32prov/win32-networkadapter). Counter name was not checked on this pass.
- **Status:** partly checked.
- **Pitfall:** Speed is the negotiated link speed. A link that negotiates a lower speed or half duplex reduces throughput, and utilization percentages will change with it.

### Packets Received/sec, Packets Sent/sec (Network Interface)

- **Unit:** packets per second.
- **Source:** `Network Interface(<adapter>)\Packets Received/sec`, `\Packets Sent/sec`.
- **Reference:** [Network-Related Performance Counters](https://learn.microsoft.com/windows-server/networking/technologies/network-subsystem/net-sub-performance-counters) (listed under Resource Utilization). No threshold.
- **Status:** source-checked (no threshold).
- **Pitfall:** Small-packet workloads can load the CPU before the link fills. Check Processor DPC time (see [cpu/](../cpu/README.md)) alongside.

## Queueing

### Output Queue Length (Network Interface)

- **Unit:** packets (average over the interval).
- **Formula:** Length of the output packet queue.
- **Source:** `Network Interface(<adapter>)\Output Queue Length`.
- **Reference:** [Network-Related Performance Counters](https://learn.microsoft.com/windows-server/networking/technologies/network-subsystem/net-sub-performance-counters) (if the length is longer than 2, delays occur; because NDIS queues requests, the value should always be 0).
- **Status:** source-checked.
- **Pitfall:** A nonzero value is a bottleneck signal, not a normal state.

## Errors and discards

### Packets Received Errors, Packets Outbound Errors (Network Interface)

- **Unit:** packets (count).
- **Formula:** Δerrors / Δt.
- **Source:** `Network Interface(<adapter>)\Packets Received Errors`, `\Packets Outbound Errors`.
- **Reference:** [Network-Related Performance Counters](https://learn.microsoft.com/windows-server/networking/technologies/network-subsystem/net-sub-performance-counters) (listed under Potential Network Problems).
- **Status:** source-checked.
- **Pitfall:** Any sustained nonzero rate needs investigation at the driver or cabling layer.

### Packets Received Discarded, Packets Outbound Discarded (Network Interface)

- **Unit:** packets (count).
- **Source:** `Network Interface(<adapter>)\Packets Received Discarded`, `\Packets Outbound Discarded`.
- **Reference:** [Network-Related Performance Counters](https://learn.microsoft.com/windows-server/networking/technologies/network-subsystem/net-sub-performance-counters) (listed under Potential Network Problems).
- **Status:** source-checked.
- **Pitfall:** Compare discards with Output Queue Length. Discards that rise with a growing queue point to a saturated path.

## IP

### Datagrams Received/sec, Datagrams Sent/sec (IPv4, IPv6)

- **Unit:** datagrams per second.
- **Source:** `IPv4\Datagrams Received/sec`, `IPv4\Datagrams Sent/sec`; same for `IPv6`.
- **Reference:** [Network-Related Performance Counters](https://learn.microsoft.com/windows-server/networking/technologies/network-subsystem/net-sub-performance-counters) (listed under Resource Utilization). No threshold.
- **Status:** source-checked (no threshold).
- **Pitfall:** These are stack-wide totals. Use them for trend, not per-application attribution.

## TCP

### Segments Received/sec, Segments Sent/sec (TCPv4, TCPv6)

- **Unit:** segments per second.
- **Source:** `TCPv4\Segments Received/sec`, `TCPv4\Segments Sent/sec`.
- **Reference:** [Network-Related Performance Counters](https://learn.microsoft.com/windows-server/networking/technologies/network-subsystem/net-sub-performance-counters) (listed under Resource Utilization).
- **Status:** source-checked.
- **Pitfall:** Segment size varies with MSS and offload. Interpret rates together with Bytes Total/sec.

### Segments Retransmitted/sec (TCPv4, TCPv6)

- **Unit:** segments per second.
- **Formula:** Δretransmitted segments / Δt. Retransmission ratio = Segments Retransmitted/sec / Segments Sent/sec.
- **Source:** `TCPv4\Segments Retransmitted/sec`.
- **Reference:** [Network-Related Performance Counters](https://learn.microsoft.com/windows-server/networking/technologies/network-subsystem/net-sub-performance-counters) (listed under Resource Utilization). No threshold.
- **Status:** source-checked (no threshold).
- **Pitfall:** No universal threshold. Compare the ratio against the baseline for the same path.

### Connection Failures, Connections Reset (TCPv4, TCPv6)

- **Unit:** count (rate per counter).
- **Source:** `TCPv4\Connection Failures`, `TCPv4\Connections Reset`.
- **Reference:** [Network-Related Performance Counters](https://learn.microsoft.com/windows-server/networking/technologies/network-subsystem/net-sub-performance-counters) (listed under Potential Network Problems).
- **Status:** source-checked.
- **Pitfall:** Resets can be normal application behavior. Confirm with the application before treating them as network faults.

## UDP

### Datagrams Received Errors (UDP)

- **Unit:** datagrams (count).
- **Formula:** Δerrors / Δt.
- **Source:** `UDPv4\Datagrams Received Errors`.
- **Reference:** [Network-Related Performance Counters](https://learn.microsoft.com/windows-server/networking/technologies/network-subsystem/net-sub-performance-counters) (listed under Potential Network Problems).
- **Status:** source-checked.
- **Pitfall:** Errors often mean no listening socket on the port. Confirm with the application before treating them as a network fault.

## Receive Side Coalescing (RSC)

### RSC counters

- **Unit:** count or rate (per counter).
- **Source:** `Network Adapter(<adapter>)\TCP Active RSC Connections`, `\TCP RSC Average Packet Size`, `\TCP RSC Coalesced Packets/sec`, `\TCP RSC Exceptions/sec`.
- **Reference:** [Network-Related Performance Counters](https://learn.microsoft.com/windows-server/networking/technologies/network-subsystem/net-sub-performance-counters) (Receive Side Coalescing section).
- **Status:** source-checked.
- **Pitfall:** RSC applies only to supported NICs and configurations. Missing RSC counters are normal on unsupported hardware.

## Per-process network

### Process I/O bytes (Process)

- **Unit:** bytes.
- **Source:** `Process(<name>)\IO Data Bytes/sec`, `\IO Other Bytes/sec`.
- **Reference:** [Network-Related Performance Counters](https://learn.microsoft.com/windows-server/networking/technologies/network-subsystem/net-sub-performance-counters) does not list a per-process network counter. The .NET socket counters are in [.NET performance counters: networking](https://learn.microsoft.com/dotnet/framework/debug-trace-profile/performance-counters#networking-performance-counters) (content not checked on this pass).
- **Status:** partly checked.
- **Pitfall:** `Process\IO Data Bytes/sec` counts all I/O (disk, pipes, and sockets), not network traffic alone. For per-process network use, use Resource Monitor or an ETW trace. The ETW provider name was not checked against Microsoft Learn.

## Windows Internals and tool references

- **TCP/IP stack:** tcpip.sys, NDIS miniport, and the Winsock layer. Reference: *Windows Internals* (networking chapter). Not a public API.
- **Sysinternals TCPView:** per-connection state and process owner. [Sysinternals TCPView](https://learn.microsoft.com/en-us/sysinternals/downloads/tcpview).
