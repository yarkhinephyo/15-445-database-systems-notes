**Transaction**: A sequence of one or more operations to perform some higher level function.

<u>Transfer money example</u> - Check balance. Deduct balance. Add to the receipient's account.

**SQL Transactions**: Starts with "BEGIN" and ends with "COMMIT" or "ABORT".

**ACID**

<u>Atomic</u> - All happens or none happens.

<u>Consistency</u> - If the database is correct and the transaction is correct, the database will remain correct.

<u>Isolation</u> - Illusions that transactions occur in serial order.

<u>Durability</u> - If a transaction commits, the effects persist.

**Atomicity mechanism**

<u>Logging</u> - DBMS logs all actions so that undo can be done on aborted transactions.

<u>Shadow paging</u> - Makes copies of individual pages and makes changes to those copies. Only when a transaction commits, the page is visible to others. Rarely used because of fragmentation issues. Advantage is that the original database is fine if there is a crash.

**Isolation of transactions**: Interleave the transactions while making them appear as if they were run one at a time.

**Interleaving transactions**

Interleaving is done because if one transaction is stalled, we would want another transaction to make forward progress.

In the example below, the total amount of money will become inconsistent in the system.

Database only sees "READ" and "WRITE" transactions, it does not know about commutativity/associativity of operations and so on.

![](images/Pasted%20image%2020221027123605.png)

**Schedules**: Schedule is correct if it is equivalent is <u>serial execution</u>. <u>Equivalent schedules</u> mean that the effect of schedules on the database produces the same states.

**Serializable schedule**: Schedule is equivalent is some serial execution of transactions.

**Conflicting operations**: Operations are on different transactions and one of them is a write.

<u>Unrepeatable read</u> - R1, W2, R1, COMMIT1, COMMIT2.

<u>Dirty read</u> - R1, W1, R2, W2, ABORT1, COMMIT2.

<u>Lost update</u> - W1, W2, COMMIT1, COMMIT2.

**Methods to check serializability (Not to produce)**

<u>Conflict serializability</u> - DBMS generally try to support this.

Two schedules are conflict equivalent if both schedules involve same actions of same transactions and every pair of conflicting actions is ordered in the same way.  Schedule is conflict serializable if it is conflict equivalent to some serial schedule. (Can transform the schedule by swapping non-conflicting operations of different transactions)

For more than two transactions, draw out a precedence graph where edges are drawn from one transaction to another when there is a conflict between operations. A schedule is conflict serializable if the precedence graph is acyclic.

![](images/Pasted%20image%2020221101112516.png)

<u>View serializability</u> - No DBMS does this. Required to understand the application logic.