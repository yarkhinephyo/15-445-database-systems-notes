**Optimistic protocol**: Transactions are assumed not to conflict.

DBMS must ensure that execution schedule is equivalent to a serial schedule where transactions are ordered by timestamps.

**Basic timestamp ordering (Basic T/O) protocol**

Transactions do not use locks. Every object in database maintains two attributes - last_read_timestamp and last_write_timestamp. These timestamps must always move forward in time.

<ins>Reading</ins>

If timestamp for transaction is less than last_write_timestamp of an object the transaction has be be aborted and restarted with a new later timestamp. If not, allow the transaction to read the object and update the last_read_timestamp of the object.

<ins>Writing</ins>

If timestamp is less than last_read_timestamp or last_write_timestamp, abort transaction and restart. If not, allow transaction to write to the object and update the last_write_timestamp.

**Thomas write rule**

Allow violation of timestamp ordering protocol. If transaction timestamp is less than last_write_timestamp, ignoring its own write is okay. If there are subsequent reads, the transaction can read its own write.

![](images/Pasted%20image%2020221103123513.png)

**Problems with T/O protocol**

- High overhead for updating timestamps, every read requires a transaction to write to the database.
- Long running transactions can get starved as they are more likely to read something from a newer transaction.
- Timestamp allocation can be a bottleneck with high concurrency.

**Optimistic concurrency control (OCC) protocol**: Private workspace for each transaction.

Objects read are copied into workspace and modifications are applied to workspace. When a transaction commits, the DBMS compares its workspace to see whether it conflicts with other transactions.

1. <ins>Read phase</ins> - Track read/write sets of transactions in private workspace.
2. <ins>Validation phase</ins> - Timestamp is assigned during this phase. Check whether there are RW and WW conflicts with other transactions. If there are conflicts, ensure that all conflicts go one way.
3. <ins>Write phase</ins> - If validation succeeds, write set is applied to the global database.

<ins>Backward validation</ins>

Transaction checks if there is conflict with write_timestamp in the database.

![](images/Pasted%20image%2020221103125626.png)

<ins>Forward validation</ins>

Check timestamp of the committing transaction against other running transactions. The transaction checks against other <ins>private workspaces</ins> instead of the database.

![](images/Pasted%20image%2020221103125709.png)

![](images/Pasted%20image%2020221213172948.png)

**Phantom problem**

Two-phase locking cannot solve this because you can only acquire locks on tuples that exist. It only accounts for update and delete.

![](images/Pasted%20image%2020221108143952.png)

<ins>Repeating Scans</ins> (Simplest) - Re-execute just the scan portions to check if the same result is retrieved. 

<ins>Predicate locking</ins>  (Only in literature) - If locking predicate " status = 'lit' " locks all the records that satisfy the predicate, it is possible to solve the problem.

<ins>Index locking</ins> (Traditional) - If there is an index on the status field, transaction can lock the index page containing the data that satisfy the condition.

Gap locks handle where keys have not be placed yet.

Key range locks cover a key value and the gap to the next key value.

Hierarchical locks work on key ranges with different locking modes.

**Isolation levels**

<ins>Serial</ins>: Commits are done in the same order of submission.

<ins>Serializable</ins>: No phantoms, all reads repeatable, no dirty read.

One implementation is to use index locks (or range locks), plus strict 2PL.

Another implementation is Serializable Snapshot Isolation. It uses an optimistic approach, allowing transactions to proceed without blocking. Before a transaction commits, it is checked and aborted if the execution was not serializable.

<ins>Snapshot isolation</ins>: Phantoms and write skews may happen. Provides repeatable read. Readers and writers do not block the other party.

MVCC is the common implementation. MVCC can be used with 2PL to provide versioning on top of repeatable reads.

<ins>Repeatable reads</ins>: Phantoms may happen.

Strict 2PL, where both shared locks and exclusive are held to the end of transaction.

<ins>Read committed</ins>: Phantoms and unrepeatable reads may happen.

Strict 2PL where shared locks are released immediately after the operation (no need to be shrinking phase). Guarantees that any data read has been committed at the moment that that it is read, but does not guarantee that it will find the same data again.

Another implementation is that instead of requiring shared lock, while the exclusive lock is being used, DBMS can store two versions of the object. The version before and after the write operation. Other transactions can read the "before" version instead of requiring a lock.

<ins>Read uncommitted</ins>:  All may happen except dirty writes.

2PL for exclusive locks. No shared locks.

**My own note on 2PL, Strict 2PL, Rigourous 2PL**

- 2PL: During shrinking phase, no new lock is requested.
- Strict 2PL: During shrinking phase, exclusive locks are not released. Prevents cascading aborts and dirty reads. Read locks can be released.
- Rigourous 2PL: During shrinking phase, no locks are released. There is no need to know the data access pattern (No need to gauge when read lock can be released).