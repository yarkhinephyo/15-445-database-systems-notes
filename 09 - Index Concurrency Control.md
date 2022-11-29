**Concurrency control**: Method that DBMS uses to ensure correct results for concurrent operations on an object.

**Logical correctness**: Can a thread see the data its supposed to see?

**Physical correctness**: Is the internal data structures still sound?

**Lock**: Protects logical contents from other transactions. Need to be able to rollback changes.

**Latches**: Protects the critical sections of internal data from other threads. There are read and write modes (Many readers, one writer).

**Latch implementations**

<u>Blocking OS mutex</u> - Slow and non-scalable.

<u>Test-and-set spin latch</u> - Efficient but non-scalable. Using a while loop will burn out CPU cycles.

<u>Reader/writer latch</u> - Must manage read/write queues to avoid starvation. Can be implemented on top of spin latches.

**Hash table latching**

Easy to support concurrency as threads only access a single slot at a time (Simple linear probe hash table). For resizing, a global latch can be acquired for the entire table.

<u>Page latch</u> only uses one latch per page (less storage) but less parallelism.

![](images/Pasted%20image%2020221011173615.png)

<u>Slot latches</u> take up more storage and acquiring time but may provide more parallelism.

![](images/Pasted%20image%2020221011173648.png)

Since every thread is scanning top to bottom (same direction), there will not be deadlock. Latches can be released before jumping from slot to slot or page to page.

**B+Tree latching**

- Threads may modify one node at the same time.
- Threads may traverse the tree while another thread splits/merges nodes.

<u>Latch crabbing/coupling</u>

Protocols for multiple threads to access B+Tree. Only release latch for the parent if the child that has been entered is assessed as "safe". A "safe" node is one that will not split or merge when updated.

<u>Finding</u> - Acquire latch on child and unlatch parent.

<u>Insert/Delete</u> - One a child is considered safe, release all latches on ancestors.

![](images/Pasted%20image%2020221011174459.png)
*Example for inserting 45, release lock on B and D once node I is deemed as safe to insert*

![](images/Pasted%20image%2020221011174358.png)
*Example for deleting 38*

<u>Acquiring write-latch on root</u>: Bottleneck for higher concurrency.

**Better latching algorithm (Inserts/Deletes)**

Use read latches to reach the leaf node and verify that it is safe. 

If safe, set the write latch on leaf node. If it is not safe, then redo with write latches.

**How to support leaf node scan**

<u>Finding</u> - Do not release previous latch until the next latch is acquired. No problem for multiple threads due to reader locks.

<u>Deleting</u> - If read latch cannot be acquired due to another thread writing, just kill itself.

![](images/Pasted%20image%2020220927142030.png)