**ARIES**

1. Write-ahead logging (Steal and no-force buffer pool)
3. Repeating history during redo
4. Logging changes during undo

**WAL Record**

Log sequence number is globally unique number for each log.

- flushedLSN - In-memory tracking of the last LSN log on disk.
- pageLSN - Newest update to page. In every page.
- recLSN - Oldest update to page since it was last flushed. In every page.
- lastLSN - Latest record of a transaction.
- MasterRecord - LSN of the latest "checkpoint".

Before the DBMS write a page to disk, it must flush the log at least to the point where pageLSN <= flushedLSN. (The latest update is flushed)

**Writing WAL records**

If pageLSN <= flushLSN, it means that the latest log for the page has been flushed. It is safe to evict the page.

If not, that means the latest log for the page is still in memory only. Not safe to flush the page.

![](images/Pasted%20image%2020221115234622.png)

Update the pageLSN every time a transaction modifies a record.

Update the flushedLSN every time the DBMS writes out the WAL buffer to disk.

**Transaction commit**

DBMS guarantees that all log records up to the transactions "COMMIT" are flushed to disk before returning to the application.

When the commit finally succeeds (done with internal meta-data updates), write a special TXN-END record to log. This means no new log record for the transaction will appear in the log ever again. The in-memory log can be trimmed up to the flushedLSN.

![](images/Pasted%20image%2020221116000246.png)

**Transaction abort**

"prevLSN" tracks the previous LSN for a transaction.

To abort, the log records need to include the steps we take to undo the transaction.

![](images/Pasted%20image%2020221116000812.png)

<u>Compensation log record (CLR)</u>

CLR record describes an undo action. It contains all the fields of an update record plus the "undoNext" pointer which points to the next LSN to undo.

DBMS does not wait for them to be flushed before notifying the application that transaction aborted.

If the DBMS crash before the ABORT line is flushed, the recovery will undo the changes.

If another transaction wants to use the values that were updated during an aborted transaction, the logs will just be appended after CLR records are added. (No need to wait for disk flush)

**Non-fuzzy checkpoints**

Original checkpointing method waits till all active transactions finish before flushing dirty pages to disk. This is bad for runtime performance.

Pause modifying transactions while the DBMS takes checkpoint. No write latches will be acquired on pages.

<u>Metadata</u> - What transactions were running during checkpoint? What pages were dirty during checkpoint? 

**Active transaction table (ATT)**

An entry per active transaction. Remove after "TXN-END" record.

```
(txnId, status, lastLSN)
```

<u>Status</u> - Running, Committing, Candidate for Undo.

**Dirty page table (DPT)**

One entry per dirty page in the buffer pool.

**Example**

![](images/Pasted%20image%2020221116094923.png)

**Fuzzy checkpoint**: Allow transactions to keep modifying while taking the snapshot. No attempt to force dirty pages to disk.

<u>New log records</u> - CHECKPOINT_BEGIN, CHECKPOINT_END{ATT, DPT}

![](images/Pasted%20image%2020221116095435.png)

1. When checkpoint begins, system looks at log records before checkpoint begin and try to write all the pages in the buffer pool onto the disk.
2. Existing transactions can still make modifications.
3. After everything before CHECKPOINT_BEGIN is flushed, DBMS records CHECKPOINT_END with active transactions and dirty pages modified by these transactions. (Transactions that begin after checkpoint starts are excluded from ATT.)

Master record is updated to LSN of CHECKPOINT_BEGIN after completion. That is the anchor point to start analysis during recovery.

**Recovery phases**

![](images/Pasted%20image%2020221116131732.png)

<u>Analysis phase</u>

Scans forward from the last checkpoint.  If a log record is not in ATT, add it with status "UNDO". On commit, change transaction status to "COMMIT". If TXN-END record is found, remove the corresponding transaction from ATT.

For update log records, if a page is not in DPT, add it to DPT and update its "recLSN" to "LSN".

ATT tells us what transactions were active at the time of crash. DPT tells us what dirty pages might not have been made it to disk.

In the example below, T96 has been added to ATT with status "UNDO". The CHECKPOINT_END provides active transactions and dirty pages that occur before CHECKPOINT_BEGIN. These are also used to update ATT and DPT.

![](images/Pasted%20image%2020221116132857.png)

Mark T96 as "COMMIT" in ATT.

![](images/Pasted%20image%2020221116132910.png)

Remove T96 from ATT after TXN-END is seen.

![](images/Pasted%20image%2020221116132918.png)

<u>Redo phase</u>

Scan forward from the log record containing the smallest "recLSN" in DPT. Redo the actions for each update or CLR log records as long as 1) the affected page is in DPT and 2) log record's LSN is equal or more than the page's "recLSN".

To redo, reapply logged update and set the "pageLSN" to log record's LSN. No need logging!

<u>Undo phase</u>

Undo all transactions that were active at the time of crash. Transactions with status "UNDO" after the analysis phase.

Write a CLR for every modification.

![](images/Pasted%20image%2020221116135112.png)

If the server has a 2nd crash before T2 is undone, there will only be T2 in the ATT after the analysis phase. During the redo phase, the "lastLSN" for T2 will be updated to 20. The undo phase will begin with LSN 20 for T2.

![](images/Pasted%20image%2020221116135758.png)