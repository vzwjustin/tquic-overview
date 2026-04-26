# TQUIC Kernel Module

> Multipath QUIC for in-kernel WAN bonding across heterogeneous cellular links.

![Status](https://img.shields.io/badge/status-active%20development-orange)
![Language](https://img.shields.io/badge/language-C-blue)
![Layer](https://img.shields.io/badge/layer-Linux%20kernel-black)

---

## Overview

TQUIC is a Linux kernel module that implements multipath QUIC at the kernel layer. The goal is to bond two or more WAN links — typically heterogeneous cellular paths (LTE, 5G, mixed carriers) — into a single resilient transport with both failover and aggregate-throughput modes.

Most production multipath solutions operate in userspace (mptcp helpers, userspace bonding daemons, SDN overlays), which adds per-packet latency and operational complexity. TQUIC pushes path selection and reordering logic into the kernel, where the cost of each decision is minimal.

---

## Why kernel-layer multipath QUIC?

Cellular links — even on the same carrier — exhibit highly variable latency, jitter, and packet loss. A naive WAN bond that round-robins packets across two LTE modems often performs worse than either link in isolation. The interesting work sits in:

- **Path selection** under non-stationary RTT and loss
- **Reordering tolerance** without inducing head-of-line blocking
- **Failover behavior** that does not tear down active flows
- **Cost-aware routing** when one link is metered (e.g., carrier data caps)

QUIC's connection-ID-based design is a natural fit for multipath. Doing it in kernel keeps the per-packet overhead low enough to actually improve real-world throughput rather than degrade it.

---

## Architecture (high level)

```
   ┌──────────────────┐                    ┌──────────────────┐
   │   Application    │                    │   Application    │
   └────────┬─────────┘                    └────────┬─────────┘
            │                                       │
   ┌────────▼─────────┐                    ┌────────▼─────────┐
   │  TQUIC kernel    │◄──────────────────►│  TQUIC kernel    │
   │     module       │  multipath QUIC    │     module       │
   └──┬────────────┬──┘  streams + sched   └──┬────────────┬──┘
      │            │                          │            │
   ┌──▼──┐      ┌──▼──┐                    ┌──▼──┐      ┌──▼──┐
   │ LTE │      │  5G │                    │ LTE │      │  5G │
   └─────┘      └─────┘                    └─────┘      └─────┘
            Endpoint A                              Endpoint B
```

Two endpoints, each with multiple WAN paths. The kernel module manages QUIC streams across the available links, scheduling packets based on real-time path metrics (RTT, observed loss, queue depth, metered/unmetered status).

---

## Test environment

- **Endpoints:** Two cloud VPSes, each provisioned with two reserved public IPs
- **Path verification:** Confirmed independent egress paths via reserved IPs at each endpoint to ensure genuine multipath rather than NAT shenanigans
- **Workloads under test:** TCP/UDP throughput, latency-sensitive flows, sustained transfers under simulated link degradation (loss injection, latency injection, asymmetric capacity)

---

## Status

🚧 **Active development.** Implementation lives in a private repository while design and stabilization work is ongoing.

This repository (`tquic-overview`) is intentionally README-only. It exists to document the project at a high level for recruiters, collaborators, and anyone interested in the design.

---

## Related work

This project sits in the same neighborhood as:

- **MPTCP** (Multipath TCP) — different transport, similar goals
- **MASQUE / CONNECT-UDP** — userspace approaches to multipath proxying
- **mptcpize / mptcpd** — userspace MPTCP daemons
- **picoquic, quiche, msquic** — userspace QUIC implementations with experimental multipath drafts

TQUIC differs primarily in placing path scheduling inside the kernel, with QUIC chosen specifically for its connection-ID model rather than as a port of MPTCP semantics.

---

## Contact

**Christopher "Justin" Adams**
[spotty118@gmail.com](mailto:spotty118@gmail.com)
Verizon Wireless — Assistant General Manager, Small Business
Alabama Gulf Coast

If you're working on multipath transports, kernel networking, or carrier-grade WAN bonding and want to compare notes, reach out.
