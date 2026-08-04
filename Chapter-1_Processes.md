## Chapter 1 — Processes

```
A.pop()          # removes A1
C.append(C1)

A = [A7, A6, A5, A4, A3, A2]
C = [C1]
```

**Relaxing A1; introducing C1.**

> **C1.** Processes take a finite, variable amount of time to execute. Each process performs a sequence of operations that move the system from one state to another through a series of intermediate states, and individual processes may be paused, crashed, or rolled back.

### 1.1 Foundational Definitions

- **Process:** An independent execution instance of a program (a single program may itself spawn multiple processes), each with its own private memory space.
- **Thread:** The smallest unit of execution within a process. Threads belonging to the same process share that process's memory space and resources.
- **Transaction:** A logical unit of work composed of one or more database operations (reads, writes, updates, or deletes) intended to execute as a single unit.

Throughout this chapter and the next five, a "crash" refers to the software-level failure of a single process or transaction: an unhandled exception, a bug, or an explicit abort triggered by the application itself. The node's operating system, disk, and network remain available throughout; whole-machine and network failure are introduced only once A7 is relaxed, in Chapter 7.

### 1.2 ACID Properties

**ACID** is the standard acronym for four desirable properties of a database transaction:

- **Atomicity:** An all-or-nothing guarantee, meaning that either every operation in the transaction is applied or none are, so the system never ends up in a partially applied, corrupted state.
- **Consistency:** A guarantee that the database remains in a valid state before and after every transaction, preserving application-specific invariants. (This notion of consistency, an application-level correctness property, is distinct from the network-partition-era notion of "consistency" introduced in Chapter 4; the two share a name but not a definition.)
- **Isolation:** A guarantee that the observable effect of a set of concurrently executing transactions with overlapping data dependencies is equivalent to some serial, one-at-a-time execution of those same transactions.
- **Durability:** A guarantee that the effects of a committed transaction persist even in the presence of subsequent failures (failure scenarios are treated in depth in Chapter 7).

Given constraint C1, atomicity becomes the property of immediate concern: because a process can now be paused, crash, or roll back partway through a multi-step transaction, the system must guard against being left in an intermediate, inconsistent state.

### 1.3 Achieving Atomicity

On a single machine, atomicity is most commonly achieved through a **write-ahead log (WAL)**: a durable, append-only record of every change a transaction makes, written before that change is applied to the database itself, together with the transaction metadata recording whether the transaction ultimately committed. On recovery, the log allows the system to distinguish committed from uncommitted transactions, undoing (rolling back) the latter and completing (rolling forward) the former.

### 1.4 Achieving Isolation

Constraint C1 implies that multiple threads or transactions accessing the same resources may be interleaved arbitrarily, which can produce **concurrency anomalies**. The principal anomalies are:

- **Dirty Read:** A transaction reads a data item written by another transaction that has not yet committed and that may subsequently abort, in which case the value read was never part of any committed state.
- **Dirty Write:** A transaction overwrites a data item previously written by another, still-uncommitted transaction, so that a later abort or commit by either transaction can leave the database holding a mixture of the two transactions' writes.
- **Non-repeatable Read:** A transaction reads the same row twice within its lifetime and observes two different values because another, concurrently committed transaction modified the row in between.
- **Phantom Read:** A transaction re-executes a range query and observes a different set of rows than an earlier execution, because another transaction inserted or deleted rows matching the query predicate in the interim.
- **Lost Update:** Two transactions concurrently read the same data item, compute new values based on that read, and write back, with the second write silently overwriting the first, so that one of the two updates is lost.
- **Read Skew:** A transaction reads multiple, related data items and observes them in a combination that violates an application-level consistency constraint, because a concurrent transaction updated only a subset of those items between the reads.
- **Write Skew:** Two transactions concurrently read the same set of data items, observing the same values; each makes a decision based on that shared read and then writes to disjoint subsets of those items, leaving the database, taken as a whole, in a state that violates a consistency constraint that neither transaction violated individually.

### 1.5 Algorithms for Preventing Anomalies

