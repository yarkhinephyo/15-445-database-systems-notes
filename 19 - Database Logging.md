**Failure classifications**

<ins>Transaction failures</ins>

1. Logical errors - Transaction cannot complete due to logical error.
2. Internal state errors - Terminate active transaction due to an error (deadlock).

<ins>System failures</ins>

1. Software failure - Problem with OS or DBMS implementation.
2. Hardware failure. The computer hosting the DBMS crashes.

<ins>Storage media failure</ins>

1. Non-repairable hardware failure. (Unrecoverable)

**Steal policy**: Does DBMS allow an uncommitted transaction to overwrite the most recent committed value in non-volatile storage?

For example, no-steal means that any value that is modified but not committed cannot leave the buffer pool back onto the disk.

**Force policy**: Does the DBMS require updates made by a transaction to reflect on the non-volatible storage (flush to disk) before they can say "committed"?

**Buffer pool with no-steal and force policy**

Must make a copy of a page, only apply changes that T2 made and flush to page. After flushing, can return as "T2 committed".

![](images/Pasted%20image%2020221114215122.png)

<ins>Advantages</ins> - Never have to undo changes of an aborted transaction because the changes are not written to disk. Never have to redo changes because all the changes are guaranteed to be written to disk.

<ins>Disadvantages</ins> - For both transactions in the example commit, the buffer pool manager has to flush page twice. Cannot support writesets that exceed that physical memory (RAM) available.

**Shadow paging**

DBMS copies page on write to create two versions. Master version contain changes from committed transactions and shadow version contains changes from uncommitted transactions.

When a transaction commits, overwrite the root pointer to point to the shadow version, thus swapping the two.

![](images/Pasted%20image%2020221114221241.png)

<ins>Advantages</ins> - Never have to redo changes. Undo is only removing the shadow pages.

<ins>Disadvantages</ins> - Copying the entire table is expensive. Data gets fragmented during copy-on-writes. Need garbage-collection. Only supports one transaction at a time.

**Write-ahead log**

Log contains enough information to undo and redo the database. DBMS must write to disk the log file records of an object before flushing the object to disk.

<ins>Steal and no-force policy</ins>

Can write out dirty pages before they are commited. Not required to flush out the dirty pages before saying "committed". (Have to still flush out the log records)

**WAL protocol**

All transaction log records are staged in volatile storage. These are written to non-volatile storage before the page itself is written (flushed) in non-volatile storage.

1. Write a BEGIN log record to mark the starting point.
2. When transaction finishes, write a COMMIT log record. Make sure all log records are flushed before returning an acknowledgement to the application (client).

```
Transaction ID, Object ID, Before Value (Undo), After Value (Redo)
```

In MVCC, there is no need for "Before Value" because only appending of new values is done; you never need to undo a previous value.

<ins>Example</ins>

Put the log record in memory before writing to buffer pool.

![](images/Pasted%20image%2020221114230630.png)

Ensure all the log records are committed before returning "committed" status to the outside world.

![](images/Pasted%20image%2020221114230806.png)

**WAL Group committ**

Every time flushing to disk, need to wait for disk to respond before flushing again. To optimize, batch transactions together so that multiple potential flushes are combined together.

Flush the logs 1) when there is a timeout or 2) when the buffer is full.

![](images/Pasted%20image%2020221114231813.png)

**Logging schemes**

<ins>Physical logging</ins> - Byte level changes made to a page. (Like git diff) 

<ins>Logical logging</ins> - Record high level operations by transactions such as INSERT, UPDATE.

<ins>Physiological logging</ins> - Hybrid approach with byte level changes for a single tuple identified by page ID + slot. Don't know the tuple's offset in the page. If there was a vaccuming, the tuple will be still found with page ID and slot.

![](images/Pasted%20image%2020221114233839.png)

**Log-structured systems**

Log-structured DBMSs do not have dirty pages. DBMS buffers log records in in-memory pages (MemTable). If the buffer is full, must flush to disk. It may contain changes for uncommitted transactions.

These DBMSs maintain a separate WAL to recreate the MemTable on crash.

**Checkpoints**

The DBMS periodically takes a checkpoint where all buffers are flushed to disk. This prevents DBMS from replaying very long WAL records after crashes.

1. Pause all queries.
2. Flush WAL records to disk.
3. Flush modified pages from the buffer pool.
4. Write a checkpoint entry in WAL and flush to disk.
5. Resume queries.

After a crash, need to redo transactions that committed after the checkpoint. In the example below, need to <ins>redo</ins> T2 and <ins>undo</ins> T3.

![](images/Pasted%20image%2020221114235628.png)