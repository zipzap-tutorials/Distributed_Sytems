## Chapter 6 — Clocks

```
A.pop()          # removes A6
C.append(C6)

A = [A7]
C = [C1, C2, C3, C4, C5, C6]
```

**Relaxing A6; introducing C6.**

> **C6.** In real distributed systems, there is no universal notion of time. Each machine maintains its own local clock, typically driven by an inexpensive quartz oscillator; because distinct oscillators drift at different rates, **clock skew** inevitably develops between machines over time. Time in a distributed system is therefore relative, not absolute.

The absence of a single, authoritative global clock means that the precise global ordering of events can no longer be determined from timestamps alone. This undermines, in particular, the Last-Write-Wins conflict-resolution strategy relied upon in Chapters 4 and 5, since those results depended on timestamps faithfully reflecting true chronological, and therefore causal, order. This chapter develops the mechanisms used to reason about ordering once that assumption no longer holds.

### 6.1 Synchronized Clocks

Even without a perfect global clock, several mechanisms narrow the practical impact of clock skew. Monotonically increasing transaction identifiers can be used for sequence ordering within a single-leader replication scheme, sidestepping any need for wall-clock time at all. In more sophisticated environments, clocks can be modeled as reporting a **confidence interval**, a bounded range guaranteed to contain the true time, and a process can wait out the width of this interval before committing a transaction, ensuring that intervals for causally ordered transactions do not overlap and thereby preserving a consistent, externally observable order. (Bounded-uncertainty schemes of this kind, combining GPS and atomic clock references, are used in some globally distributed database systems to bound clock uncertainty to a few milliseconds.) Atomic-clock-based synchronization more generally is used to keep clock skew within acceptable bounds across a fleet of machines, improving the reliability of any mechanism that depends on approximate wall-clock agreement. A classic implementation for keeping physical clocks in approximate agreement is the **Network Time Protocol (NTP)**, a hierarchical synchronization scheme in which atomic and GPS reference clocks occupy the top stratum and each successive stratum synchronizes from the one above it over an IP network, with synchronization accuracy degrading at each progressively lower level from the global reference.

### 6.2 Event Ordering

Event ordering in a distributed system can be either a **total order** or a **partial order**. A total order places every pair of events in a definite sequence, regardless of whether the events are causally related, yielding a single, comprehensive timeline of all system activity. A **causal (partial) order**, by contrast, orders only those pairs of events that are causally linked, leaving causally unrelated events unordered with respect to one another.

Unique identifiers are frequently used to impose a total order without coordination between nodes. Randomly generated schemes (UUIDv4) are unique but carry no ordering information at all, whereas time-ordered schemes (UUIDv7) place a timestamp in the high-order bits so that identifiers sort in approximately the order they were generated. The order a time-ordered identifier induces is only as trustworthy as the clock that produced it, so it approximates causal order rather than tracking it.

### 6.3 Causal Relationships

The **happened-before relation**, denoted →, defines a strict partial order over events, capturing causal dependency: if event E1 happened-before E2, E1 may have causally influenced E2. This relation is **transitive**: if E1 → E2 and E2 → E3, then E1 → E3. Two events for which neither E1 → E2 nor E2 → E1 holds are said to be **concurrent**, written E1 ‖ E2, meaning they are causally independent and may be observed in either order, or simultaneously, without violating causal consistency.

### 6.4 Logical Clocks

**Logical clocks** are integer counters, initialized to zero at each node, that serve as a proxy for local and global logical time. Each node increments its counter by one whenever a local event, a message send, or a message receive occurs. This simple mechanism is the basis for the two schemes below.

#### Lamport Clocks

A Lamport clock is a per-node logical clock that is incremented by one immediately before each local event. Outgoing messages carry the sender's current clock value; on receiving a message, a node sets its own clock to the maximum of its current value and the received value, then increments by one before processing the message. This scheme guarantees an ordering of events that is *consistent* with causality: if E1 → E2, then the Lamport timestamp of E1 is less than that of E2. Because two events on different nodes can carry equal timestamps, a total order requires an additional deterministic tie-break, conventionally on node identifier. The converse of the causal property does not hold: a smaller Lamport timestamp does not imply a causal relationship. Lamport clocks are therefore suitable for establishing a total order, but are insufficient, on their own, to recover causal relationships between arbitrary event pairs.

#### Vector Clocks