- **Optimistic Concurrency Control (OCC):** Transactions execute concurrently without acquiring locks. At commit time, the system validates whether any conflicting update has occurred since the transaction began; if so, the transaction is aborted and may be retried. OCC performs well under low contention, but repeated retries under high contention can cause throughput collapse and starvation.
- **Pessimistic Concurrency Control (PCC):** Transactions acquire exclusive locks on the data items they touch before operating on them, forcing conflicting transactions into serial execution. Concurrent transactions must wait for a lock to be released or must time out. PCC is well suited to high-contention workloads, though lock-acquisition overhead can become significant.
- **Two-Phase Locking (2PL):** A classical pessimistic mechanism that guarantees conflict serializability. Shared (read) locks permit concurrent readers; exclusive (write) locks block all other readers and writers; a transaction holding a shared lock may upgrade it to exclusive if it needs to write. A 2PL transaction proceeds through a **growing phase**, in which it acquires locks, followed by a **shrinking phase**, in which it releases them, with no further lock acquisition permitted once release has begun. Production implementations use the strict variant, which holds exclusive locks until commit or abort and thereby also rules out cascading aborts. The principal drawback of 2PL is the possibility of deadlock, which must be detected and resolved through transaction aborts and retries. Two-phase locking prevents all of the anomalies listed above once it is extended with the predicate or index-range locks described next, which are what close the phantom case, but it typically does so at the cost of higher latency and reduced throughput under contention.
- **Predicate Locks and Index-Range Locks:** A predicate lock applies to every object matching a logical condition, including "phantom" rows that do not yet exist but would match the predicate if inserted, and is therefore the mechanism of choice for preventing phantom reads under 2PL. Index-range locks are a coarser, lower-overhead approximation of predicate locks that lock a contiguous range on an index rather than an arbitrary predicate, trading some additional (false-positive) blocking for substantially lower bookkeeping cost.
- **Multi-Version Concurrency Control (MVCC):** A general mechanism, not a single isolation level, that allows multiple versions of a data record, created by different concurrent transactions, to coexist so that readers never block writers and writers never block readers. Which isolation level results depends on when a transaction's snapshot is taken and how commit-time conflicts are checked: a snapshot taken once at transaction start and validated against concurrent writes yields **snapshot isolation**, the level most commonly built on MVCC; a fresh snapshot taken at the start of every statement instead yields Read Committed; and additional conflict-detection machinery layered on top of snapshot isolation can yield a fully serializable level. In every case, stale versions are periodically garbage-collected once no active transaction can still observe them.

### 1.6 Isolation Levels

Isolation levels formalize how much interleaving between concurrent transactions is permitted, trading anomaly prevention against performance:

- **Strict Serializability:** The strongest guarantee for database transactions: concurrent transactions appear to have executed one after another in a single sequence that additionally respects real (wall-clock) time, so that if transaction T1 commits before T2 begins, T1 must appear before T2 in the equivalent serial order.
- **Serializability:** A guarantee that the combined effect of a set of concurrently executing transactions is equivalent to *some* serial execution of those transactions, though not necessarily the order in which they actually ran. No anomaly listed above is possible under serializability.
- **Repeatable Read:** A guarantee that any data item read once by a transaction will not appear to change for the remainder of that transaction. (Of the anomalies above, only phantom reads remain possible.)
- **Snapshot Isolation:** A guarantee that a transaction reads from a consistent snapshot of the database taken at its start, and commits successfully only if no other transaction has since modified the same data. (Write skew remains possible; phantom reads are not, since every read is served from the transaction's own snapshot.)
- **Read Committed:** A guarantee that a transaction can only observe data written by transactions that have already committed. (Dirty reads and dirty writes are prevented; non-repeatable reads, phantom reads, lost updates, read skew, and write skew remain possible.)
- **Read Uncommitted:** The weakest isolation level: a transaction may observe uncommitted data written by another, still-in-flight transaction. (Only dirty writes are prevented; dirty reads and all other anomalies remain possible.)

These levels form a strength hierarchy:

```
Strict Serializability → Serializability → { Repeatable Read, Snapshot Isolation } → Read Committed → Read Uncommitted
```

Repeatable Read and Snapshot Isolation both sit strictly below Serializability and strictly above Read Committed, but are not directly comparable to one another in the general case: each permits a different residual anomaly, as noted above, so neither strictly subsumes the other. In practice this formal distinction is frequently invisible in vendor documentation. Many production databases label their snapshot-isolation implementation "Repeatable Read," and others use "Serializable" for the same underlying mechanism, despite neither matching the lock-based definitions above. A specific database's reported isolation-level name is therefore a vendor label, not a guarantee that its behavior matches the formal definition given here.

### 1.7 Chapter Summary

Under assumption A1, a system had no interior to speak of: every process was an instantaneous, infallible transition from one state to the next, so questions of interleaving, partial completion, or interference between concurrent operations simply did not arise. Constraint C1 dissolves that convenience by giving every process a finite, interruptible execution window, and with it come two distinct problems. A transaction may now be caught mid-flight by a crash, leaving the database in a corrupted, half-applied state, and two or more transactions may now interleave their now-nonzero execution windows, producing anomalies (dirty reads and writes, lost updates, read and write skew, and the rest) that a purely sequential system could never exhibit.

The chapter's mechanisms address these two problems along largely separate axes. The write-ahead log restores atomicity by making every state transition durable and recoverable before it is considered complete. The concurrency-control algorithms, optimistic, pessimistic, two-phase locking, and multi-version concurrency control, each offer a different way of bounding which anomalies interleaved transactions are permitted to exhibit, formalized as the isolation-level hierarchy running from Read Uncommitted up to Strict Serializability. The trade-off running through all of these mechanisms is correctness versus performance: locking-based approaches such as 2PL buy strong guarantees at the cost of blocking, deadlock risk, and reduced throughput under contention, while optimistic and multi-version approaches buy higher concurrency by risking wasted work on conflict or by paying a bookkeeping cost in retained versions. In practice, the right point on this spectrum is set by the workload's contention profile and the cost of an anomaly to the application: low-contention, read-heavy workloads favor OCC or Snapshot Isolation, high-contention workloads where correctness is non-negotiable favor 2PL or fully serializable execution, and most general-purpose systems settle for Read Committed or Snapshot Isolation as a pragmatic middle ground. There is no isolation level that is simultaneously the fastest and the safest, only a defensible choice given a specific workload's contention and correctness requirements.

---
