**Locks**

Use locks to guarantee that all schedules are serializable without knowing the entire schedule ahead of time.

Locks isolate user transactions while latches isolate threads.

![](images/Pasted%20image%2020221101115700.png)

**Lock execution**

1. Transaction requests lock.
2. Lock manager grants or blocks requests.
3. Transaction releases lock.
4. Lock manage updates the internal lock-table.

**Two-phased locking (Pessimistic protocol)**

1. <u>Growing</u> - Transaction requests all the locks it need. (Cannot unlock during growing phase)
2. <u>Shrinking</u> - Transaction is allowed to only release or downgrade locks that it previously acquired.

![](images/Pasted%20image%2020221101120914.png)

Subjected to <u>cascading aborts</u> which is a performance issue. For example, an abort of one transaction will cause other transactions to abort too.

![](images/Pasted%20image%2020221101121255.png)

To avoid <u>dirty reads</u> (reading uncommitted data), we use strong strict 2PL which only release the exclusive locks at the end of transactions. It can release shared locks during the shrinking phase. (In rigorous 2PL, both shared and exclusive locks are held till commit/abort)

![](images/Pasted%20image%2020221101121640.png)

![](images/Pasted%20image%2020221101122431.png)

![](images/Pasted%20image%2020221101122442.png)

**Universe of schedules**

![](images/Pasted%20image%2020221101122922.png)

**Deadlock detection**

Background thread that builds <u>waits-for graph</u> to keep track of what locks each transaction is waiting to acquire.

When detected, a "victim" transaction is restarted or aborted (more common) to break the cycle.

![](images/Pasted%20image%2020221101123501.png)

Trade-off between frequency of checking for deadlocks and how long transactions wait before deadlocks are broken.

**Deadlock prevention**: When a transaction acquires a lock, if it is already held, kill one of them.

<u>Wait-Die</u> - If requesting transaction has higher priority, it waits for holding transaction. If it has lower priority, aborts.

<u>Wound-Wait</u> - If requesting transaction has higher priority, holding transaction aborts and releases lock. If it has lower priority, it waits.

![](images/Pasted%20image%2020221101130013.png)

**Lock granularity**

Trade-off between parallelism and overhead. DBMS should hold the fewest locks that a transaction needs.

Every transaction holds all the locks as they go down. For example, if you have a tuple lock, you also have the table lock.

![](images/Pasted%20image%2020221101130546.png)

**Intention lock**

<u>Intention-shared</u> - There is some transaction in shared lock at lower level.

<u>Intention-exclusive</u> - There is some transaction in exclusive lock at lower level.

<u>Shared-intention-exclusive</u> - Node is locked in shared mode and there is some exclusive locking at lower level.

![](images/Pasted%20image%2020221103120102.png)

**Intention lock protocol**

To get S or IS lock on a node, transaction must hold IS on parent node. To get X, IX, SIX on a node, must hold at least IX on parent node.

- T1 - Scan table R and update a few tuples.
- T2 - Read a single tuple.
- T3 - Scan all tuples in table R.

![](images/Pasted%20image%2020221103120742.png)

**Lock escalation**

DBMS can switch to coarser-grained locks when transaction acquires too many low level locks.