## Chapter 3 — Data

```
A.pop()          # removes A3
C.append(C3)

A = [A7, A6, A5, A4]
C = [C1, C2, C3]
```

**Relaxing A3; introducing C3.**

> **C3.** Data volume can be arbitrarily large, far exceeding the storage and memory capacity of a single machine.

Vertically scaling a database, that is, adding more CPU, memory, or disk to a single machine, is bounded, can be prohibitively expensive at the high end, and increases resource contention (particularly lock contention) as more work is packed onto one node. **Horizontal scaling**, distributing data and load across many machines, is therefore necessary to manage large data volumes.

### 3.1 Partitioning

**Partitioning** is the process of splitting a large dataset into smaller, disjoint subsets called **partitions** (or **shards**), such that every piece of data belongs to exactly one partition. The goal is to spread both data and query load evenly across nodes, enabling parallelism and improving aggregate throughput.

- **Key Range-Based Partitioning:** Each partition is assigned a contiguous range of keys, with partition boundaries defining which keys fall into which partition. Because keys are stored in sorted order, range queries within a partition are efficient. The drawback is that skewed access patterns, for instance a hot range of sequential keys, can create **hotspots**, where some partitions receive disproportionately more traffic than others.
- **Key Hash-Based Partitioning:** To avoid hotspots, this scheme assigns data to a partition based on a hash of the key rather than the key's raw value, producing a much more uniform load distribution. The trade-off is that range queries become inefficient, since consecutive keys are no longer co-located.
- **Consistent Hashing:** Naïve hash-based partitioning (e.g., `hash(key) mod n`) causes almost all keys to be remapped whenever the number of partitions `n` changes, triggering massive, disruptive data movement on every scaling event. Consistent hashing avoids this by mapping both nodes and keys onto positions on a hash ring, with each key assigned to the first node encountered when traveling clockwise from the key's position, so that each node owns the contiguous ring segment terminating at its own position. When a node joins or leaves, only the keys in the segment adjacent to its ring position change ownership, rather than the entire dataset, which enables low-disruption rebalancing in dynamic clusters. A single ring position per node leaves segment sizes uneven and confines every join or departure to a single neighbor, since a joining node draws its whole range from its successor and a departing node hands its whole range to it. **Virtual nodes** address both by giving each physical node many positions on the ring, so that its ownership is split into many small segments interleaved with those of its peers: segment sizes average out, and the transfer on a join or departure is spread across many nodes rather than concentrated on one.

### 3.2 Request Routing

Once data is partitioned across nodes, incoming requests must be routed to the partition, and therefore the node, that owns the relevant key. Three common mechanisms exist:

- **Client-Side Awareness:** Clients are themselves aware of the partitioning scheme and route requests directly to the appropriate node.
- **Routing Tier / Load Balancer:** A dedicated, partition-aware proxy layer routes each incoming request to the correct node on the client's behalf.
- **Forwarding Between Nodes:** Any node may receive a request and, if it does not own the relevant partition, forward the request internally to the node that does.

Whenever partition-to-node assignments change, whether due to scaling, rebalancing, or failover, all of these routing mechanisms must be kept consistent with the current assignment. This typically requires a consensus protocol (discussed in Chapter 7) to keep the routing metadata synchronized across the cluster.

### 3.3 Scaling and Rebalancing

To scale with growing data volume or throughput, a database's data is divided into many partitions distributed across the available nodes. Keeping many small partitions per node, rather than one large partition per node, allows for flexible rebalancing: as data grows or nodes are added, partitions can be split or reassigned without moving the entire dataset at once. **Dynamic partitioning** schemes automatically adjust the number of partitions in proportion to data volume and cluster size, rather than fixing the partition count at deployment time.

### 3.4 Secondary Indexing

A **secondary index** is an index built on a non-primary-key column, used to accelerate queries that filter or search on that column's values. In a partitioned system, a secondary index can itself be partitioned in one of two ways:

- **Document-Based Partitioning (Local Index):** Each partition maintains its own secondary index, covering only the data stored on that partition. Because a given secondary-key value may match documents on any subset of partitions, satisfying a query generally requires a **scatter-gather** operation across all partitions, which is comparatively expensive for reads. Writes and updates, by contrast, are efficient, since the relevant secondary index entry lives on the same node as the underlying data.
- **Term-Based Partitioning (Global Index):** A single logical secondary index is itself partitioned, typically by the indexed term or a hash of it, independently of how the underlying data is partitioned. Reads are faster, since a query can go directly to the partition holding the relevant index entries without a scatter-gather. Writes are slower and more complex, since a single write to the underlying data may need to update index entries that live on a different node, and possibly a different partition, from the data itself, and this update is often performed asynchronously.

### 3.5 Chapter Summary

As long as A3 held, every partitioning question was moot, since a single machine could always hold the entire dataset, and Chapter 2's storage structures could be optimized purely for access pattern with no thought given to where the data lived. Constraint C3 forces data beyond the capacity of any one machine, which converts storage from a local optimization problem into a distributed one: data must now be split into partitions spread across many nodes, clients must be able to find the partition responsible for a given key, and both the partition count and the partition-to-node assignment must be able to change as data and cluster size grow, all without incurring disruptive, wholesale data movement.

The chapter's mechanisms map directly onto these sub-problems. Range- and hash-based partitioning make opposite trade-offs between range-query efficiency and load uniformity, with range partitioning risking hotspots that hash partitioning avoids at the cost of destroying key locality. Consistent hashing then solves the specific problem of rebalancing, ensuring that adding or removing a node moves only the data in the ring segments adjacent to that node's positions rather than the entire dataset. The three request-routing mechanisms solve the complementary problem of keeping clients pointed at the correct, possibly-changing partition assignment. The two secondary-indexing strategies extend partitioning to non-primary-key queries, trading read efficiency for write efficiency depending on whether the index is kept local to each partition or maintained as its own globally partitioned structure.

The recurring trade-off is uniform load distribution versus locality-dependent efficiency: hash-based schemes scale predictably but sacrifice range queries and, for global secondary indexes, write simplicity, while range-based and local-index schemes preserve locality and write efficiency at the risk of skew and expensive scatter-gather reads. Systems with roughly uniform, high-cardinality access patterns and a need for elastic, low-disruption rebalancing are best served by hash-based or consistent-hashing partitioning; systems whose queries are dominated by ordered range scans are better served by range partitioning, provided the key space itself is engineered to avoid hotspots; and the choice between local and global secondary indexes should track whether the workload is read-dominated or write-dominated, since no single partitioning or indexing strategy is uniformly superior across access patterns.

---
