# Network Simulation: TCP and OSPF Experiments in CORE

Course project for Computer Networks (ELC 3080, Spring 2025), Faculty of Engineering, Cairo University. Done with Yahia Ebrahim.

We built and ran experiments on a virtual network in the CORE network emulator (Linux), using iperf3 for traffic, Wireshark for packet captures and Quagga's `vtysh` to configure the OSPF routers. The topology comes from the course file `Project_OSPF_TCP.imn`. The experiments look at TCP behaviour on one side and OSPF behaviour on the other.

## Files

- [`network_simulation.pdf`](network_simulation.pdf): our report, with screenshots, plots and answers
- [`CUNetProject_2025.pdf`](CUNetProject_2025.pdf): the project handout

## Experiments

### 1. TCP window size

iperf3 from n7 to n11 for 40 s, with the window size going from 1 KB up to 32 KB.

- Throughput peaks at about 6.8 Mbps with 3 to 4 KB windows.
- At 6 KB it collapses to roughly 0.12 Mbps, and retransmissions start appearing.
- Retransmissions peak at 12 KB (about 2,400), where throughput recovers a little to 2.8 Mbps. After that it stays low (0.2 to 0.4 Mbps) for 16, 24 and 32 KB.
- We also dissected a data segment and an ACK in Wireshark: 32 bytes of TCP header (20 + 12 bytes of options: two NOPs and timestamps), 20 bytes of IP, and 16 bytes of link-layer header.

### 2. Short vs. long path

Same 4 KB window, n7 to n11 compared with n7 to n8: 7.91 Mbps vs. 5.46 Mbps. Link capacities are the same, but the longer path has a higher RTT, and with a small fixed window that limits throughput.

### 3. Link capacity vs. loss (n4 to n5 link, 4 KB window)

| Case | Capacity | Loss n4→n5 | Loss n5→n4 | Throughput |
|---|---|---|---|---|
| a | 10 Mbps | 0 | 0 | 7.9 Mbps |
| b | 3 Mbps | 0 | 0 | 2.82 Mbps |
| c | 10 Mbps | 5% | 5% | 648 Kbps |
| d | 100 Mbps | 10% | 10% | 192 Kbps |
| e | 10 Mbps | 1% | 0 | 5.82 Mbps |
| f | 10 Mbps | 0 | 1% | 3.05 Mbps |

A slower but clean link (b) beats a faster lossy one (c, d), because TCP cuts its sending rate on every loss.

### 4. OSPF link cost changes

- Raising the cost of the n5–n4 interface (eth1 on n5) to 40 moved the n7→n11 path from n5→n4→n9 to a detour via n6, since the detour had a lower total cost (50 vs. 70).
- With two connections running (n7→n11 and n11→n7) and the cost raised on n4's side instead, only the n11→n7 path changed. OSPF costs are per direction, so each side routes using its own outgoing interface costs.

### 5. OSPF database updates

- Looked at the link-state database and routing table on n2 (router and network LSAs, equal-cost routes).
- After setting a link cost on n4, the LSA exchange seen in Wireshark finished in about 0.8 s.
- After bringing n4's interfaces down, its router LSA disappeared from n2's database and the routes that went through n4 were recalculated.

## Tools

CORE network emulator, Linux, iperf3, Wireshark 1.6.7, Quagga (`vtysh`).
