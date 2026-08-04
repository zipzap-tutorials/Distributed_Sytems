## Chapter 5 — Network

```
A.pop()          # removes A5
C.append(C5)

A = [A7, A6]
C = [C1, C2, C3, C4, C5]
```

**Relaxing A5; introducing C5.**

> **C5.** In real-world distributed systems, network latency is non-zero, variable, and dependent on physical distance, and bandwidth is finite.

The standard model for a realistic network link is the **fair-loss link**: messages may be lost, duplicated, or delivered out of order, but a combination of timeouts and retries ensures that a message retried indefinitely is eventually delivered. Building reliable messaging over a fair-loss link requires choosing among three delivery semantics: **exactly-once** processing, the ideal and most difficult to guarantee; **at-most-once** delivery, which guarantees no duplicates at the cost of possible message loss; and **at-least-once** delivery, which guarantees no message loss at the cost of possible duplicate delivery. **Idempotence**, the property that an operation can be safely repeated multiple times with the same effect as applying it once, combined with **deduplication**, allows an at-least-once channel to be used to build an effectively exactly-once processing guarantee at the application layer.

### 5.1 Communication Protocols

As network delay and finite bandwidth become first-class concerns, the choice of communication protocol involves explicit trade-offs between ordering guarantees, reliability, and efficiency:

- **TCP (Transmission Control Protocol):** A core internet transport protocol providing reliable, ordered, error-checked byte-stream delivery between applications.
- **UDP (User Datagram Protocol):** Provides fast but unreliable message delivery; it performs no flow control and does not retransmit lost packets, avoiding some of the sources of variable delay inherent to TCP at the cost of delivery guarantees.
- **HTTP / HTTPS:** The Hypertext Transfer Protocol is the foundational protocol for transferring data on the web; HTTPS is its encrypted variant. **REST** (Representational State Transfer) is a set of architectural conventions for structuring HTTP-based APIs.
- **FTP (File Transfer Protocol):** A standard protocol for transferring files between a client and a server over TCP/IP.
- **SSH (Secure Shell):** A cryptographic protocol for securely connecting to and managing remote machines over an otherwise unsecured network.
- **RPC (Remote Procedure Call):** An inter-process communication style that allows a process to invoke a subroutine that executes on a different machine, while abstracting away most of the underlying network mechanics.
- **SSE (Server-Sent Events):** A protocol allowing a client to hold open a long-lived connection over which a server can push data to the client, without the client needing to re-initiate a request for each message.
- **Short Polling:** The client repeatedly requests data from the server; the server responds immediately, either with the requested data or an empty response if none is yet available.
- **Long Polling:** Similar to short polling, but with a much longer timeout: the server holds the connection open until data becomes available, or the timeout elapses, and the client immediately re-polls upon receiving a response.
- **WebSockets:** Establishes a full-duplex communication channel over a single TCP connection, allowing bidirectional data transfer between client and server for the lifetime of the connection.
- **Webhooks:** An event-driven mechanism by which a server pushes data to a client via an HTTP POST request whenever a predefined event occurs, without the client maintaining an open connection.

### 5.2 Information Dissemination Mechanisms

**Gossip (epidemic) protocols** disseminate information via probabilistic, cooperative propagation, modeled on epidemic spread. An "infective" process that has just learned an update spreads it to a random subset of "susceptible" peers, which in turn do the same. Once a process has received the same update repeatedly, a signal that it has likely already reached the entire cluster, that process stops propagating it and transitions to a "removed" state.

### 5.3 Anti-Entropy Mechanisms

**Read Repair.** An anti-entropy mechanism that detects and corrects divergence between replicas at read time. When a read is served, the coordinator compares the responding replicas' versions of the requested data item; if any replica is found to be out of date, the coordinator forwards the missing update or updates to that replica, restoring consistency. Read repair can be performed **synchronously**, blocking the client's read until all queried replicas are brought up to date, or **asynchronously**, by scheduling the correction as a background task so that it does not add latency to the read itself.

