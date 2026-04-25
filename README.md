# PRISM — Performance Real-time iPerf3 Stream Manager

> Enterprise-grade multi-stream traffic orchestration with a live QoS dashboard.

[![Version](https://img.shields.io/badge/version-8.3.8-blue.svg)](https://github.com/waqasdaar/prism)
[![Shell](https://img.shields.io/badge/shell-Bash%203.2%2B-green.svg)](https://www.gnu.org/software/bash/)
[![Platform](https://img.shields.io/badge/platform-Linux%20%7C%20macOS-lightgrey.svg)](https://github.com/waqasdaar/prism)
[![License](https://img.shields.io/badge/license-MIT-orange.svg)](LICENSE)

---

## Table of Contents

1. [Introduction](#introduction)
2. [How It Works](#how-it-works)
3. [Requirements](#requirements)
4. [Installation](#installation)
5. [Quick Start](#quick-start)
6. [Application Menu](#application-menu)
7. [Implemented Use Cases](#implemented-use-cases)
8. [Dashboard Reference](#dashboard-reference)
9. [DSCP Reference Table](#dscp-reference-table)
10. [Output and Log Files](#output-and-log-files)
11. [Cleanup and Signal Handling](#cleanup-and-signal-handling)
12. [Troubleshooting](#troubleshooting)
13. [Known Limitations](#known-limitations)
14. [Contributing](#contributing)
15. [Author](#author)
16. [License](#license)

---

## Introduction

**PRISM** (Performance Real-time iPerf3 Stream Manager) is a
production-grade Bash wrapper for
[iperf3](https://github.com/esnet/iperf) that transforms
single-stream benchmarking into a comprehensive network testing
platform.

Rather than running one iperf3 process manually, PRISM orchestrates
multiple simultaneous streams across different protocols, QoS classes,
VRFs, and network paths — all while rendering a live terminal dashboard
that shows bandwidth, RTT, TCP congestion window, traffic ramp curves,
and stream health in real time.

### What makes PRISM different

| Capability | Vanilla iperf3 | PRISM |
|---|---|---|
| Simultaneous multi-stream | Manual scripting | Built-in, up to 64 streams |
| Live dashboard | None | Real-time TUI with sparklines |
| DSCP / QoS marking | Single stream | Per-stream with live verification |
| Mixed traffic profiles | None | Percentage-based traffic mix generator |
| TCP CWND tracking | Verbose log only | Live dashboard row + results table |
| Traffic ramp-up | None | Kernel-level `tc tbf` shaping |
| RTT measurement | None | Parallel `ping` per stream |
| VRF-aware routing | None | Full `ip vrf exec` integration |
| Path MTU discovery | None | Binary-search ICMP probe |
| Pre-flight checks | None | Ping + TCP port + traceroute |
| JSON results export | None | Structured per-second time-series |
| Session history | None | JSON Lines audit log |
| FQDN targets | None | 6-method DNS resolution chain |
| Bidirectional test | Manual two processes | Native `--bidir` (iperf3 ≥ 3.7) |
| Network impairment | None | `tc netem` delay / jitter / loss |
| DSCP verification | None | Live tcpdump packet capture overlay |
| Theme support | None | Dark / Light / Mono terminal themes |

---

## How It Works



```
┌──────────────────────────────────────────────────────────────────┐
│                       PRISM Architecture                         │
│                                                                  │
│  ┌──────────────┐    ┌──────────────────────────────────────┐    │
│  │  Operator    │───▶│  Configuration Wizard                │    │
│  │  (terminal)  │    │  Protocol · Target · DSCP · VRF ...  │    │
│  └──────────────┘    └──────────────────────────────────────┘    │
│                                      │                           │
│                          ┌───────────▼──────────┐                │
│                          │   Pre-flight Engine  │                │
│                          │  Ping · TCP · MTU    │                │
│                          │  Traceroute          │                │
│                          └───────────┬──────────┘                │
│                                      │                           │
│          ┌───────────────────────────▼────────────────────┐      │
│          │                Launch Engine                   │      │
│          │  iperf3 clients · ping · bidir · ramp (tc)     │      │
│          └──────┬─────────┬──────────┬────────────────────┘      │
│                 │         │          │                           │
│          ┌──────▼──┐  ┌───▼────┐  ┌──▼──────────┐                │
│          │ iperf3  │  │  ping  │  │  tc tbf     │                │
│          │ streams │  │ (RTT)  │  │  (ramp)     │                │
│          └──────┬──┘  └───┬────┘  └─────────────┘                │
│                 │         │                                      │
│          ┌──────▼─────────▼──────────────────────────────┐       │
│          │              Live TUI Dashboard               │       │
│          │  BW · RTT · CWND · Ramp · Progress · DSCP     │       │
│          └──────────────────────┬────────────────────────┘       │
│                                 │                                │
│          ┌──────────────────────▼────────────────────────┐       │
│          │              Results & Export                 │       │
│          │  Table · JSON · Session History · Logs        │       │
│          └───────────────────────────────────────────────┘       │
└──────────────────────────────────────────────────────────────────┘
```
