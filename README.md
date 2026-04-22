# C-Shark — The Command-Line Packet Predator

> A lightweight, terminal-based network packet sniffer written in C, powered by `libpcap`. Capture, filter, and inspect live network traffic — layer by layer — right from your terminal.

---

## Table of Contents

- [Overview](#-overview)
- [Features](#-features)
- [Architecture](#-architecture)
- [Requirements](#-requirements)
- [Installation](#-installation)
- [Usage](#-usage)
- [BPF Filters](#-bpf-filters)
- [Session Inspection](#-session-inspection)
- [Project Structure](#-project-structure)

---

## Overview

**C-Shark** is a command-line packet analyzer inspired by Wireshark, built entirely in C using the `libpcap` library. It captures live packets from any available network interface, decodes them across all OSI layers (Ethernet → IP/IPv6 → TCP/UDP → Application), and stores them in memory for post-capture inspection.

Designed for networking enthusiasts, students, and developers who want deep visibility into network traffic without leaving the terminal.

---

## Features

- **Interactive CLI** — Menu-driven interface for interface selection and capture control
-  **Live Packet Capture** — Capture all packets or apply protocol filters in real time
-  **Multi-Layer Parsing** — Full OSI layer analysis:
  - **Layer 2** — Ethernet (MAC addresses, EtherType)
  - **Layer 3** — IPv4, IPv6, ARP
  - **Layer 4** — TCP (flags, ports), UDP
  - **Layer 7** — Application-layer payload inspection
-  **In-Memory Storage** — Stores up to **10,000 packets** per session
-  **Post-Capture Inspection** — Browse and deep-dive into captured packets with hex dumps
-  **BPF Filtering** — Filter by HTTP, HTTPS, DNS, ARP, TCP, or UDP
-  **Graceful Exit** — Stop capture with `Ctrl+C`; exit the program with `Ctrl+D`

---

##  Architecture

C-Shark is organized into focused, single-responsibility modules:

```
┌─────────────┐
│    main.c   │  ← Entry point, main loop, interface selection
└──────┬──────┘
       │
  ┌────┴────┐
  │  Capture │  capture.c — libpcap loop, SIGINT/Ctrl+D handling, packet dispatch
  └────┬────┘
       │
  ┌────▼──────────────────────────────────────┐
  │              Packet Parsers               │
  │  ethernet.c  →  network.c  →  transport.c →  application.c  │
  └───────────────────────────────────────────┘
       │
  ┌────▼────┐         ┌───────────┐
  │ storage │◄────────│ inspector │  storage.c / inspector.c
  └─────────┘         └───────────┘
       │
  ┌────▼────┐
  │  filter │  filter.c — BPF filter menu & string builder
  └─────────┘
```

---

## Requirements

| Dependency | Details |
|---|---|
| **OS** | Linux (tested on Ubuntu/Debian) |
| **Compiler** | `gcc` with C99 or later |
| **Library** | `libpcap` (≥ 1.8) |
| **Privileges** | `root` or `CAP_NET_RAW` capability for live capture |

Install `libpcap` on Debian/Ubuntu:

```bash
sudo apt-get install libpcap-dev
```

---

## Installation

```bash
# Clone the repository
git clone https://github.com/your-username/C-Shark.git
cd C-Shark

# Build the project
make

# Run with root privileges (required for raw packet capture)
sudo ./cshark
```

To clean build artifacts:

```bash
make clean
```

---

## Usage

```
[C-Shark] The Command-Line Packet Predator
==============================================

Available Interfaces:
  1. eth0
  2. lo
  3. wlan0

> 1

[C-Shark] Interface 'eth0' selected. What's next?

1. Start Sniffing (All Packets)
2. Start Sniffing (With Filters)
3. Inspect Last Session
4. Exit C-Shark

Select an option (1-4):
```

### Controls

| Key | Action |
|---|---|
| `Ctrl+C` | Stop the current capture session |
| `Ctrl+D` | Exit C-Shark entirely |

---

## BPF Filters

When selecting **Option 2 (Start Sniffing With Filters)**, C-Shark presents a filter menu backed by BPF (Berkeley Packet Filter) expressions:

| # | Filter | BPF Expression |
|---|---|---|
| 1 | HTTP | `tcp port 80` |
| 2 | HTTPS | `tcp port 443` |
| 3 | DNS | `port 53` |
| 4 | ARP | `arp` |
| 5 | TCP (all) | `tcp` |
| 6 | UDP (all) | `udp` |
| 7 | Cancel | — |

---

## Session Inspection

After stopping a capture, select **Option 3 (Inspect Last Session)** to:

- View a **summary table** of all captured packets (ID, timestamp, size, protocol)
- Select any individual packet for **detailed analysis**, including:
  - Parsed Ethernet, IP, TCP/UDP headers
  - Source & destination addresses and ports
  - Application-layer payload
  - Full **hex dump** of raw packet bytes

Up to **10,000 packets** are retained in memory per session. The session is cleared automatically when a new capture begins.

---

## Project Structure

```
C-Shark/
├── main.c            # Entry point — interface selection & main menu
├── capture.c/h       # Live packet capture loop (libpcap integration)
├── filter.c/h        # BPF filter menu & string builder
├── storage.c/h       # In-memory packet store (up to 10,000 packets)
├── inspector.c/h     # Per-packet detailed analysis & hex dump
├── interface.c/h     # Network interface enumeration via libpcap
├── ethernet.c/h      # Layer 2 — Ethernet header parser
├── network.c/h       # Layer 3 — IPv4, IPv6, ARP parsers
├── transport.c/h     # Layer 4 — TCP & UDP parsers, port name lookup
├── application.c/h   # Layer 7 — Application payload inspector
├── util.c/h          # Utility helpers (input reading, etc.)
└── Makefile          # Build configuration
```
