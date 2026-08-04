## Chapter 7 — Failures

```
A.pop()          # removes A7
C.append(C7)

A = []
C = [C1, C2, C3, C4, C5, C6, C7]
```

**Relaxing A7; introducing C7.**

> **C7.** Nodes can be taken down, fail and later recover, or crash and stop permanently; networks can be partitioned, isolating groups of nodes from one another and causing **omission faults**, the loss of individual messages as distinct from the failure of an entire node.

With this final relaxation, every idealizing assumption has now been dropped, and the model reflects the full set of failure modes a production distributed system must tolerate.

### 7.1 Foundational Definitions

- **Fault Tolerance:** The ability of a system to continue functioning correctly even when some machines fail or the network partitions, typically achieved through redundancy.
- **Liveness Property:** A guarantee that some specific, intended event *will eventually* occur, for example "a leader will eventually be elected."
- **Safety Property:** A guarantee that some specific, unintended event *will never* occur, for example "two nodes will never simultaneously believe themselves to be leader."
- **Node Failure Models.** Distributed algorithms are typically designed against one of the following assumptions about how a node can fail:
  - **Crash-Stop:** A faulty node may crash at any moment and, once crashed, never resumes execution.
  - **Crash-Recovery:** A faulty node may crash at any moment, losing its in-memory state, but may subsequently resume execution, typically by replaying durable state from disk.

### 7.2 Failure Detection

Simple failure detectors rely on periodic **heartbeats** or **pings** to determine, via silence beyond some fixed timeout, whether a node is alive. The **Phi Accrual Failure Detector** improves on this fixed-timeout approach with a statistical model: rather than a binary alive/dead verdict, it computes a continuously varying suspicion level (denoted φ) from the observed timing and frequency distribution of recent heartbeats, allowing applications to trade detection speed against false-positive rate rather than fixing that trade-off in advance.

### 7.3 Achieving Atomicity in Distributed Transactions

Guaranteeing atomicity across multiple nodes, that is, ensuring that a transaction either commits on every involved node or aborts on all of them, requires an explicit coordination protocol. The three principal families are Two-Phase Commit, Three-Phase Commit, and quorum-based protocols.

**Two-Phase Commit (2PC).** A designated coordinator, which may also be one of the participants, drives the protocol in two phases. In the **voting phase**, the coordinator sends the proposed transaction to every participant, each of which executes its local portion of the work, durably records its readiness, and replies with a "yes" or "no" vote. If every participant votes "yes," the coordinator proceeds to the **commit phase**, instructing all participants to commit; if any participant votes "no," or fails to respond within a timeout, the coordinator instead instructs all participants to abort. A participant that fails during the voting phase is treated as having voted "no"; a participant that fails during the commit phase is caught up once it recovers, since the coordinator's decision has already been made and the remaining participants proceed to completion regardless. The critical weakness of 2PC is that a coordinator failure after the voting phase but before all commit/abort instructions are delivered leaves the surviving participants **blocked**, unable to safely commit or abort on their own, until the coordinator recovers, which can significantly harm availability.

**Three-Phase Commit (3PC).** 3PC addresses 2PC's blocking problem by inserting an additional phase before the final commit: once every participant has voted to commit, the coordinator broadcasts a pre-commit acknowledgment to all participants before proceeding to the final commit instruction, so that any participant receiving this acknowledgment thereby knows every other participant is prepared and can independently deduce the eventual outcome even if the coordinator subsequently fails. Combined with per-phase timeouts, this allows participants to safely commit or abort on their own if the coordinator disappears. However, 3PC does not fully solve the problem: its non-blocking property depends on bounded message delay and accurate failure detection, neither of which holds under a network partition, and a partition can still leave some nodes committing while others abort, producing an inconsistent outcome. Quorum-based protocols, discussed under Consensus below, are generally required to close this remaining gap.

### 7.4 Ensuring Isolation in Distributed Transactions

**Distributed locking**, commonly implemented via **leases**, which are locks that automatically expire after a timeout, extends pessimistic concurrency control across nodes, but introduces its own safety risk: a process may believe it still holds a lease that has, in fact, already expired (for instance, due to a long garbage-collection pause or network delay), and may then act on stale authority. **Fencing tokens** address this: each lease acquisition is issued a monotonically increasing token, and any resource protected by the lease rejects operations tagged with an older token than one it has already seen, safely blocking a node whose lease has silently expired and notifying it once the expiry is detected.

