**Transaction**: A sequence of one or more operations to perform some higher level function. In SQL, starts with "BEGIN" and ends with "COMMIT" or "ABORT".

<ins>Example</ins> - Check balance. Deduct balance. Add to the receipient's account.

**ACID properties**

<ins>Atomic</ins> - All happens or none happens.

<ins>Consistency</ins> - If the database is correct and the transaction is correct, the database will remain correct.

<ins>Isolation</ins> - Illusions that transactions occur in serial order.

<ins>Durability</ins> - If a transaction commits, the effects persist.

**Atomicity mechanism**

<ins>Logging</ins> - DBMS logs all actions so that undo can be done on aborted transactions.

<ins>Shadow paging</ins> - Makes copies of individual pages and makes changes to those copies. Only when a transaction commits, the page is visible to others. Rarely used because of fragmentation issues. Advantage is that the original database is fine if there is a crash.

**Isolation of transactions**: Interleave the transactions while making them appear as if they were run one at a time.

**Interleaving transactions**

Interleaving is done because if one transaction is stalled, we would want another transaction to make forward progress.

In the example below, the total amount of money will become inconsistent in the system.

Database only sees "READ" and "WRITE" transactions, it does not know about commutativity/associativity of operations and so on.

![](images/Pasted%20image%2020221027123605.png)

**Schedule**: The order in which the DBMS execute operations.

A schedule is correct if it is equivalent is <ins>serial execution</ins>. <ins>Equivalent schedules</ins> mean that the effect of schedules on the database produces the same states.

**Serializable schedule**: Schedule is equivalent is some serial execution of transactions.

**Conflicting operations**: Operations are on different transactions and one of them is a write.

<ins>Unrepeatable read</ins> - Caused by read-write conflict. For example, R1, W2, R1, COMMIT1, COMMIT2.

<ins>Dirty read</ins> - Caused by write-read conflict. For example, R1, W1, R2, W2, ABORT1, COMMIT2.

<ins>Lost update</ins> - Caused by write-write conflict. For example, W1, W2, COMMIT1, COMMIT2.

**Methods to check serializability (Not to produce)**

<ins>Conflict serializability</ins> - DBMS generally try to support this.

Two schedules are conflict equivalent if both schedules involve same actions of same transactions and every pair of conflicting actions is ordered in the same way.  Schedule is conflict serializable if it is conflict equivalent to some serial schedule. (Can transform the schedule by swapping non-conflicting operations of different transactions)

For more than two transactions, draw out a precedence graph where edges are drawn from one transaction to another when there is a conflict between operations. A schedule is conflict serializable if the precedence graph is acyclic.

![](images/Pasted%20image%2020221101112516.png)

<ins>View serializability</ins> - No DBMS does this. Required to understand the application logic.