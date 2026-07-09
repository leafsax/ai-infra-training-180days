### Day 126: Congestion Control in RDMA/RoCEv2 (ECN, PFC, DCQCN)

**1) Topic and Core Examination Areas**
- Topic: Congestion Control in RDMA/RoCEv2.
- Core Examination Areas: Understanding network congestion in RDMA/Ethernet, PFC (Priority Flow Control), ECN (Explicit Congestion Notification), and DCQCN (Data Center Quantized Congestion Notification).

**2) Requirement Clarification and Metric Definitions**
- **Packet Loss Rate**: Target <10^-5 for RoCEv2/IB networks. Even 0.1% packet loss can cause severe throughput degradation in RDMA due to retransmissions.
- **Buffer Size**: Switch on-chip buffers should be sized to absorb micro-bursts (e.g., 1-2 ms of line-rate traffic, ~100-200 MB for 800G switches).
- **Latency Spikes**: Tail latency (TP99) should not exceed 1-2x the base RTT during congestion events.

**3) Core Architecture/Technical Component Design**
- *PFC (Priority Flow Control)*: A link-level mechanism that pauses specific traffic classes (priorities) when a switch port's buffer reaches a threshold.
- *ECN (Explicit Congestion Notification)*: Marks packets instead of dropping them when congestion is detected, allowing receivers to signal senders to reduce rate.
- *DCQCN*: A congestion control algorithm for RoCEv2 that combines PFC and ECN to achieve high throughput and low latency without deadlocks.

**4) Deep Dive into Key Technologies and Possible Solutions**
- *PFC Deadlocks*: If multiple ports pause each other, a PFC storm or deadlock can occur, halting all traffic. Solutions include PFC watchdogs, asymmetric PFC, or ECN-based congestion control (DCQCN) which minimizes PFC usage.
- *ECN + DCQCN*: ECN marks packets when buffer occupancy crosses a threshold. The receiver echoes the ECN mark back to the sender via RDMA headers. The sender reduces its transmission rate, avoiding PFC pauses.

**5) Trade-off analysis**
- *PFC-heavy vs ECN-heavy*: PFC provides immediate pause but risks deadlocks and global flow stalling. ECN is smoother and avoids deadlocks but requires precise tuning of marking thresholds and sender rate reduction algorithms.
- *Complexity vs Reliability*: DCQCN is complex to tune (ECN thresholds, PFC high/low watermarks) but provides the most reliable performance for AI workloads at scale.

**6) How to determine the optimal solution**
- For production RoCEv2 AI clusters, use DCQCN with carefully tuned ECN marks and minimal PFC usage. Avoid PFC-only configurations to prevent PFC deadlocks. Regularly run congestion benchmarking tools (e.g., `rcache`, `pfc-starve-test`).

**7) Full names and explanations of all nouns and abbreviations**
- **DCQCN**: Data Center Quantized Congestion Notification — an RDMA congestion control algorithm for RoCEv2 networks.
- **PFC Storm**: A condition where PFC pauses cascade across switches, causing network paralysis.
- **Watermark**: A buffer occupancy threshold that triggers PFC or ECN actions (low watermark to resume, high watermark to pause/mark).

---