**Merkle Trees.** An anti-entropy mechanism for reconciling data that is queried infrequently. Comparing two replicas record by record is prohibitively expensive, so each replica instead maintains a compact hashed summary of its own contents: a tree whose leaves hash contiguous ranges of records, and whose internal nodes hash the concatenated hashes of their children, so that a single root hash summarizes the entire dataset. Two replicas reconcile by exchanging hashes recursively, from the root downward, descending only into those subtrees whose hashes disagree; only the records in the leaf ranges that remain divergent are then compared and repaired. Locating a divergence therefore costs a number of comparisons logarithmic in the number of leaf ranges, rather than the linear cost of a pairwise scan over records. The cost is that every write invalidates the hashes on the path from its leaf to the root and forces their recomputation, so the size of the tree is itself a tunable trade-off: finer leaf ranges localize divergence more precisely and narrow what must be transferred to repair it, at the price of a taller tree and more hash recomputation on every update.

### 5.4 Implications for Replication

**Geo-distributed systems.** Because network delay is variable and grows with the physical distance between client and server, geographically distributed deployments commonly place nodes closer to end users to reduce perceived latency. Multi-leader replication is particularly well suited to this pattern; indeed, minimizing write latency for geographically dispersed clients is one of multi-leader replication's principal advantages over its single-leader and leaderless counterparts.

**Re-analyzing consistency models.** Under the current constraints (C1–C5), the consistency results derived in Chapter 4 for single-leader replication carry over unchanged, since network delay simply adds to the process delay already accounted for there, though synchronous replication now incurs an additional, distance-dependent latency penalty that makes it materially harder to scale across regions. For multi-leader and leaderless replication, Last-Write-Wins conflict resolution can still achieve causal consistency under the present constraints, since, as in Chapter 4, a single synchronized global clock (A6) remains assumed at this stage. What changes is that propagation is no longer instantaneous, so replicas must apply and expose conflicting writes in timestamp order rather than in order of arrival for that guarantee to hold. This assumption is the next to be relaxed.

### 5.5 Chapter Summary

Assumption A5 let Chapter 4's replication architectures be analyzed as though propagation between nodes were instantaneous, so the only source of staleness under asynchronous replication was process delay. Constraint C5 introduces real, distance-dependent network latency together with a fair-loss link that may lose, duplicate, or reorder messages, which raises two new problems: how to build any reliable guarantee at all, whether ordering, delivery, or exactly-once processing, on top of a channel that offers none of these by default, and how the now-material cost of propagating updates between nodes should shape both the choice of communication protocol and the placement and reconciliation strategy of a replicated system.

The chapter's mechanisms resolve these problems along complementary lines. Delivery semantics combined with idempotence and deduplication let an unreliable, at-least-once channel be used to build effectively exactly-once application behavior. The survey of communication protocols matches each interaction pattern to the transport best suited to it, trading strict reliability and ordering (TCP) against raw speed and loss tolerance (UDP). Gossip protocols disseminate information across large clusters without relying on any single broadcast path, trading deterministic delivery for scalable, probabilistic convergence. Read repair, as an anti-entropy mechanism, opportunistically closes the gap that quorum operations alone leave open, trading either read latency (synchronous repair) or short-lived staleness (asynchronous repair) for eventual convergence. Merkle trees close the same gap from the opposite direction, comparing replicas by hash tree so that divergence in data no read has touched is still found, trading the upkeep of the tree and the cost of the comparison itself for coverage that does not depend on the access pattern. These mechanisms, taken together, are what make Chapter 4's replication architectures viable once propagation delay is real rather than theoretical, and the geo-distribution discussion shows directly how distance-dependent latency motivates multi-leader replication's core advantage.

The practical guidance follows the same shape as the trade-offs themselves: reliable, ordered protocols for control-plane and correctness-sensitive traffic; loss-tolerant protocols for latency-critical or high-volume streams where occasional loss is acceptable; gossip-based dissemination for large or highly dynamic clusters where centralized broadcast would not scale; and synchronous anti-entropy only where the read path's consistency is worth its added latency, with Merkle comparison wherever divergence in rarely-read data cannot be allowed to accumulate. As elsewhere in this framework, the right choice tracks the specific latency, reliability, and scale requirements of the workload rather than any single default.

---
