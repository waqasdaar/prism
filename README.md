# iperf3 Multi-Stream Traffic Manager 

## What Is This?

The iperf3 Multi-Stream Traffic Manager is an interactive Bash script that wraps the standard iperf3 network testing tool with enterprise-grade capabilities. It transforms iperf3 from a simple point-to-point bandwidth tester into a comprehensive network validation platform.

If you've ever needed to:

- Test bandwidth across multiple paths simultaneously
- Validate QoS policies by generating traffic with specific DSCP markings
- Run iperf3 inside Linux VRFs without hand-crafting ip vrf exec commands
- Compare how different TCP congestion algorithms perform on your links
- Simulate real-world network impairments (delay, jitter, loss) during testing
- Actually see what your test packets look like at L2, L3, and L4

…then this script was built for you.

## Who Is This For?

| **Audience**                     | **Why They'd Use It**                                               |
|------------------------------|-----------------------------------------------------------------|
| Network Engineers            | QoS validation, path testing, VRF-aware traffic generation      |
| Systems Engineers            | Bandwidth baselining, congestion control tuning                 |
| Lab Engineers                | Multi-stream test scenarios without manual command construction |
| Pre-Sales / Proof of Concept | Demonstrable QoS differentiation and traffic visualization      |
| Students / Learners          | Understanding DSCP, TCP/UDP headers, congestion algorithms      |

## Prerequisites

| **Requirement**                                | **Purpose**                                          |
|------------------------------------------------|------------------------------------------------------|
| Linux (Ubuntu 20.04+ / Debian 11+ recommended) | Primary OS                                           |
| iperf3 (3.x)                                   | Core traffic generator                               |
| iproute2                                       | VRF detection, interface enumeration                 |
| tc / netem (optional)                          | Congestion simulation                                |
| Root / sudo                                    | VRF sysctl tuning, netem rules, binding to low ports |

### Use Case 1: Basic Bandwidth Testing (Single Stream)

#### Scenario

You have a new 10 Gbps link between two data center switches. Before putting it into production, you want to validate that the link actually delivers expected throughput end-to-end.


##### How to Run

###### On the server side:
```
$ sudo ./iperf3-traffic-flows.sh
```
**Select Option 1**: Start iperf3 Server, choose the interface and port (e.g., ens192 on port 5201).

**On the client side**:
```
$ sudo ./iperf3-traffic-flows.sh
```
__Sample Output (Client Selection)__

```
╔══════════════════════════════════════════════════════════════════════╗
║               iperf3 Multi-Stream Traffic Manager v6.2               ║
╠══════════════════════════════════════════════════════════════════════╣
║  0) Show DSCP Reference Table     5) Congestion Simulation (tc)      ║
║  1) Start iperf3 Server           6) Compare Congestion Algorithms   ║
║  2) Start iperf3 Client           7) Quick Loopback Test             ║
║  3) Multi-Stream Client           8) Manage Logs                     ║
║  4) Bandwidth Monitor             9) Exit                            ║
╚══════════════════════════════════════════════════════════════════════╝

Enter choice [0-9]: 2
```

```
=== Client Configuration ===
Enter server IP: 10.10.10.2
Enter server port [5201]: 5201
Select protocol:
  1) TCP
  2) UDP
Enter choice [1-2]: 1
Enter test duration in seconds [10]: 30
Enter target bandwidth (e.g., 1G, 500M) [0 = unlimited]: 0
Enter DSCP marking [default]: default

=== Interface Selection ===
Available interfaces:
  1) ens192    10.10.10.1    (global)
  2) ens224    10.20.20.1    (vrf: vrf10)
  3) ens256    10.30.30.1    (vrf: vrf20)
  4) lo        127.0.0.1     (global)
Enter interface number: 1
Binding to 10.10.10.1 on ens192
```
What You'll See

