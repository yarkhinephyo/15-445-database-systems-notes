**Inner equijoin**: Joins tables where keys are equal. Can be tweaked to support other joins.

**Runtime decisions**

<u>Output types</u>: What data does the join operator emit to parent? For example, should the tuples be ordered or what format should they be?

<u>Cost analysis</u>: Which join algorithm should be used?

**Output Types**

<u>Early materialization</u>: Copy values of attributes in outer and inner tuples into a new tuples. Subsequent operators (in parent) would not need to go back to original table for data.

![](images/Pasted%20image%2020221011165945.png)

<u>Late materialization</u>: Only copy the join keys with record IDs. Subsequent operators would have to copy data from the original records.

![](images/Pasted%20image%2020221011170003.png)

**Cost analysis criteria**

Cost metric is I/O count. The I/O occurred to output the result is not considered since it will be the same for different join algorithms.

**1. Nested loop join**

<u>Stupid nested loop</u>: Nested for loops. If the keys match, add to the output. Cost = M + (m x N) where M and N are number of pages and m is the number of tuples on left table.

<u>Block nested loop</u>: Inner for-loops for caching. Cost = M + (M x N) if there are only 1 buffer for each table. If there are B buffers, Cost = M + ⌈M/(B-2)⌉ x N. If there are enough buffers, reduces to Cost = M + N.

<u>Index nested loop:</u> What most OLTP systems implement. A naive scan on outer table, for every tuple, if there is a match found in index, add to output. Cost = M + (m x C) where C is the time to search up the index.

Takeaway is that smaller table should be the outer table.

**2. Sort-Merge join**

Step through two sorted tables with cursors and emit matching tuples.

Cost = SortCost + (M + N), if keys are unique. Worst case is SortCost + (M x N) where there is only one value for all join keys in both tables.

**3. Hash Join**

<u>Build</u>: Scan outer relation and build a hash table using the join attributes. The original keys are still needed for hash collisions. Values can be tuples or tuple IDs.

<u>Probe</u>: Scan the inner relation and use hash function to find the matching tuple.

**Bloom filter**

More compact than hash table itself. Threads check the filter before probing the hash table. If it does not exist in Bloom filter, will not exist in hash table.

Giant bitmap that answers set member queries. It only says whether a key exists, not the location. False negatives will never occur (If it says key does not exist, it really does not).

<u>Insert</u>: Use k hash functions to set bits in the filter to 1.

<u>Lookup:</u> Check the bitmap to see whether bits are 1 for each hash function.

**Partition hash join**

For when hash table does not fit into memory.

<u>Build</u>: Hash both tables on the join attributes into partitions.

<u>Probe</u>: Compare tuples in corresponding partitions and use a join algorithm (hash join/nested join). The pages will fit in memory so the join operation will be fast.

If the buckets do not fit into memory, hash a bucket recursively into more buckets.

![](images/Pasted%20image%2020221004130453.png)

Cost = 3 x (M + N) for partitioned hash join. 2 x (M + N) for read/write both tables during building, 1 x (M + N) for reading during probing. The output is not included.

**Summary of join algorithms**

![](images/Pasted%20image%2020221004131103.png)