### 7.5 Failure Handling Mechanisms

The implications of a replica failure differ depending on the replication scheme in use.

**Leaderless replication.** **Hinted handoff**, used together with **sloppy quorums**, allows a write to succeed even when one of its target replicas is temporarily unreachable: the coordinator stores the write as a "hint" on another, healthy node, and replays it to the intended target once that node recovers. This trades a temporary weakening of the W + R > N guarantee for improved write availability during partial outages. If the node holding the hint is lost permanently before the handoff completes, however, the write it was holding is lost with it, a risk that anti-entropy and read repair can reduce but not eliminate, and one that Section 7.12 revisits in the context of eventual consistency's convergence guarantee.

**Leader-based replication.**
- **Replica (Follower) Recovery:** When a follower fails and later recovers, its write-ahead log identifies the last transaction it had durably applied; the leader then streams every subsequent update to bring it back in sync.
- **Leader Failover:** The failure of the leader itself requires a **re-election**: the failure must first be detected, typically via heartbeats or pings; the best available candidate, ideally the most up-to-date remaining replica, must be selected; and the cluster must then be reconfigured to recognize the new leader. This process must be handled carefully to avoid data loss, particularly when the newly elected leader has not yet replicated every write the old leader had accepted before failing.

### 7.6 Leader Election in Distributed Systems

Because many distributed algorithms are simpler to reason about with a single coordinating node, leader (or coordinator) election is a recurring building block. A correct election protocol must maintain the liveness property that a leader is active for most of the system's operating time, while also guarding against the **split-brain problem**, in which a partition or a botched election leaves two nodes simultaneously believing themselves to be leader, a safety violation that can lead to directly conflicting operations.

The principal leader-election algorithms are:

- **Bully Algorithm:** Nodes are assigned unique, totally ordered ranks. On detecting a leader failure, a node initiates an election; higher-ranked nodes "bully" lower-ranked ones out of contention, and the highest-ranked surviving node becomes leader.
- **Next-in-Line Failover:** The current leader maintains an explicit, pre-agreed list of backup nodes in priority order. On failure, the highest-ranked available node on that list is promoted directly, avoiding the overhead of a full election.
- **Candidate/Ordinary Nodes:** Nodes are divided into two roles; ordinary nodes are responsible for initiating an election among the candidate nodes, and the highest-ranked active candidate wins, with an explicit tie-breaking rule for elections that start simultaneously.
- **Invitation Algorithm:** Every node begins as the leader of its own single-node group and invites other nodes to join. Groups merge whenever two group leaders discover and connect to each other, eventually converging on a single group with a single leader.
- **Ring Algorithm:** Nodes are arranged in a logical ring; an election message circulates around the ring, and the highest-ranked node encountered during a full traversal is elected leader.

These algorithms are effective at *initiating* a new election during failover, but on their own do not guarantee the safety property needed to prevent split-brain during network partitions, since a partition can leave two sides of the cluster each electing their own leader independently. Achieving that stronger safety guarantee requires consensus.

### 7.7 Consensus Algorithms

**Consensus** is the process by which multiple processes in a distributed system agree on a single value. Consensus is trivial under ideal conditions, with no network delay, no node failures, and no partitions, but real systems must solve it under exactly the conditions those idealizations exclude, which is what makes consensus one of the most consequential problems in distributed systems. A correct consensus algorithm must satisfy three properties:

- **Agreement:** No two correct, non-faulty processes decide on different values.
- **Validity:** The value that is decided must have been proposed by one of the participating processes, ruling out trivial "solutions" that simply hard-code an answer.
- **Termination:** Every correct process eventually reaches a decision.

**Paxos.** Paxos structures consensus around three roles: **proposers**, who receive values from clients and put forward proposals; **acceptors**, who vote on proposals; and **learners**, who record the outcome of accepted proposals. A proposal is only accepted once a quorum of acceptors has voted for it. Each proposal carries a proposal number, conventionally a combination of a monotonic counter and the proposing node's identifier, used to establish a total order over proposals and resolve conflicts between competing ones. Despite its reputation for complexity, Paxos guarantees safety, that is, agreement, even in the presence of node failures and an unreliable network, through this quorum-based voting and replication process; liveness, that is, termination, is not guaranteed in the fully general asynchronous case, a consequence of the FLP impossibility result discussed in Section 7.10, but is readily achieved in practice with reasonable timeouts.

