# iperf3 Multi-Stream Traffic Tool

<div align="center">

![Version](https://img.shields.io/badge/version-7.7-blue?style=flat-square)
![Shell](https://img.shields.io/badge/shell-bash%204.0%2B-green?style=flat-square)
![Platform](https://img.shields.io/badge/platform-Linux-orange?style=flat-square)
![License](https://img.shields.io/badge/license-MIT-yellow?style=flat-square)
![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen?style=flat-square)

*An enterprise-grade interactive Bash wrapper for `iperf3` featuring
multi-stream orchestration, real-time live dashboards, VRF awareness,
DSCP/QoS marking, TCP tuning, and network impairment injection.*

</div>

---

## Table of Contents

- [Introduction](#introduction)
- [How It Works](#how-it-works)
- [Requirements](#requirements)
- [Installation](#installation)
- [Quick Start](#quick-start)
- [Application Menu](#application-menu)
- [Use Cases](#use-cases)
  - [Use Case 1 — Basic TCP Throughput Test](#use-case-1--basic-tcp-throughput-test)
  - [Use Case 2 — UDP Performance and Loss Test](#use-case-2--udp-performance-and-loss-test)
  - [Use Case 3 — Multi-Stream Parallel Testing](#use-case-3--multi-stream-parallel-testing)
  - [Use Case 4 — Server Mode with Multiple Listeners](#use-case-4--server-mode-with-multiple-listeners)
  - [Use Case 5 — VRF-Aware Testing](#use-case-5--vrf-aware-testing)
  - [Use Case 6 — DSCP and QoS Marking](#use-case-6--dscp-and-qos-marking)
  - [Use Case 7 — TCP Tuning and CCA Comparison](#use-case-7--tcp-tuning-and-cca-comparison)
  - [Use Case 8 — Network Impairment Injection](#use-case-8--network-impairment-injection)
  - [Use Case 9 — Reverse Mode Asymmetric Testing](#use-case-9--reverse-mode-asymmetric-testing)
  - [Use Case 10 — Loopback Local Validation](#use-case-10--loopback-local-validation)
- [Dashboard Reference](#dashboard-reference)
- [DSCP Reference Table](#dscp-reference-table)
- [Output and Log Files](#output-and-log-files)
- [Cleanup and Signal Handling](#cleanup-and-signal-handling)
- [Troubleshooting](#troubleshooting)
- [Known Limitations](#known-limitations)
- [License](#license)

---

## Introduction

`iperf3_manager.sh` is a production-grade terminal application that wraps
`iperf3` in a fully interactive menu-driven interface. It was built to
solve the real operational challenges that network engineers face when using
raw `iperf3` commands in enterprise and carrier environments.

### The Problem with Raw `iperf3`

Using `iperf3` directly works well for a single stream, but breaks down
quickly in production scenarios:

| Challenge | Why It Hurts |
|---|---|
| Multiple simultaneous streams | Requires multiple terminals, manual PID tracking |
| VRF-specific testing | Must remember exact `ip vrf exec` syntax and VRF names |
| DSCP marking | Must manually convert names like `EF` or `AF41` to TOS values |
| Live monitoring | Output scrolls away and is hard to follow across streams |
| TCP tuning | Must remember flags for CCA, window size, MSS across streams |
| Cleanup after Ctrl+C | Orphaned processes and `tc` rules left behind |
| UDP bandwidth target | Easy to forget and run at default 1 Mbps |
| Post-test analysis | No consolidated summary across multiple streams |

### What This Tool Provides

`iperf3_manager.sh` replaces all of that complexity with a single guided
workflow that any operator can use without memorising `iperf3` flags.

### Who Should Use This Tool

- Network engineers validating WAN or LAN throughput
- Operations teams running before/after change benchmarks
- Lab engineers testing QoS, DSCP, and traffic classification
- Anyone testing within Linux VRF environments
- Teams needing reproducible multi-stream test results

---

## How It Works

The script builds a complete orchestration layer around `iperf3`:

```
┌─────────────────────────────────────────────────────────────────────┐
│ iperf3-traffic-manager.sh                                           │
│                                                                     │
│ 1. Detect system capabilities and iperf3 version                    │
│ 2. Discover interfaces, IPs, speeds, and VRF memberships            │
│ 3. Guide operator through stream/listener configuration             │
│ 4. Build per-stream launch scripts with correct flags               │
│ 5. Launch iperf3 processes in background                            │
│ 6. Track PIDs and log file paths per stream                         │
│ 7. Parse plain-text iperf3 output every second                      │
│ 8. Render overwrite dashboard without terminal scrolling            │
│ 9. Show consolidated final results on completion                    │
│ 10. Clean up all processes and tc rules on exit                     │
└─────────────────────────────────────────────────────────────────────┘
```
