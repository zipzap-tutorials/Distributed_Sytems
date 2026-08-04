# Distributed_Systems
Distributed Systems Overview using A Stacked Assumption-Relaxation and Constraint-Introduction Framework

[Arpit Rathi](https://www.linkedin.com/in/arpitrathi-iitb/)

---

## Abstract

Distributed systems are difficult to reason about primarily because they force several interacting concerns (concurrency, storage, data volume, throughput, network behavior, time, and failure) to be addressed simultaneously. This paper presents a framework for reasoning about distributed systems by starting from an idealized, single-machine baseline in which computation is deterministic, resources are unbounded, and failures never occur. From this baseline, we systematically relax one simplifying assumption at a time, replace it with the corresponding real-world constraint, and examine the mechanisms, trade-offs, and theoretical results that the relaxation makes necessary. The resulting seven-stage progression (processes, storage, data volume, throughput, network, clocks, and failures) reconstructs the major results of distributed systems theory and engineering practice, from ACID transactions and B-trees to CAP/PACELC, replication, vector clocks, consensus, and state machine replication, as consequences of specific, named assumptions being dropped, rather than as an unordered catalogue of mechanisms. The goal is pedagogical: to give practitioners and researchers a single mental model, an "assumption stack," for navigating the field's breadth while retaining conceptual coherence.

---

## Table of Contents

- [Introduction](https://github.com/zipzap-tutorials/Distributed_Sytems/blob/main/Introduction.md)
- [The Assumption–Constraint Framework](https://github.com/zipzap-tutorials/Distributed_Sytems/blob/main/The_Assumption%E2%80%93Constraint_Framework.md)

- Video Timeline (Course Introduction):
  - [0:00](https://www.youtube.com/watch?v=CrX9meMtXS0&list=PLEbp4D3leK9g&index=1) Course Introduction
  - [3:13](https://www.youtube.com/watch?v=CrX9meMtXS0&list=PLEbp4D3leK9g&index=1&t=193s) Graphical Legend System
  - [5:05](https://www.youtube.com/watch?v=CrX9meMtXS0&list=PLEbp4D3leK9g&index=1&t=305s) Assumption-Constraint Framework
 
- [Chapter 1 — Processes](https://github.com/zipzap-tutorials/Distributed_Sytems/blob/main/Chapter-1_Processes.md)
  - 1.1 Foundational Definitions
  - 1.2 ACID Properties
  - 1.3 Achieving Atomicity
  - 1.4 Achieving Isolation
  - 1.5 Algorithms for Preventing Anomalies
  - 1.6 Isolation Levels
  - 1.7 Chapter Summary

- Video Timeline (Chapter 1):
  - [0:00](https://www.youtube.com/watch?v=YuOsiDKzoPo&list=PLEbp4D3leK9g&index=2) Relaxing Processes-related Assumptions
  - [2:12](https://www.youtube.com/watch?v=YuOsiDKzoPo&list=PLEbp4D3leK9g&index=2&t=132s) Process, Thread, and Transaction 
  - [3:26](https://www.youtube.com/watch?v=YuOsiDKzoPo&list=PLEbp4D3leK9g&index=2&t=206s) ACID
  - [5:01](https://www.youtube.com/watch?v=YuOsiDKzoPo&list=PLEbp4D3leK9g&index=2&t=301s) Write-ahead log
  - [6:30](https://www.youtube.com/watch?v=YuOsiDKzoPo&list=PLEbp4D3leK9g&index=2&t=390s) Concurrency Anomalies (frame-1/2)
  - [7:59](https://www.youtube.com/watch?v=YuOsiDKzoPo&list=PLEbp4D3leK9g&index=2&t=479s) Concurrency Anomalies (frame-2/2)
  - [9:39](https://www.youtube.com/watch?v=YuOsiDKzoPo&list=PLEbp4D3leK9g&index=2&t=579s) OCC vs PCC
  - [11:01](https://www.youtube.com/watch?v=YuOsiDKzoPo&list=PLEbp4D3leK9g&index=2&t=661s) 2PL
  - [13:19](https://www.youtube.com/watch?v=YuOsiDKzoPo&list=PLEbp4D3leK9g&index=2&t=799s) MVCC
  - [14:58](https://www.youtube.com/watch?v=YuOsiDKzoPo&list=PLEbp4D3leK9g&index=2&t=898s) Isolation Levels
  - [18:14](https://www.youtube.com/watch?v=YuOsiDKzoPo&list=PLEbp4D3leK9g&index=2&t=1094s) Chapter Summary

- [Chapter 2 — Storage](https://github.com/zipzap-tutorials/Distributed_Sytems/blob/main/Chapter-2_Storage.md)
  - 2.1 Data Structures
  - 2.2 Data Models
  - 2.3 Specialized Databases
  - 2.4 Caching Strategies
  - 2.5 Chapter Summary

- Video Timeline (Chapter 2):
  - [0:00](https://www.youtube.com/watch?v=8sVi8BzgaXY&list=PLEbp4D3leK9g&index=3) Relaxing Storage-related Assumptions
  - [1:45](https://www.youtube.com/watch?v=8sVi8BzgaXY&list=PLEbp4D3leK9g&index=3&t=105s) Hash Index
  - [2:59](https://www.youtube.com/watch?v=8sVi8BzgaXY&list=PLEbp4D3leK9g&index=3&t=179s) B-Trees & B+ Trees
  - [4:39](https://www.youtube.com/watch?v=8sVi8BzgaXY&list=PLEbp4D3leK9g&index=3&t=279s) LSM Tree
  - [7:08](https://www.youtube.com/watch?v=8sVi8BzgaXY&list=PLEbp4D3leK9g&index=3&t=428s) Data Structures' Trade-off Analysis
  - [8:26](https://www.youtube.com/watch?v=8sVi8BzgaXY&list=PLEbp4D3leK9g&index=3&t=506s) Data Models
  - [12:02](https://www.youtube.com/watch?v=8sVi8BzgaXY&list=PLEbp4D3leK9g&index=3&t=722s) Specialized Databases
  - [13:58](https://www.youtube.com/watch?v=8sVi8BzgaXY&list=PLEbp4D3leK9g&index=3&t=838s) Caching Mechanisms
  - [16:39](https://www.youtube.com/watch?v=8sVi8BzgaXY&list=PLEbp4D3leK9g&index=3&t=999s) Chapter Summary

- [Chapter 3 — Data](https://github.com/zipzap-tutorials/Distributed_Sytems/blob/main/Chapter-3_Data.md)
  - 3.1 Partitioning
  - 3.2 Request Routing
  - 3.3 Scaling and Rebalancing
  - 3.4 Secondary Indexing
  - 3.5 Chapter Summary
 
- Video Timeline (Chapter 3):
  - [0:00](https://www.youtube.com/watch?v=TZm4z8dbf8s&list=PLEbp4D3leK9g&index=4) Relaxing Data-related Assumptions
  - [1:40](https://www.youtube.com/watch?v=TZm4z8dbf8s&list=PLEbp4D3leK9g&index=4&t=100s) Range vs Hash Partitioning
  - [3:20](https://www.youtube.com/watch?v=TZm4z8dbf8s&list=PLEbp4D3leK9g&index=4&t=200s) Consistent Hashing
  - [5:23](https://www.youtube.com/watch?v=TZm4z8dbf8s&list=PLEbp4D3leK9g&index=4&t=323s) Request Routing
  - [7:03](https://www.youtube.com/watch?v=TZm4z8dbf8s&list=PLEbp4D3leK9g&index=4&t=423s) Scaling & Rebalancing Partitions
  - [8:29](https://www.youtube.com/watch?v=TZm4z8dbf8s&list=PLEbp4D3leK9g&index=4&t=509s) Secondary Indexes
  - [10:16](https://www.youtube.com/watch?v=TZm4z8dbf8s&list=PLEbp4D3leK9g&index=4&t=616s) Chapter Summary

- [Chapter 4 — Throughput](https://github.com/zipzap-tutorials/Distributed_Sytems/blob/main/Chapter-4_Throughput.md)
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

- Video Timeline (Chapter 4):
  - [0:00](https://www.youtube.com/watch?v=OrNwHjzeR_w&list=PLEbp4D3leK9g&index=5) Relaxing Throughput-related Assumptions
  - [1:59](https://www.youtube.com/watch?v=OrNwHjzeR_w&list=PLEbp4D3leK9g&index=5&t=119s) CAP & PACELC
  - [5:37](https://www.youtube.com/watch?v=OrNwHjzeR_w&list=PLEbp4D3leK9g&index=5&t=337s) Single-leader Replication
  - [6:29](https://www.youtube.com/watch?v=OrNwHjzeR_w&list=PLEbp4D3leK9g&index=5&t=389s) Multi-leader Replication
  - [8:51](https://www.youtube.com/watch?v=OrNwHjzeR_w&list=PLEbp4D3leK9g&index=5&t=531s) CRDTs
  - [10:33](https://www.youtube.com/watch?v=OrNwHjzeR_w&list=PLEbp4D3leK9g&index=5&t=633s) Replication Topologies
  - [11:35](https://www.youtube.com/watch?v=OrNwHjzeR_w&list=PLEbp4D3leK9g&index=5&t=695s) Leaderless Replication
  - [13:17](https://www.youtube.com/watch?v=OrNwHjzeR_w&list=PLEbp4D3leK9g&index=5&t=797s) Adding Replicas
  - [14:19](https://www.youtube.com/watch?v=OrNwHjzeR_w&list=PLEbp4D3leK9g&index=5&t=859s) Replication Models
  - [15:41](https://www.youtube.com/watch?v=OrNwHjzeR_w&list=PLEbp4D3leK9g&index=5&t=941s) Data-centric Consistency Levels
  - [18:42](https://www.youtube.com/watch?v=OrNwHjzeR_w&list=PLEbp4D3leK9g&index=5&t=1122s) Client-centric Consistency Levels
  - [20:20](https://www.youtube.com/watch?v=OrNwHjzeR_w&list=PLEbp4D3leK9g&index=5&t=1220s) Replication Models x Consistency Levels
  - [24:36](https://www.youtube.com/watch?v=OrNwHjzeR_w&list=PLEbp4D3leK9g&index=5&t=1476s) Chapter Summary

- [Chapter 5 — Network](https://github.com/zipzap-tutorials/Distributed_Sytems/blob/main/Chapter-5_Network.md)
  - 5.1 Communication Protocols
  - 5.2 Information Dissemination Mechanisms
  - 5.3 Anti-Entropy Mechanisms
  - 5.4 Implications for Replication
  - 5.5 Chapter Summary

- Video Timeline (Chapter 5):
  - [0:00](https://www.youtube.com/watch?v=4HVhyft5wW8&list=PLEbp4D3leK9g&index=6) Relaxing Network-related Assumptions
  - [2:22](https://www.youtube.com/watch?v=4HVhyft5wW8&list=PLEbp4D3leK9g&index=6&t=142s) Communication Protocols
  - [5:08](https://www.youtube.com/watch?v=4HVhyft5wW8&list=PLEbp4D3leK9g&index=6&t=308s) Gossip Protocols
  - [6:24](https://www.youtube.com/watch?v=4HVhyft5wW8&list=PLEbp4D3leK9g&index=6&t=384s) Read-repair & Merkle-trees
  - [9:07](https://www.youtube.com/watch?v=4HVhyft5wW8&list=PLEbp4D3leK9g&index=6&t=547s) Geo-distributed DBs & Replication Consistency
  - [11:14](https://www.youtube.com/watch?v=4HVhyft5wW8&list=PLEbp4D3leK9g&index=6&t=674s) Chapter Summary

- [Chapter 6 — Clocks](https://github.com/zipzap-tutorials/Distributed_Sytems/blob/main/Chapter-6_Clocks.md)
  - 6.1 Synchronized Clocks
  - 6.2 Event Ordering
  - 6.3 Causal Relationships
  - 6.4 Logical Clocks
  - 6.5 Managing Data Consistency
  - 6.6 Revisiting Replication
  - 6.7 Chapter Summary

- Video Timeline (Chapter 6):
  - [0:00](https://www.youtube.com/watch?v=0LMmokx1tWI&list=PLEbp4D3leK9g&index=7) Relaxing Clocks-related Assumptions
  - [1:57](https://www.youtube.com/watch?v=0LMmokx1tWI&list=PLEbp4D3leK9g&index=7&t=117s) Tackling Clock Skew
  - [4:09](https://www.youtube.com/watch?v=0LMmokx1tWI&list=PLEbp4D3leK9g&index=7&t=249s) Event Ordering & Causality
  - [6:36](https://www.youtube.com/watch?v=0LMmokx1tWI&list=PLEbp4D3leK9g&index=7&t=396s) Lamport Clocks
  - [8:29](https://www.youtube.com/watch?v=0LMmokx1tWI&list=PLEbp4D3leK9g&index=7&t=509s) Vector Clocks
  - [10:04](https://www.youtube.com/watch?v=0LMmokx1tWI&list=PLEbp4D3leK9g&index=7&t=604s) Version Vectors
  - [11:47](https://www.youtube.com/watch?v=0LMmokx1tWI&list=PLEbp4D3leK9g&index=7&t=707s) Replication Consistency under Clock Skew 
  - [13:45](https://www.youtube.com/watch?v=0LMmokx1tWI&list=PLEbp4D3leK9g&index=7&t=825s) Chapter Summary

- [Chapter 7 — Failures](https://github.com/zipzap-tutorials/Distributed_Sytems/blob/main/Chapter-7_Failures.md)
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

- Video Timeline (Chapter 7):
  - [0:00](https://www.youtube.com/watch?v=pSaEpcQBTHk&list=PLEbp4D3leK9g&index=8) Relaxing Failures-related Assumptions
  - [1:31](https://www.youtube.com/watch?v=pSaEpcQBTHk&list=PLEbp4D3leK9g&index=8&t=91s) Key Definitions
  - [3:25](https://www.youtube.com/watch?v=pSaEpcQBTHk&list=PLEbp4D3leK9g&index=8&t=205s) Failure Detection
  - [5:28](https://www.youtube.com/watch?v=pSaEpcQBTHk&list=PLEbp4D3leK9g&index=8&t=328s) 2PC
  - [7:04](https://www.youtube.com/watch?v=pSaEpcQBTHk&list=PLEbp4D3leK9g&index=8&t=424s) 3PC
  - [8:28](https://www.youtube.com/watch?v=pSaEpcQBTHk&list=PLEbp4D3leK9g&index=8&t=508s) Distributed Locking
  - [10:28](https://www.youtube.com/watch?v=pSaEpcQBTHk&list=PLEbp4D3leK9g&index=8&t=628s) Handling Node Failures
  - [12:56](https://www.youtube.com/watch?v=pSaEpcQBTHk&list=PLEbp4D3leK9g&index=8&t=776s) Leader Election Algorithms
  - [15:32](https://www.youtube.com/watch?v=pSaEpcQBTHk&list=PLEbp4D3leK9g&index=8&t=932s) Consensus: Paxos & Raft
  - [18:41](https://www.youtube.com/watch?v=pSaEpcQBTHk&list=PLEbp4D3leK9g&index=8&t=1121s) Broadcast Ordering & State Machine Replication
  - [22:04](https://www.youtube.com/watch?v=pSaEpcQBTHk&list=PLEbp4D3leK9g&index=8&t=1324s) Two Generals Problem & FLP Impossibility
  - [24:07](https://www.youtube.com/watch?v=pSaEpcQBTHk&list=PLEbp4D3leK9g&index=8&t=1447s) Distributed Snapshots
  - [25:47](https://www.youtube.com/watch?v=pSaEpcQBTHk&list=PLEbp4D3leK9g&index=8&t=1547s) Replication Models x Consistency Levels (revisited)
  - [27:53](https://www.youtube.com/watch?v=pSaEpcQBTHk&list=PLEbp4D3leK9g&index=8&t=1673s) Chapter Summary

- [Summary](https://github.com/zipzap-tutorials/Distributed_Sytems/blob/main/Summary.md)
- [References](https://github.com/zipzap-tutorials/Distributed_Sytems/blob/main/References.md)

- Video Timeline (Course Summary):
  - [0:00](https://www.youtube.com/watch?v=RF6mHix0UwE&list=PLEbp4D3leK9g&index=9) Course Summary
  - [3:40](https://www.youtube.com/watch?v=RF6mHix0UwE&list=PLEbp4D3leK9g&index=9&t=220s) References & Recommendations

---
