## Chapter 4 — Throughput

```
A.pop()          # removes A4
C.append(C4)

A = [A7, A6, A5]
C = [C1, C2, C3, C4]
```

**Relaxing A4; introducing C4.**

> **C4.** Read throughput, and independently write throughput, can each exceed the computational capacity of a single machine. Vertically scaling CPU and memory to absorb this load is expensive and, past a point, impractical.

When request throughput exceeds the capacity of a single machine, several general mitigation strategies are available before, or alongside, scaling out:

- **Queuing:** Temporarily buffering incoming requests to be processed once capacity becomes available.
- **Load Shedding:** Deliberately dropping or rejecting a fraction of incoming requests to keep the system within capacity.
- **Back-Pressure:** A flow-control mechanism in which a downstream component signals an upstream component to slow or stop sending further requests.

All three mechanisms trade throughput stability for increased latency, reduced availability, or both; none of them increases the system's underlying capacity.

A fourth mechanism, **rate limiting**, is often applied at admission rather than under saturation: each client, tenant, or key is granted a fixed quota of requests per unit time, and requests beyond that quota are rejected or delayed. Common implementations include the token bucket, leaky bucket, fixed window, sliding window counter, and sliding window log.

### 4.1 CAP and PACELC Theorems

This is a natural point at which to introduce one of the foundational results in distributed systems theory.

**CAP Theorem:** In the presence of a network partition, a scenario that cannot be avoided in any real network, a distributed system can guarantee either **Consistency** (a CP system) or **Availability** (an AP system), but not both simultaneously.

**PACELC Theorem:** An extension of CAP that additionally addresses the common case in which no partition is occurring. In the presence of a **P**artition, a system must choose between **A**vailability and **C**onsistency; **E**lse, that is, during normal operation, it must choose between **L**atency and **C**onsistency.

| Theorem | Governing condition | Trade-off |
|---|---|---|
| CAP | Network partition | Consistency vs. Availability |
| PACELC | Network partition (P) | Availability vs. Consistency |
| PACELC | Normal operation (E, "else") | Latency vs. Consistency |

These two results make explicit some of the most fundamental trade-offs a distributed system's design must resolve. Their terms are defined precisely as follows:

- **Consistency** (in this network-partition sense): Every read receives either the most recent write or an explicit error. Section 4.7 below refines this into a spectrum of stronger and weaker models; note again that this is a distinct notion from ACID's "Consistency," despite sharing a name.
- **Availability:** Every request receives a non-error response, without any guarantee that the response reflects the most recent write. A system may also operate in a state of *partial* availability, in which only some requests can be served.
- **Partition Tolerance:** The system continues to operate despite arbitrary message loss or the failure of part of the network.

C and A here are binary, worst-case notions: a system either satisfies linearizability or it does not, and is either available or is not. Sections 4.7 through 4.9 give the graded picture, namely which point on the consistency spectrum, and at what latency cost, a specific replication configuration actually achieves.

Although this chapter still operates under the assumption of a failure-free, partition-free network (A7 has not yet been relaxed; see Chapter 7), it is useful to introduce CAP and PACELC here, since the replication mechanisms discussed below are largely a response to the availability/consistency and latency/consistency trade-offs these theorems describe. Network partitions themselves, and their operational consequences, are treated in full in Chapter 7.

We now turn to mechanisms for absorbing throughput beyond the capacity of a single machine, beginning with the case in which only *read* throughput exceeds that capacity. **Replication**, the practice of maintaining multiple copies (**replicas**) of the same data on different machines in order to increase availability, increase read throughput, or reduce latency by serving reads from a replica nearer to the requester, is the primary tool available. Replicas must be kept synchronized with one another; three major replication architectures address this.

### 4.2 Single-Leader Replication

One node is designated the **leader**; all others are **followers**. The leader accepts and locally executes all write requests, then propagates the resulting updates to every follower in the order in which they were executed. Reads may be served by the leader or, to increase read throughput, by any follower.