**Raft.** Raft was designed explicitly to be easier to understand and implement than Paxos while providing equivalent guarantees. Each node is at all times in one of three states, **leader**, **follower**, or **candidate**, and the protocol proceeds through two largely independent sub-protocols: leader election and log replication. All nodes start as followers; a follower that stops receiving heartbeats from a leader transitions to candidate and starts an election, becoming leader if it wins a majority of votes, or reverting to follower otherwise. Once elected, the leader is solely responsible for handling client requests: it appends each request to its own log and replicates that log entry to its followers; once a majority of the cluster has acknowledged the entry, the leader commits it, and consensus on that entry is achieved.

### 7.8 Broadcast Protocols

Broadcast protocols disseminate a message from one process to a group, and are the basic building block underlying both database replication and higher-level distributed coordination. Three broadcast orderings are commonly distinguished:

- **FIFO Broadcast:** Guarantees that messages originating from any single process are delivered to every recipient in the order that process sent them, though messages from different senders may still interleave arbitrarily.
- **Causal Broadcast:** Guarantees that causally related messages, regardless of sender, are delivered in an order consistent with their causal dependencies.
- **Total Order (Atomic) Broadcast:** Guarantees that every recipient delivers every message in the *same* sequence, combining reliable delivery, atomicity (a message is delivered to all correct processes or none), and a single total order. Because achieving this in the presence of failures requires detecting and working around coordinator failures, it is typically implemented using failure detectors and fallback election mechanisms. **Virtual synchrony** is a related technique for delivering messages consistently to a dynamically changing group of recipients, and total order broadcast can itself be framed as an instance of the consensus problem, with agreement here meaning agreement on the relative order of messages rather than on a single value.

**ZooKeeper Atomic Broadcast (ZAB).** ZAB is a leader-based atomic broadcast protocol, originally developed for a widely used coordination-service system, that provides both leader election and consensus on the ordering of write operations. It proceeds through three phases within each **epoch**: (1) **Discovery**, in which a prospective leader proposes a new epoch to the other nodes; (2) **Synchronization**, in which the elected leader synchronizes every follower's state by delivering any proposals from earlier epochs the follower may be missing; and (3) **Broadcast**, in which the leader accepts client requests, establishes their order, and broadcasts them to the followers.

### 7.9 State Machine Replication

**State Machine Replication (SMR)** models each replica as a deterministic state machine, all initialized to an identical starting state. As long as every replica applies exactly the same sequence of client requests, in exactly the same order, the replicas remain identical, providing fault tolerance, since any individual replica may fail independently without the service as a whole becoming unavailable or inconsistent. SMR therefore relies directly on a total order broadcast protocol to guarantee that every replica agrees on the request sequence, which is what ultimately guarantees the replicated state machine's correctness.

### 7.10 Theoretical Results

- **Two Generals' Problem:** A classical thought experiment demonstrating that two parties communicating over a channel with possible message loss cannot reach mutual, common knowledge of an agreed decision using a finite number of messages, and, by extension, that no protocol built purely on unreliable message exchange can guarantee agreement with absolute certainty.
- **FLP Impossibility:** The result, due to Fischer, Lynch, and Paterson, that in a fully asynchronous system, one with no bound on message delay, no deterministic algorithm can guarantee agreement, validity, and termination simultaneously if even a single process may crash. In practice, real consensus protocols such as Paxos and Raft sidestep this impossibility by relying on partial synchrony, meaning bounded but unknown message delay in the common case, and on timeouts, sacrificing guaranteed termination in the worst case in exchange for termination that holds with overwhelming likelihood in practice.

### 7.11 Recovering from Failures

The **distributed snapshot problem** addresses how a system can capture and later restore a consistent record of its global state as it existed at some point before a failure. The central difficulty is that, as established in Chapter 6, no node has direct access to any other node's notion of "the same instant," so nodes must coordinate explicitly to record both their individual local states and the state of the communication channels between them, together, in a way that yields a globally consistent picture. A captured snapshot is considered **consistent** if it corresponds to a state the system could actually have passed through, reachable from the state at the start of the snapshot procedure and capable of reaching the state at its end. The **Chandy–Lamport algorithm** is the standard technique for capturing such a consistent global snapshot in an asynchronous distributed system without halting the system during the process.

