## Chapter 2 — Storage

```
A.pop()          # removes A2
C.append(C2)

A = [A7, A6, A5, A4, A3]
C = [C1, C2]
```

**Relaxing A2; introducing C2.**

> **C2.** Memory and disk access incur different costs in both time and money. Memory is significantly more expensive and much faster to access than disk, and on disk, sequential access is substantially cheaper than random access. Furthermore, even absent a power failure (still excluded by A7 at this stage), memory remains volatile and can lose state due to software errors.

### 2.1 Data Structures

- **Hash Index:** An in-memory hash map whose keys are the unique identifiers of records and whose values are the corresponding byte offsets of that record on disk. The underlying on-disk structure is an append-only log: every write is appended sequentially to a **segment file**. This design supports very fast point lookups and high write throughput, but does not support efficient range queries, and the in-memory hash map must be small enough to fit in main memory, which bounds the number of distinct keys the structure can efficiently support.
- **B-Tree:** A balanced search tree with a much higher fanout and correspondingly lower height than a balanced binary search tree, achieved by storing multiple sorted keys per node. Nodes are stored as fixed-size, contiguous blocks on disk, allowing several keys to be retrieved in a single disk seek, which makes the B-tree well suited as an on-disk structure. B-trees support both point and range queries efficiently.
- **B+ Tree:** An optimization of the B-tree, and the structure most commonly used in relational database engines. Internal nodes store only separator keys, used purely for routing, while all key–value pairs are stored in the leaf nodes; this increases fanout and reduces tree height relative to a plain B-tree. Leaf nodes additionally maintain sorted key–value pairs and are linked together for efficient sequential range scans without re-traversing the tree.
- **LSM Tree (Log-Structured Merge Tree):** A disk-resident structure used extensively in modern NoSQL databases, built from several cooperating components that are periodically merged and compacted in the background:
  - **Memtable:** An in-memory sorted structure, typically a balanced binary search tree or a skip list, that buffers incoming writes before they are flushed to disk; reads consult it directly for any key not yet flushed to an SSTable.
  - **SSTable (Sorted String Table):** An on-disk structure comprising a collection of segment files, each storing a sorted run of key–value pairs together with a sparse index for efficient lookup.
  - **Write-Ahead Log (WAL):** A disk-resident, append-only log that records every modification applied to the memtable before that modification is considered durable, providing the basis for crash and transaction recovery. Data buffered in memory is periodically flushed (checkpointed) to disk, after which the corresponding WAL entries can be discarded.
  - **Bloom Filter:** A probabilistic, space-efficient structure that determines, with no false negatives though with a tunable rate of false positives, whether a key is *definitely absent* from a given SSTable, avoiding unnecessary disk reads against segment files that cannot contain the key.

  A defining characteristic of the LSM-tree approach is that segment files are written once, sequentially, and are thereafter treated as immutable; since encoded key–value entries also vary in size, true in-place updates are impractical. Updates are instead appended as new entries, deletions are recorded as **tombstones**, and superseded or tombstoned entries are physically discarded during background compaction.

### 2.2 Data Models

Different data models are optimized for different data properties: object size, the relationships between objects (one-to-one, one-to-many, many-to-one, many-to-many), and how rigidly a schema is enforced.

**In-Memory DBMS.** These systems store data exclusively in RAM and serve reads and writes with very low latency, but the data they hold is not durable by default, and total capacity is bounded by the cost and volatility of RAM.

**Disk-Based DBMS.**

*Relational databases* store structured data that conforms to a fixed schema, provide ACID guarantees natively, and are queried using SQL. Data is organized into tables of tuples, each with a primary key and additional attributes, typically stored in row-oriented layout. Normalization eliminates data redundancy by decomposing data about different entities into separate tables linked by foreign keys. A well-known drawback is the **impedance mismatch** between the relational model and the in-memory object structures used by application code, which typically requires an explicit mapping layer, either an ORM or hand-written relational-algebra-based queries, to bridge the two representations.

*NoSQL databases* are a family of more flexible, historically non-ACID-compliant data models optimized for semi-structured or unstructured data, horizontal scalability, high availability, and low latency, generally at the cost of strong consistency guarantees. Their simpler data models also avoid the impedance-mismatch problem and typically avoid the need for expensive join operations. Common categories include:

- **Key-Value Store:** Data is stored as key–value pairs, where the key is a unique identifier and the value may be an object of arbitrary complexity. Well suited to session-oriented workloads where access is always by primary key.
- **Document Database:** Stores self-describing documents, that is, hierarchical, tree-structured records whose shape may vary from document to document. Well suited to unstructured data or complex, nested structures naturally stored as a single unit.
- **Columnar Database:** Stores data on disk column-by-column rather than row-by-row, which makes retrieving a subset of fields, as in analytical aggregation queries, significantly more efficient. Best suited to read-heavy, analytical workloads.
- **Wide-Column Database:** Stores semi-structured data with a flexible per-row schema, organizing data into rows grouped by column families. This layout is generally better suited to write-heavy workloads than the columnar model above; note that "columnar" and "wide-column" describe two distinct storage strategies despite the similarity in name.
- **Graph Database:** Uses a graph structure for storage, where nodes represent entities and edges represent relationships between them, each carrying its own set of attributes. Well suited to data with a high, irregular degree of interconnection, and to queries such as shortest path and reachability that are awkward to express relationally.

### 2.3 Specialized Databases

