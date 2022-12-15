The DBMS maintains multiple <u>physical</u> versions of a single local object. Transactions read the newest version of the object when the transaction started.

**Advantages**

- Writers do not block readers and readers do not block writers. (The writer writes on a new physical object while the readers are reading the previous one)
- Read transactions do not require locks to ensure consistency.
- Possible to support time-travel queries.

**Implementation**

Timestamps are assigned to transactions at the <u>start of transaction</u>. This is different from OCC where they are assigned at the validation phase.

<u>Example 1</u>

A<sub>0</sub> will be read by transactions between 0 inclusive and 2 exclusive. So transaction with timestamp 1 will always read A<sub>0</sub>.

In a separate location, track whether transactions have been committed.

![](images/Pasted%20image%2020221108130317.png)

<u>Example 2</u>

In this example, transaction with timestamp 2 reads A<sub>0</sub> because the earlier transaction has not committed yet even though there was a write.

![](images/Pasted%20image%2020221108131741.png) 

After transaction 1 commits, transaction 2 commits. Let's assume 2PL is being used. Under read committed, transaction 2 would commit. Under snapshot isolation, transaction 2 will abort (If it is deadlock avoidance with first-writer-win rule).

![](images/Pasted%20image%2020221108132322.png)

**Snapshot isolation**

When a transaction starts, it sees a consistent snapshot of the database from the beginning of the transaction. It means if someone else commits writes after the transaction's read timestamp, it will not see the changes.

The database enforces that two committed transactions have disjoint write sets.

Prone to <u>write skew anomaly</u>.

![](images/Pasted%20image%2020221123193752.png)

**Concurrency control protocol**

<u>Timestamp ordering</u> - Transaction timestamps that determine serial order.

<u>Optimistic concurrency control</u> - Three-phase protocol with private workspace.

<u>Two phase locking</u> - Transaction acquire local of physical version before reading and writing logical tuples.

**MVCC + Two phase locking**

Use a different concurrency control scheme for read transactions than for update transactions.

Use MVCC for reads so that they do not block the writers. Read-only transactions are assigned a timestamp and they only see the versions of objects with timestamps before the start of transaction.

Use strict 2PL to schedule the operations of update transactions. Read-only transactions are ignored for 2PL.

**Version storage**

DBMS maintains version chain per logical tuple. Indexes point to the head of the chain.

<u>Append-only</u> - New versions are appended to table on every update. As shown in previous examples.

Oldest-to-newest (O2N) ordering means the index points to the oldest part of the chain and adding new version is just appending to the chain, but lookups need to traverse chain. Newest-to-oldest (N2O) ordering means every new version must update index pointers but lookups do not require traversing the chain.

<u>Time-travel</u> - Old versions are copied to separate table space.

There are main table and time travel table physically in the database. Update means the value in main table is copied into time travel table and updates the version.

![](images/Pasted%20image%2020221110145610.png)

<u>Delta table</u> - Every update, only copy the values that were modified to the delta storage. Faster writes but slower reads.

![](images/Pasted%20image%2020221110145916.png)

**Garbage collection**

DBMS needs to remove reclaimable physical versions over time. Need to know if 1) expired 2) safe to reclaim from memory.

<u>Tuple-level</u> - Find old versions by examining tuples.

In background vacuuming, background threads scan the table and look for reclaimable versions (outside active transactions).

In cooperative cleaning, worker threads (not background threads) identify reclaimable versions as they traverse version chain (Only for O2N). Disadvantage is that some tuples may never be reclaimed if unread.

<u>Transaction-level</u> - Transactions keep track of old versions.

On commit/abort, the transaction provide read and write sets to the centralized vacuum worker.

**Indexes and version chains**

Primary key indexes point to version chain head (oldest/newest). When primary key is updated, it is treated as delete + insert.

![](images/Pasted%20image%2020221213194813.png)

For secondary indexes, there can be a logical pointer that requires an indirection layer (RID, primary key) or a physical pointer that points to the head of version chain. (PosgreSQL)

![](images/Pasted%20image%2020221213194842.png)

![](images/Pasted%20image%2020221213194827.png)

For physical pointer, if there are many secondary indexes, all will have to be updated. (MySQL) For retrieving the logical pointer, the primary key index has to be queried again. If there are many secondary indexes, only the primary index has to be updated.