This architecture scales read throughput easily by adding followers, but write throughput remains bounded by the capacity of the single leader, which becomes a bottleneck once write load alone exceeds what one machine can handle. The following two architectures address this limitation.

### 4.3 Multi-Leader Replication

Multiple nodes are simultaneously designated as leaders, each capable of accepting writes; leaders asynchronously propagate their local changes to one another. This allows writes to be accepted concurrently at different nodes, increasing both write throughput and availability, since a client can always find some leader to write to. The cost is added complexity from **write conflicts**, which arise whenever concurrent writes modify the same data item at two different leaders.

**Conflict Resolution.** Conflicts are, by construction, detected asynchronously, only once the concurrent writes are compared against one another. To minimize the frequency of conflicts, systems commonly route all writes for a given record, or all writes originating from a given user, through one designated leader; some systems relax even this constraint to further improve availability, accepting the resulting risk of temporary inconsistency. Where conflicts do occur, resolution strategies include **Last-Write-Wins (LWW)** based on timestamps; UUID- or hash-based tie-breaking; combining a timestamp with a replica identifier; ranking replicas by priority; merging the conflicting writes; or retaining all conflicting versions for later, application-level reconciliation. Resolution may be performed automatically, at write time, at read time, or during background reconciliation, or manually, for example by prompting an end user to resolve the discrepancy.

**Conflict-Free Replicated Data Types (CRDTs).** CRDTs are data structures specifically designed so that concurrent updates never need explicit conflict resolution. Replicas are permitted to diverge temporarily as operations propagate asynchronously; when replicas synchronize, their states are merged using a merge function that is commutative, associative, and idempotent by construction, rather than through operational transformation, which is a distinct technique used in some collaborative-editing systems. Because the merge function's mathematical properties guarantee the same result regardless of the order or duplication of the inputs, all replicas converge to an identical state without requiring negotiation, and eventual consistency is guaranteed structurally rather than through ad hoc reconciliation logic.

**Replication Topologies.** The pattern of communication links used to propagate writes between leaders defines the system's replication topology:

- **All-to-All:** Every leader communicates directly with every other leader, maximizing redundancy at the cost of more complex ordering guarantees, since messages can arrive via links of differing speed.
- **Ring:** Leaders are arranged in a logical circle, and updates are passed sequentially around the ring. This simplifies propagation logic but can introduce additional latency, since an update may need to traverse several intermediate nodes.
- **Star (or Tree):** A central node forwards updates to all other nodes, or a tree of nodes forwards updates to their children. This is efficient to disseminate from, but the central node is a single point of failure for propagation.

More elaborate topologies are possible and are used in some large-scale systems, but are not treated further here.

### 4.4 Leaderless Replication

No node is designated as primary. Clients, or an intermediate coordinator, send write requests directly to multiple replicas, and no strict global ordering of writes is enforced; there is correspondingly no leader-failover mechanism, since there is no leader to fail over from. Consistency is instead pursued through **quorums**: a write is considered successful once acknowledged by W replicas, and a read is considered successful once R replicas have responded, with the most up-to-date value among those R responses being returned to the client. A coordinator issues the request to all N replicas and waits for the requisite quorum to respond, returning an error if it cannot be reached. The condition **W + R > N** guarantees that every read quorum and every write quorum overlap in at least one replica, which is necessary, though as discussed in Section 4.9 not sufficient, for the read to observe the most recent write. W and R are tunable, subject to a trade-off between write latency and read latency.

**Anti-entropy** mechanisms, introduced briefly here and discussed in depth in Chapter 5, run in the background to reconcile any divergence between replicas that quorum reads and writes alone do not resolve.

### 4.5 Scaling Replicas

Adding a new replica typically involves taking a consistent snapshot of an existing node, usually the leader in single- or multi-leader configurations, and then applying every change made since that snapshot was taken, in order, using the write-ahead log. In a partitioned database, each partition may maintain its own leader, or its own independent replication scheme, which allows horizontal scaling and fault tolerance to be achieved along both the partitioning and replication axes simultaneously.

### 4.6 Replication Modes

