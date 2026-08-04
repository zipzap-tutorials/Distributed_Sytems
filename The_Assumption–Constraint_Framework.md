## The Assumption–Constraint Framework

To make the relaxation process precise, we track the state of the model using two ordered collections: a stack of remaining **Assumptions**, denoted `A`, and a list of accumulated real-world **Constraints**, denoted `C`. The assumption collection follows the semantics of a standard last-in-first-out stack, while the constraint collection is append-only: at each stage, exactly one assumption is popped from `A` and the corresponding constraint is appended to `C`. The system described in chapter *k* therefore operates under all assumptions *not yet popped* and all constraints *introduced so far*. Every conclusion drawn in a given chapter is implicitly qualified by the current contents of `A` and `C`, and is subject to revision once further assumptions are relaxed. This is why several results are explicitly revisited in later chapters as additional constraints are introduced: a conclusion that holds given a synchronized global clock, for instance, may no longer hold once clock synchronization is relaxed, a point made explicit in Chapters 4 through 6.

Initially, no constraints have been introduced, and all seven idealizing assumptions are in force:

```
A = [A7, A6, A5, A4, A3, A2, A1]
C = []
```

The seven idealizing assumptions are:

- **A1 — Process assumptions.** A single, infinitely powerful machine with unlimited processing cores is available. All processes represent instantaneous state transitions, and no process ever fails.
- **A2 — Storage assumptions.** All memory and disk operations complete in infinitesimal time, independent of data size or memory footprint. Both memory and disk are non-volatile, and all computational resources incur equal marginal cost with respect to both time and money.
- **A3 — Data-volume assumptions.** The total data volume never exceeds the storage capacity of a single machine.
- **A4 — Throughput assumptions.** The system's read and write throughput never exceeds the computational capacity of a single machine.
- **A5 — Network assumptions.** Network latency is infinitesimally small, bandwidth is infinite, and message-transmission latency is entirely independent of the physical distance between nodes.
- **A6 — Clock assumptions.** All machines share a single, universal notion of time, synchronized to one perfectly accurate global clock.
- **A7 — Failure assumptions.** No failures of any kind occur: no power outages, node crashes, or network partitions. (This concerns hardware, OS, and network failure; the software-level failure of an individual process is addressed separately in §1.1.)

Each of these seven assumptions is relaxed in turn over the seven chapters that follow, in the order A1 → A7. Each relaxation introduces exactly one corresponding constraint (C1 through C7), and each chapter examines the complexities and design trade-offs that the newly introduced constraint reveals.

---