- **Time-Series Database:** Optimized for storing and querying time-stamped data; designed for high-throughput ingestion, uses compression algorithms tailored to time-series data patterns, and is optimized for time-range queries and downsampling.
- **Geospatial Database:** Optimized for storing and querying location-based data; commonly implements geohashing, which divides the earth's surface into a grid and assigns each cell a hash, to index spatial data efficiently.
- **Full-Text Search Engine:** Optimized for search over unstructured text; built on an **inverted index**, which maps each term to a sorted list of the documents containing it, so that a multi-term query reduces to a merge of those lists rather than a scan of the documents themselves.
- **Vector Database:** Optimized for semantic search over unstructured data represented as high-dimensional embedding vectors; answers similarity queries by locating nearest neighbors under a distance metric such as cosine or Euclidean distance. To sustain high volumes and dimensionality, implementations rely on **approximate nearest-neighbor (ANN)** search, indexing the vectors with structures such as Hierarchical Navigable Small World (HNSW) graphs and trading a tunable loss of recall for lower query latency.
- **Blob Store:** Optimized for storing large, immutable, unstructured objects such as photos, video, audio, executable binaries, and other multimedia, typically as chunked files, designed to scale horizontally while remaining highly available and durable.

### 2.4 Caching Strategies

Caching applies C2's cost hierarchy across a tier boundary rather than within a single structure. We discuss caching mechanisms in the context of the memory and disk tiers of a single machine: a hot subset of the data is held in memory, which is fast, expensive, and volatile, while the authoritative copy remains on disk, which is slow, cheap, and durable. What distinguishes one strategy from another is where a write is acknowledged and whether it populates the cache at all, since those choices fix how far the cached copy may drift from the durable store, which tier a read must consult to be correct, and how much data sits exposed on the volatile tier.

- **Write-Through:** A write is applied to cache and disk together and acknowledged only once both have completed. A read is served from the cache when the key is present, and a miss loads from disk and populates the cache before sending a response. Because both tiers are updated in lockstep, neither copy is ever stale with respect to the other, at the cost of write latency bounded by the slower tier and of cache space spent on data that may never be read.
- **Write-Back:** A write is acknowledged as soon as the cache is updated, and the disk is written asynchronously, often coalescing repeated writes to the same key into one. Reads must be served through the cache, falling back to disk on a miss, since until a flush completes it is the cache, and not the disk, that holds the current value. This gives the lowest write latency and absorbs bursts, but whatever has not yet been flushed is lost if the cache crashes, which under C2 includes the loss of memory to software errors even though node and power failure remain excluded by A7.
- **Write-Around:** A write goes directly to disk, bypassing the cache, and any existing entry for that key must be invalidated; the cache is populated later, on a read miss. This keeps write-heavy, rarely-read data from displacing hot entries, at the cost of a guaranteed miss on the first read following a write, and of a stale read for as long as an invalidation is delayed or missed.

Because the fast tier is bounded, every strategy additionally needs a policy for deciding what to discard. Replacement policies infer future usefulness from past behavior, evicting the least recently used (LRU) or least frequently used (LFU) entry, while a **time-to-live (TTL)** discards an entry after a fixed interval regardless of use, bounding how long a cached value may disagree with the durable copy wherever the two can diverge at all, without requiring the writer to know which entries exist.

### 2.5 Chapter Summary

Assumption A2 treated memory and disk as interchangeable, costless, and durable, so Chapter 1's concurrency mechanisms could be layered on top of storage with no regard for where data physically lived. Constraint C2 puts a real cost hierarchy in its place: memory is fast but expensive and, notwithstanding A7 not yet being relaxed, still volatile to software faults, while disk is durable and cheap but penalizes random access relative to sequential access. Storage design becomes the problem of arranging and placing data so that the operations an application actually performs land on the cheap side of that hierarchy.

The chapter's mechanisms address that problem at three levels. The data structures differ in which access they make cheap: hash indexes turn every write into a sequential append and answer point lookups from an in-memory map, forfeiting efficient range queries and bounding the key space by RAM; B-trees and B+ trees keep both point and range queries efficient at the cost of more expensive in-place updates; and LSM trees absorb writes in memory and flush them as immutable sorted runs, trading read amplification, mitigated by Bloom filters, for consistently sequential writes. The data models repeat the same tension one level up, with relational systems buying strong invariants and a rigid schema at the cost of the impedance mismatch and more constrained scaling, while the NoSQL family and the specialized databases give up generality for a layout matched to one access shape. Caching then applies the same reasoning across the boundary between memory and disk rather than within a single structure, where the choice between write-through, write-back, and write-around fixes how stale the cached copy may become and how much unflushed data sits exposed on volatile memory.

The recurring trade-offs are therefore write efficiency versus read efficiency, memory cost versus disk cost, rigidity versus flexibility, and freshness and durability versus latency. In practice, LSM-based engines fit write-heavy, append-like workloads such as logging or time-series ingestion; B+ trees fit read-heavy workloads dominated by range queries, the traditional OLTP case; relational databases fit applications where strong invariants and complex relationships matter more than raw scale; NoSQL variants fit applications where horizontal scale and schema flexibility outweigh the need for strong consistency; the specialized databases are worth their narrower scope only when the access pattern is itself narrow and well defined; and caching is worth its staleness only when reads concentrate on a small, reusable subset of the data, with write-back reserved for cases where the durability window it opens is acceptable. No single structure, model, or caching strategy is preferable across all of these dimensions simultaneously.

---
