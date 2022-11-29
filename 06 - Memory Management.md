**Memory management**: How DBMS moves data back-and-forth from disk. The other parts of the system does not need to know.

**Buffer pool**: Memory region with array of fixed-size pages (frame). When there is a request for a page, an exact copy (no manipulation) is placed into these frames. Dirty pages are not written to disk immediately.

**Page table (Buffer pool)**

- Keeps track of what frame includes what page.
- Dirty flag to check if they are dirty.
- Pin counter which tracks how many threads are referencing the page.

**Locks vs latches (Database world)**

<u>Lock</u>: Protects logical contents from other transactions. Held for transaction duration. Need to be able to rollback changes. (Different from locks in OS)

<u>Latches</u>: Low level protection primitives to protect regions of memory. Only held for the duration of the operation.

**Page table vs page directory**

<u>Page directory</u>: Mapping from page IDs to page locations in database files

<u>Page table</u>: Mapping from page IDs to copies in buffer pool frames. No need to be durable.

**Allocation policies**

<u>Global policy</u> - Decisions for all active transactions.

<u>Local policy</u> - Allocate frames to a specific transaction without considering behaviours of other transactions.

**Multiple buffer pools**

DBMS usually have multiple buffer pools with their own page tables and policies.

Help reduce latch contention.

<u>Approach 1 to find buffer pool</u> - Embed object identifer in object IDs and maintain a mapping from objects to buffer pools.

<u>Approach 2 to find buffer pool</u> - Hash the page ID to select which buffer pool to access.

**Pre-fetching**: Prefetch pages based on query plan. For example, sequential scans/ index scans.

`mmap` may be able to prefetch similar to sequential scan, but it cannot do index scans as OS does not have context about the queries.

![](images/Pasted%20image%2020220915125251.png)

**Scan sharing**

If a query wants to scan a table and another query is doing this, the DBMS will attach the second query cursor to the existing cursor. Different from result caching because queries can be different.

The second query may "wrap around" the end to cover all rows.

Relational model does not specify order. So if second query starts at page 3 with a query like `SELECT SUM(A) FROM A LIMIT 100`, it will not look at page 0 at all.

**Buffer pool bypass**: Allocate a small amount of memory to query and the pages go to here instead of the buffer pool. Memory is local to the query. Called "light scans".

**OS Page Cache**

All the disk operators go through OS API. Most DBMS use direct I/O (O_DIRECT) to bypass the OS's cache.

<u>O_DIRECT</u>: Instructs to bypass page cache and perform any IO operations directly against storage. Buffers in the application space are flushed directly to disk without copying data to page cache and waiting for the write-back operations.

**Buffer replacement policies**

Simple way is to use LRU cache to remove page with oldest timestamp.

**1. Clock**: Approximation of LRU that does not need separate timestamps.

Each page has a reference bit. If a page is accessed, set to 1. When sweeping, set reference bits from 1 to 0, and evict the 0s.

<u>Problems</u> - Susceptible to sequential flooding. Sequential scan pollutes the buffer pool with pages that are not read again.

**2. LRU-K**: Track history of last K references' timestamps, compute intervals between them. If intervals are long, evict them.

**3. Localization**: DBMS chooses which pages to evict on per transaction basis. Minimizes the pollution of the buffer pool from each query.

**Dirty pages**

<u>Fast evictions</u>: If a page is not dirty, drop it.

<u>Slow evictions</u>: If a page is dirty, DBMS must write back to disk.