### 7.12 Replication and Consistency, Revisited

**Revisiting replication.** One of the central motivations for replication, beyond throughput, is fault tolerance itself. Single-leader replication, despite its simplicity, introduces a **single point of failure (SPOF)**: if the leader fails, write availability is compromised until failover completes. Multi-leader, or equivalently multi-primary, replication mitigates this by distributing both load and leadership responsibility across multiple nodes, often placed in physically distinct data centers, so that the nodes fail largely independently of one another, improving overall system reliability.

**Revisiting consistency models.** Finally, we re-analyze the achievable consistency models across all replication schemes under the complete set of real-world constraints, C1–C7. Under active failure scenarios and network partitions, achieving strong guarantees such as linearizability becomes substantially harder: doing so now generally requires the consensus protocols and total order broadcast mechanisms developed in this chapter, since maintaining a single, globally agreed sequence of operations in the presence of failures is precisely the consensus problem. Outside of active failure scenarios, the consistency results established in earlier chapters remain unchanged. Eventual consistency's own convergence promise is similarly conditional: it applies only to replicas that remain reachable, and only once a write has reached a durable quorum, since a write held only as a hint on a node that is lost permanently (Section 7.5) has nothing left to converge to.

### 7.13 Chapter Summary

Every mechanism developed in Chapters 1 through 6 was analyzed under A7's convenient fiction that nodes and networks simply do not fail. Constraint C7 removes that fiction and, in doing so, forces every one of those earlier mechanisms to be re-examined for what happens when a participant crashes, partially recovers, or becomes unreachable mid-operation. The problems this creates are correspondingly broad. Failures must first be detected without perfect information. The atomicity and isolation guarantees built for a single node in Chapter 1 must be extended across nodes that can now fail independently mid-transaction. The single-leader architectures of Chapter 4 need a way to replace a failed leader without ever allowing two nodes to believe themselves leader simultaneously. Most fundamentally, this chapter's theoretical results establish that no protocol can guarantee agreement with certainty over an unreliable channel, per the Two Generals' Problem, or guarantee termination deterministically in a fully asynchronous system with even one faulty process, per FLP impossibility, which means every practical solution must consciously trade away some idealized guarantee.

The chapter's mechanisms address these problems in a clear hierarchy. Probabilistic failure detectors replace naive fixed timeouts with a tunable trade-off between detection speed and false suspicion. Two- and Three-Phase Commit extend Chapter 1's atomicity across nodes, with 3PC's extra phase and quorum-based protocols narrowing, though not eliminating, 2PC's coordinator-blocking weakness. Fencing tokens extend Chapter 1's isolation guarantees safely across leases that can silently expire. Hinted handoff and leader or replica failover extend Chapter 4's replication architectures to survive real node loss. Because leader-election algorithms alone provide liveness but not the safety needed to prevent split-brain, consensus protocols such as Paxos and Raft, together with the total-order broadcast and state-machine-replication techniques built on top of them, are what ultimately let a replicated system remain both safe and available under real failures, with the Chandy–Lamport algorithm extending Chapter 6's causal-ordering machinery to let the system recover a globally consistent checkpoint after the fact.

The trade-offs are correspondingly sharper than in earlier chapters: blocking-but-simple coordination versus non-blocking-but-complex coordination; strict quorum correctness versus availability during a partition, as traded by sloppy quorums; and, per FLP, guaranteed safety versus guaranteed termination, which every practical protocol resolves by sacrificing termination guarantees in the worst case in exchange for termination that holds overwhelmingly often via partial synchrony and timeouts. As a matter of practical guidance, lightweight heartbeat-based detection and simple failover are adequate wherever brief post-failure inconsistency is tolerable; two-phase commit remains defensible for tightly coupled, low-latency, well-managed internal systems, while loosely coupled or large-scale systems increasingly avoid distributed transactions altogether in favor of asynchronous, idempotent workflows; and full consensus protocols, Raft in particular given its comparative simplicity, are the appropriate and often unavoidable choice for any component whose correctness depends on a single, safely agreed leader or state, such as coordination services, distributed locks, and metadata stores, even though they carry the highest implementation and operational cost of any mechanism in this chapter.

---
