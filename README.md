# Distributed_Sytems
Distributed Systems Overview using A Stacked Assumption-Relaxation and Constraint-Introduction Framework

**Arpit Rathi**

---

## Abstract

Distributed systems are difficult to reason about primarily because they force several interacting concerns (concurrency, storage, data volume, throughput, network behavior, time, and failure) to be addressed simultaneously. This paper presents a framework for reasoning about distributed systems by starting from an idealized, single-machine baseline in which computation is deterministic, resources are unbounded, and failures never occur. From this baseline, we systematically relax one simplifying assumption at a time, replace it with the corresponding real-world constraint, and examine the mechanisms, trade-offs, and theoretical results that the relaxation makes necessary. The resulting seven-stage progression (processes, storage, data volume, throughput, network, clocks, and failures) reconstructs the major results of distributed systems theory and engineering practice, from ACID transactions and B-trees to CAP/PACELC, replication, vector clocks, consensus, and state machine replication, as consequences of specific, named assumptions being dropped, rather than as an unordered catalogue of mechanisms. The goal is pedagogical: to give practitioners and researchers a single mental model, an "assumption stack," for navigating the field's breadth while retaining conceptual coherence.

---

## Table of Contents

- Introduction
- The Assumption–Constraint Framework
Video Timeline:
[0:00](https://www.youtube.com/watch?v=CrX9meMtXS0&list=PLEbp4D3leK9g&index=1) Course Introduction
[3:13](https://www.youtube.com/watch?v=CrX9meMtXS0&list=PLEbp4D3leK9g&index=1&t=193s) Graphical Legend System
[5:05](https://www.youtube.com/watch?v=CrX9meMtXS0&list=PLEbp4D3leK9g&index=1&t=305s) Assumption-Constraint Framework
 
- Chapter 1 — Processes
  - 1.1 Foundational Definitions
  - 1.2 ACID Properties
  - 1.3 Achieving Atomicity
  - 1.4 Achieving Isolation
  - 1.5 Algorithms for Preventing Anomalies
  - 1.6 Isolation Levels
  - 1.7 Chapter Summary

- Chapter 2 — Storage
  - 2.1 Data Structures
  - 2.2 Data Models
  - 2.3 Specialized Databases
  - 2.4 Caching Strategies
  - 2.5 Chapter Summary

- Chapter 3 — Data
  - 3.1 Partitioning
  - 3.2 Request Routing
  - 3.3 Scaling and Rebalancing
  - 3.4 Secondary Indexing
  - 3.5 Chapter Summary

- Chapter 4 — Throughput
  - 4.1 CAP and PACELC Theorems
  - 4.2 Single-Leader Replication
  - 4.3 Multi-Leader Replication
  - 4.4 Leaderless Replication
  - 4.5 Scaling Replicas
  - 4.6 Replication Modes
  - 4.7 Data-Centric Consistency Models
  - 4.8 Client-Centric Consistency Models
  - 4.9 Analyzing Replication Schemes and Consistency
  - 4.10 Chapter Summary

- Chapter 5 — Network
  - 5.1 Communication Protocols
  - 5.2 Information Dissemination Mechanisms
  - 5.3 Anti-Entropy Mechanisms
  - 5.4 Implications for Replication
  - 5.5 Chapter Summary

- Chapter 6 — Clocks
  - 6.1 Synchronized Clocks
  - 6.2 Event Ordering
  - 6.3 Causal Relationships
  - 6.4 Logical Clocks
  - 6.5 Managing Data Consistency
  - 6.6 Revisiting Replication
  - 6.7 Chapter Summary

- Chapter 7 — Failures
  - 7.1 Foundational Definitions
  - 7.2 Failure Detection
  - 7.3 Achieving Atomicity in Distributed Transactions
  - 7.4 Ensuring Isolation in Distributed Transactions
  - 7.5 Failure Handling Mechanisms
  - 7.6 Leader Election in Distributed Systems
  - 7.7 Consensus Algorithms
  - 7.8 Broadcast Protocols
  - 7.9 State Machine Replication
  - 7.10 Theoretical Results
  - 7.11 Recovering from Failures
  - 7.12 Replication and Consistency, Revisited
  - 7.13 Chapter Summary

- Summary
- References
Video Timeline:
[0:00](https://www.youtube.com/watch?v=RF6mHix0UwE&list=PLEbp4D3leK9g&index=9) Course Summary
[3:40](https://www.youtube.com/watch?v=RF6mHix0UwE&list=PLEbp4D3leK9g&index=9&t=220s) References & Recommendations

---
