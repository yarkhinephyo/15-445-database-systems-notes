**Atomic commit protocol**

How to ensure all nodes agree to a transaction? Assumption is that all nodes in the DBMS are trusted and well-behaved.

<ins>Two phase commit</ins>

![](images/Pasted%20image%2020221122120538.png)

Each node maintains a log of messages in 2 phase commit (2PC). If there is a crash, check if whether currently involved in 2 phase commit. 

1. If node is in prepared phase (already told coordinator), contact coordinator to find out the state.
2. If node is not in prepared state, abort the transaction.
3. If transaction was committing and the node is the coordinator, send commit message to nodes.

Coordinator assumes that the node has aborted if no acknowledgement is received.

<ins>Early Prepare Voting (Rare)</ins> - Coordinator sends the last query to remote node and the node return their vote for the prepared phase with the query result.

<ins>Early Ack After Prepare (Common)</ins> - If all nodes vote to commit a transaction, the coordinator tells the application server that commit is successful. The coordinator does not wait for the nodes to acknowledge the commit message.

**Paxos**

Unless two phase comit, only the majority of the nodes have to agree.

There are proposers, acceptors and followers (not covered).

Let's say there are 1 proposer and 3 acceptors.

1. Proposer sends out message to acceptors.
2. If 2 nodes out of 3 agrees, proposer sends commit messages to all nodes.
3. If 2 nodes out of 3 acknowledges, the commit is considered successful.

![](images/Pasted%20image%2020221122122735.png)

Proposals are marked with incrementing counters (n). If the counter goes up (n+1) with a new proposal, the commit messages associated with the previous counter will be rejected by the acceptors. 

![](images/Pasted%20image%2020221122124004.png)

**Replica configurations**

<ins>Primary-replica</ins>

All updates go to a designed primary. The replicas are updated without atomic commit protocol (through write ahead log). If primary goes down, election is held to select a new primary.

If a write is commited to the primary, the reads at the replica may not see it immediately (lower consistency).

<ins>Multi-primary</ins>

Transactions can update objects at any replica. Replicas must synchronize with each other via atomic commit protocol.

**K-Safety**: Threshold for fault tolerance of the replicated database.

**Propagation levels**

<ins>Synchronous (Strong consistency)</ins>

Primary sends updates to replicas and waits for them to acknowledge that changes are fully applied before changing.

<ins>Asynchronous (Eventual consistency)</ins>

Primary immediately returns an acknowledgement to the client without waiting for the replicas to acknowledge.

**Propagation timings**

<ins>Continuous</ins> - The DBMS sends log messages as it generates them.

<ins>On commit</ins> - The DBMS sends log messages only once a transaction commits. Logs are not sent for the aborted transactions. 

**CAP Theorem**

Impossible for a distributed system to be...

1. Consistency - If one says committed, all replicas should reflect the same.
2. Availability - All requests are satisfied.
3. Partition tolerance - If network is down and up, database still operates still correctly.

Tradition DBMSs choose consistency and availability over network partition tolerance. If not the majority of nodes can be communicated, stop all requests.

NoSQL DBMSs provide mechanisms to resolve conflicts after nodes are reconnected. When network is down, still can use the nodes.

**PACELC Theorem**

In case of network partitioning (P) in a distributed system, one has to choose between availability (A) and consistency (C), else (E), even when the system runs normally without network partitions, one has to choose between latency (L) and consistency (C).