Independently of *which* replication architecture is used, replication can be configured with different synchronization guarantees:

- **Synchronous:** A transaction is applied locally on the leader, then applied on every follower, and only then is an acknowledgment returned to the client. This guarantees that all nodes present a single, consistent view of the data at essentially all times, at the cost of write latency bounded by the slowest follower.
- **Asynchronous:** The client receives an acknowledgment immediately after the local update on the leader completes; followers apply the corresponding update independently and asynchronously, at their own pace.
- **Partially (Semi-)Synchronous:** A hybrid in which a subset of followers is configured for synchronous replication, typically to guarantee some minimum durability, while the remaining followers replicate asynchronously.

In PACELC terms, synchronous replication is the "Else, prioritize Consistency" choice; asynchronous replication is "Else, prioritize Latency"; and semi-synchronous replication tunes the mix by varying how many followers sit on the consistency-preserving path.

### 4.7 Data-Centric Consistency Models

Where isolation levels (Chapter 1) govern the interaction between *concurrent transactions on a single logical copy of the data*, data-centric consistency models govern how the *multiple physical replicas* of a distributed system are permitted to differ from one another, and for how long. Ordered from strongest to weakest:

- **Strict Serializability** (= Serializability + Linearizability): The strongest available combined guarantee, under which concurrent transactions appear to execute in a single serial order that additionally respects real, wall-clock time.
- **Linearizability:** A guarantee that the system behaves as though there were only a single copy of the data, with every operation on it taking effect atomically at some instant between its invocation and its response. Once a write commits, every subsequent read, by any client, observes that write's value or a later one.
- **Sequential Consistency:** A guarantee that the result of any execution is equivalent to some sequential, interleaved ordering of all nodes' operations, in which each individual node's own operations appear in the order that node issued them, but without the real-time constraint that linearizability imposes.
- **Causal Consistency:** A weaker guarantee than sequential consistency, requiring only that operations which are causally related be observed in the same relative order by every process; operations that are concurrent, that is, causally unrelated, may be observed in different orders by different processes.
- **Eventual Consistency:** The weakest of the models listed here: if no further updates are made to a given data item, every replica will *eventually* converge to the same value for that item, with no bound on how long convergence takes and no guarantee about what is observed in the meantime.

These models form a strength hierarchy:

```
Strict Serializability → Linearizability → Sequential Consistency → Causal Consistency → Eventual Consistency
```

### 4.8 Client-Centric Consistency Models

Independently of the replica-wide guarantees above, a system may offer guarantees scoped to a single client's own sequence of operations, often called session guarantees:

- **Monotonic Reads:** Once a client has read a given value for a data item, no subsequent read by that same client will return an older value for that item.
- **Monotonic Writes:** A client's writes are applied in the order that client issued them.
- **Read-Your-Writes:** A client is guaranteed that any subsequent read it performs on a data item will return a value at least as recent as the last value that client itself wrote.
- **Write-Follows-Reads:** A write issued by a client after a read is guaranteed to be applied to a version of the data item at least as recent as the version that the read observed.

Under the network assumptions still active at this stage (A5), these guarantees are rarely violated in practice, since propagation between replicas is instantaneous and only process delay (Chapter 1) can cause a follower to lag; Chapter 5 shows how real network latency makes these violations materially more likely and these guarantees correspondingly more important to enforce explicitly.

### 4.9 Analyzing Replication Schemes and Consistency

We can now analyze which consistency models each replication architecture achieves, given the constraints introduced so far, and it is worth emphasizing that this analysis still assumes a single, perfectly synchronized global clock, since assumption A6 remains active at this stage. Chapter 6 revisits this same analysis once clock synchronization is relaxed.