```
=== Packet Detail: Stream 1 ===
┌──────────────────────────────────────────────────┐
│ Layer 2 - Ethernet                               │
│   Src MAC : (interface ens192 MAC)               │
│   Dst MAC : (next-hop ARP resolution)            │
│   EtherType: 0x0800 (IPv4)                       │
├──────────────────────────────────────────────────┤
│ Layer 3 - IPv4                                   │
│   Src IP  : 10.10.10.1                           │
│   Dst IP  : 10.10.10.2                           │
│   Protocol: 6 (TCP)                              │
│   DSCP    : default (0) → TOS: 0x00             │
├──────────────────────────────────────────────────┤
│ Layer 4 - TCP                                    │
│   Src Port: (ephemeral)                          │
│   Dst Port: 5201                                 │
│   Congestion Control: cubic                      │
└──────────────────────────────────────────────────┘

Launch this stream? [Y/n]: y
[Stream 1] iperf3 client started (PID 12345) → 10.10.10.2:5201
```
### Use Case 2: QoS Validation with Per-Stream DSCP Marking

#### Scenario

Your network has QoS policies configured. You need to verify that traffic marked with different DSCP values is actually treated differently — e.g., EF (Expedited Forwarding) gets priority over BE (Best Effort). You want to generate multiple streams, each with a distinct DSCP marking, and observe how the network handles them.

##### How to Run
Select Option 3: __Multi-Stream Client__.
Sample Configuration Flow
```
=== Multi-Stream Client Configuration ===
Enter number of parallel streams [2]: 3

--- Stream 1 of 3 ---
Enter server IP: 10.10.10.2
Enter server port [5201]: 5201
Select protocol: 1) TCP  2) UDP : 1
Enter duration [10]: 60
Enter bandwidth [0 = unlimited]: 10M
Enter DSCP [default]: ef

--- Stream 2 of 3 ---
Enter server IP: 10.10.10.2
Enter server port [5202]: 5202
Select protocol: 1) TCP  2) UDP : 1
Enter duration [10]: 60
Enter bandwidth [0 = unlimited]: 10M
Enter DSCP [default]: af21

--- Stream 3 of 3 ---
Enter server IP: 10.10.10.2
Enter server port [5203]: 5203
Select protocol: 1) TCP  2) UDP : 1
Enter duration [10]: 60
Enter bandwidth [0 = unlimited]: 10M
Enter DSCP [default]: default
```
Packet Visualization (Per-Stream)
For each stream, you see the full packet structure before launch:

```
=== Packet Detail: Stream 1 ===
┌──────────────────────────────────────────────────┐
│ Layer 3 - IPv4                                   │
│   DSCP    : ef (46) → TOS: 0xB8                  │
├──────────────────────────────────────────────────┤
│ Layer 4 - TCP                                    │
│   Dst Port: 5201                                 │
└──────────────────────────────────────────────────┘

=== Packet Detail: Stream 2 ===
┌──────────────────────────────────────────────────┐
│ Layer 3 - IPv4                                   │
│   DSCP    : af21 (18) → TOS: 0x48                │
├──────────────────────────────────────────────────┤
│ Layer 4 - TCP                                    │
│   Dst Port: 5202                                 │
└──────────────────────────────────────────────────┘

=== Packet Detail: Stream 3 ===
┌──────────────────────────────────────────────────┐
│ Layer 3 - IPv4                                   │
│   DSCP    : default (0) → TOS: 0x00              │
├──────────────────────────────────────────────────┤
│ Layer 4 - TCP                                    │
│   Dst Port: 5203                                 │
└──────────────────────────────────────────────────┘
```
Validation with Capture
While streams run, use tcpdump on either end to verify TOS byte:

```
sudo tcpdump -i ens192 -v -n 'dst port 5201' | grep 'tos 0xb8'
sudo tcpdump -i ens192 -v -n 'dst port 5202' | grep 'tos 0x48'
sudo tcpdump -i ens192 -v -n 'dst port 5203' | grep 'tos 0x0'
```
###### Why This Matters

Without this tool, you'd need to manually craft three separate iperf3 commands:

```
iperf3 -c 10.10.10.2 -p 5201 -t 60 -b 10M -S 184 &
iperf3 -c 10.10.10.2 -p 5202 -t 60 -b 10M -S 72 &
iperf3 -c 10.10.10.2 -p 5203 -t 60 -b 10M -S 0 &
```
And you'd need to remember that DSCP EF = 46, TOS = 184, the -S flag takes TOS not DSCP, etc. The script handles all of this mapping automatically from the 22-entry DSCP reference table.

