**Query plan**: Operators are arranged in a tree. Data flows from the leaves towards the node. The output of root node is the result of the query.

**Disk-oriented DBMS**: The inputs and outputs of the query may not fit in memory. Buffer pool will be used to implement algorithms that 1) may spill onto disk and 2) maximize sequential IO.

**Heapsort**: Query contains "ORDER BY" and "LIMIT". Go through each element while maintaining a priority queue with size "LIMIT", assuming it is small enough to fit into memory.

**Why quicksort may not be good**: Jumps random pivots and jump around in memory. There will be a lot of swapping in and out of disk if cannot fit in memory.

**External merge sort**

Split data into separate <u>runs</u>, sort them individually and combine them into longer sorted runs.

<u>Run</u>: A run is a list of key-value pairs.

**2-way external merge sort**: Each pass, two runs are merged into a new run.

Assuming a buffer pool holds <u>B</u> pages and there are <u>N</u> pages in the database.

<u>Pass 0</u>: Read pages into memory, sort them and write them back to disk. Total N sorted runs.

<u>Pass 1,2 ...:</u> Recursively merge pairs of runs into runs twice as long. Use 3 buffer pool pages (2 for input pages, 1 for output).

<u>Num passes</u> = 1 + ⌈log<sub>2</sub>(N)⌉

<u>I/O cost</u> = 2 x N x num_passes

![](images/Pasted%20image%2020220929141712.png)

**Double buffering optimization**: Prefetch the next run in the background and store in shadow buffers while processing current run.

**Generalized external merge sort**

<u>Pass 0</u>: Read B pages into memory, sort them and write them back to disk. Produced (N/B) sorted runs (each run is B pages long).

<u>Pass 1,2 ...:</u> Merge (B-1) runs at the same time (1 buffer page left for output).

<u>Num passes</u> = 1 + ⌈log<sub>B-1</sub>(N/B)⌉

<u>I/O Cost</u> = 2 x N x num_passes

**B+Trees for sorting**: If the table must be sorted by a key that is present in B+Tree index, just traverse the leaf pages of the tree. Only works for clustered tree (Physical pages on leaves match the sort order). Unclustered tree, where there are only pointers at the leaf nodes, will cause one I/O per record.

**Aggregation**: Collapsing multiple tuples into a single scalar value. There are sorting and hashing approaches but in general hashing approach is better.

**Sorting aggregation**

```
SELECT DISTINCT cid
FROM enrolled
WHERE grade IN ('B', 'C')
ORDER BY cid
```

Filter for ('B', 'C'), remove unnecessary columns and eliminate duplicates along the way.

<u>Note</u> - In this example, sorting helps with removing duplication. However, if there is no "ORDER BY" clause, hashing aggregating will be cheaper.

**Hashing aggregation**

<u>In-memory</u> - Populate an ephemeral hash table as DBMS scans table and check whether the record already exists.

<u>External hashing aggregation</u>

1. Phase #1 - Divide tuples into partitions based on hash key and write them out to disk when they get full. Use (B-1) buffers for partitions and 1 buffer for input. (Multiple keys can go into the same partition)
2. Phase #2 - Build in-memory hash table for each partition with another hash key and compute aggregation of matching tuples for each bucket. (Assumes that each partition fits into memory) The aggregation is stored as pairs of (key->running_val).

![](images/Pasted%20image%2020220929145112.png)