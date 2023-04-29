**Concurrency control**: Method that DBMS uses to ensure correct results for concurrent operations on an object.

**Logical correctness**: Can a thread see the data its supposed to see?

**Physical correctness**: Is the internal data structures still sound?

**Lock**: Protects logical contents from other transactions. Need to be able to rollback changes.

**Latches**: Protects the critical sections of internal data from other threads. There are read and write modes (Many readers, one writer).

**Latch implementations**

<ins>Blocking OS mutex</ins> - Slow and non-scalable.

<ins>Test-and-set spin latch</ins> - Efficient but non-scalable. Using a while loop will burn out CPU cycles.

<ins>Reader/writer latch</ins> - Must manage read/write queues to avoid starvation. Can be implemented on top of spin latches.

**Hash table latching**

Easy to support concurrency as threads only access a single slot at a time (Simple linear probe hash table). For resizing, a global latch can be acquired for the entire table.

<ins>Page latch</ins> only uses one latch per page (less storage) but less parallelism.

![](images/Pasted%20image%2020221011173615.png)

<ins>Slot latches</ins> take up more storage and acquiring time but may provide more parallelism.

![](images/Pasted%20image%2020221011173648.png)

Since every thread is scanning top to bottom (same direction), there will not be deadlock. Latches can be released before jumping from slot to slot or page to page.

**B+Tree latching**

- Threads may modify one node at the same time.
- Threads may traverse the tree while another thread splits/merges nodes.

<ins>Latch crabbing/coupling</ins>

Protocols for multiple threads to access B+Tree. Only release latch for the parent if the child that has been entered is assessed as "safe". A "safe" node is one that will not split or merge when updated.

<ins>Finding</ins> - Acquire latch on child and unlatch parent.

<ins>Insert/Delete</ins> - One a child is considered safe, release all latches on ancestors.

![](images/Pasted%20image%2020221011174459.png)
*Example for inserting 45, release lock on B and D once node I is deemed as safe to insert*

![](images/Pasted%20image%2020221011174358.png)
*Example for deleting 38*

<ins>Acquiring write-latch on root</ins>: Bottleneck for higher concurrency.

**Better latching algorithm (Inserts/Deletes)**

Use read latches to reach the leaf node and verify that it is safe. 

If safe, set the write latch on leaf node. If it is not safe, then redo with write latches.

**How to support leaf node scan**

<ins>Finding</ins> - Do not release previous latch until the next latch is acquired. No problem for multiple threads due to reader locks.

<ins>Deleting</ins> - If read latch cannot be acquired due to another thread writing, just kill itself.

![](images/Pasted%20image%2020220927142030.png)