**Single-leader replication.** The leader can sequence every write locally, for instance by arrival timestamp under the still-active global-clock assumption, and every replica applies updates in that same sequence. Under synchronous replication, this is sufficient to achieve linearizability. Under asynchronous replication, process delays mean a follower's applied state can lag the leader's. Write ordering is still preserved, so each replica always exposes some prefix of the same single write sequence, but a read served by a lagging follower may not reflect the most recent write. Linearizability therefore cannot be guaranteed for reads under asynchronous replication, though it still holds for the write sequence itself. Sequential consistency is not guaranteed in the general case either, since a client that writes to the leader and then reads from a lagging follower may fail to observe its own write, violating the per-client program order that sequential consistency requires. It is recovered once reads are routed to the leader, or once the session guarantees of Section 4.8 are enforced explicitly; absent those, the guarantee that survives is a consistent prefix of the leader's write order together with eventual convergence.

**Multi-leader replication.** Each leader executes its own local writes independently and propagates them asynchronously to the other leaders. Sequential consistency, which requires a single, globally agreed order consistent with each node's own issue order, is therefore not achievable in general. Under the current assumption of a synchronized global clock, however, resolving conflicts by Last-Write-Wins on commit timestamps correctly reflects true causal and real-time order, and because propagation is still instantaneous under A5, every replica applies writes in that same order; causal consistency is achievable this way. Alternatively, eventual consistency alone can be achieved through weaker reconciliation strategies, such as ranking replicas by a fixed priority ("node importance"). Because writes still lack a single enforced global order even when timestamps are trustworthy, conflicts between concurrent writes remain possible and must be resolved by one of the mechanisms described in Section 4.3. This result is provisional: it depends entirely on the trustworthiness of node clocks, and Section 6.6 revisits it once that assumption is relaxed, showing that the more robust route to causal consistency is via version vectors rather than timestamp comparison.

**Leaderless replication.** Because the read and write quorums are guaranteed to overlap (W + R > N), it might appear that linearizability follows directly. This is not the case: concurrent writes may be applied in different orders at different replicas, so that fewer than W replicas end up reflecting each individual concurrent write, and a subsequent quorum read using Last-Write-Wins can therefore return inconsistent results depending on exactly which R replicas respond. Causal consistency, however, is achievable with appropriate anti-entropy mechanisms in place to reconcile replicas that quorum operations alone leave divergent.

### 4.10 Chapter Summary

Under A4, throughput was never a constraint: a single machine could absorb any read or write load, so the question of how many copies of the data to keep, and how tightly to synchronize them, never arose. Constraint C4 removes that ceiling and, in doing so, forces the system to confront the CAP and PACELC theorems directly. Because read and write throughput must now be spread across multiple machines via replication, and because a network partition is an unavoidable possibility, the system must decide, explicitly or by default, whether it prioritizes consistency or availability when a partition occurs, and latency or consistency when it does not.

The three replication architectures surveyed in this chapter are best understood as three different resolutions of that same underlying tension. Single-leader replication preserves a simple, strongly ordered write path and therefore the strongest achievable consistency, but reintroduces a throughput bottleneck at the leader. Multi-leader replication removes that bottleneck and reduces write latency for geographically distributed clients, but reintroduces conflicting concurrent writes, which the chapter's conflict-resolution strategies and CRDTs address either by picking a deterministic winner or by making the merge operation itself commutative and conflict-free. Leaderless replication sacrifices any single point of ordering in favor of tunable, quorum-based availability, whose W and R parameters directly trade write latency against read latency and whose anti-entropy mechanisms are required to paper over the ordering guarantees quorums alone cannot provide. The consistency models introduced along the way give this design space a common vocabulary, letting the chapter's closing analysis show precisely which consistency ceiling each architecture-and-mode combination is capable of reaching.

In practice, single-leader synchronous replication is appropriate wherever strong consistency is worth its latency and availability cost, as in financial ledgers and systems of record; single-leader asynchronous replication where read scaling matters more than absolute freshness; multi-leader replication where writes originate from multiple geographic regions or must succeed even when connectivity to a central leader is degraded; and leaderless, quorum-based replication where the priority is maximal write and read availability at large scale and the application can tolerate, detect, or reconcile occasional conflicts. No architecture is dominant across consistency, availability, and latency simultaneously, which is precisely what CAP and PACELC assert.

---