A vector clock generalizes the logical clock into a vector of length N, one entry per node in an N-node system. Each node increments only its own entry when it performs an event, including message sends and receives, and attaches its current vector to every outgoing message. On receiving a message, a node updates each entry of its own vector to the maximum of its current value and the corresponding entry in the received vector. Vector clocks satisfy the **strong clock condition**: event E1, with vector clock V1, happened-before event E2, with vector clock V2, if and only if every entry of V1 is less than or equal to the corresponding entry of V2, and at least one entry is strictly smaller. Because this condition is both necessary and sufficient for causal precedence, unlike the Lamport clock's condition, which is only necessary, vector clocks allow the causal partial order between any two events to be recovered exactly from their timestamps alone.

### 6.5 Managing Data Consistency

**Version vectors** apply the same underlying idea as vector clocks, but to the partial ordering of *operations on individual data items* (writes, updates, deletions) rather than to events in general; they function as a per-item version-control mechanism in the presence of clock skew. Each version vector begins with every entry set to zero; a local update increments the entry corresponding to the writing node. During synchronization, two replicas' vectors are merged element-wise by taking the maximum of each entry, exactly as with vector clocks. Every write is stored together with its version vector, the identifier of the client or node that issued it, and the resulting data value; entries can then be safely overwritten when one is causally after the other, while concurrent, causally unrelated writes are flagged for explicit conflict resolution rather than being silently discarded. Because a version vector must be tracked per data item, and must have one entry per client or server that might write that item, this approach can incur significant storage overhead in systems with many writers; **dotted version vectors** are an optimization that reduces this overhead while preserving the same causal-tracking guarantees.

### 6.6 Revisiting Replication

We can now revisit the consistency analysis of Chapters 4 and 5 in light of clock skew.

For **synchronous single-leader replication**, the earlier results are unaffected: the leader still sequences every write itself, and every follower applies updates in that same leader-determined order, independent of wall-clock time.

For **multi-leader and leaderless replication**, however, the earlier reliance on Last-Write-Wins conflict resolution no longer holds up. Because C6 means timestamps across nodes can no longer be trusted to reflect true chronological, and therefore causal, order, LWW can now violate causal consistency outright, and in the worst case can silently discard a causally later write in favor of an earlier one that merely carries a larger, skewed timestamp, resulting in genuine data loss rather than a mere ordering anomaly. Achieving causal consistency under multi-leader or leaderless replication now requires version vectors, or an equivalent causality-tracking technique, in place of naïve timestamp comparison; absent such a mechanism, only eventual consistency can still be guaranteed.

### 6.7 Chapter Summary

As long as A6 held, every mechanism in the preceding chapters could treat a timestamp as ground truth: Last-Write-Wins conflict resolution in multi-leader and leaderless replication (Chapters 4 and 5) correctly reflected true causal order simply because all clocks agreed. Constraint C6 withdraws that guarantee, since clock skew means physical timestamps across nodes can no longer be trusted to reflect true chronological, and therefore causal, order. The chapter's central problem is consequently not merely academic: it exposes that the earlier reliance on LWW can now silently discard a causally later write in favor of an earlier one that merely carries a larger, skewed timestamp, turning an ordering anomaly into genuine data loss.

The chapter's mechanisms restore ordering guarantees without relying on a trustworthy global clock, at increasing levels of cost and precision. Monotonic transaction identifiers and bounded-uncertainty clock schemes sidestep the problem entirely for specific architectures, at the cost of either being restricted to single-leader designs or requiring specialized, tightly synchronized clock infrastructure. Lamport clocks provide a cheap, scalar order consistent with causality but cannot, by themselves, distinguish a causal relationship from a coincidental one. Vector clocks, together with their per-data-item specialization as version vectors, satisfy the strong clock condition and therefore recover the true causal partial order exactly, at the cost of storage and comparison overhead that grows with the number of nodes or writers, an overhead the dotted-version-vector optimization partially reclaims. The trade-off throughout is expressiveness versus overhead, since a scalar Lamport clock is nearly free but cannot detect concurrency while a full vector or version vector detects concurrency exactly but costs space linear in the number of writers. That trade-off is compounded by a second one between logical mechanisms, which need no special infrastructure but carry no wall-clock meaning, and physical bounded-clock mechanisms, which are directly usable for external consistency but require investment in specialized hardware.

In practice, systems that only need a consistent total order, such as event logging or single-leader sequencing, can rely on Lamport clocks or transaction identifiers; systems running multi-leader or leaderless replication that must detect genuine write conflicts rather than silently resolve them by unreliable timestamp should adopt version vectors regardless of the added bookkeeping; and globally distributed systems that require strong, externally consistent guarantees across regions should consider bounded-uncertainty clock infrastructure only when the cost of atomic-clock synchronization and commit-wait latency is justified by the application's consistency requirements. There is no causality-tracking mechanism that is simultaneously free, precise, and infrastructure-independent.

---
