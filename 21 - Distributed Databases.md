**System architecture**: What shared resources are directly accessible to CPUs?

1. Shared everything - Disk, memory, CPU are all located together.
2. Shared memory - CPUs not co-located. Memory and disk are the same.
3. Shared disk (Common) - Nodes with local CPU and memory. Disk is shared.
4. Shared nothing (Common) - Nodes with own CPU, memory and disk.

**Shared memory**: Each processor has a global view of the in-memory data structures. Each DBMS instance on a processor knows about other instances.

**Shared disk**: All CPUs connect to disk via an interconnect. Can scale execution layer <ins>independently</ins> from storage layer. In theory, can kill the nodes any time, which is an advantage over Shared-Nothing architecture. Must send messages between nodes to know about current states. More common in cloud-based DBMSs.

<ins>Examples</ins> - Spark, Presto, Google BigQuery

**Shared nothing**: Nodes communicate with each other via network. Better performance but harder to scale capacity and ensure consistency.

There is a catalog that tracks which ID resides in which node.

![](images/Pasted%20image%2020221120141304.png)

Challenge is when a new node gets added, data has to be shuffled. Safely and transparently.

![](images/Pasted%20image%2020221120141513.png)

**Homogenous Nodes**: Each node in the cluster can perform the same set of tasks as other nodes. Makes it easier for provisioning and failover.

**Heterogenous Nodes**: Each node has a specific task. So communication must occur between nodes. Can independently scale from one node to another. For example, MongoDB has router nodes routing queries to shards and config nodes storing mappings from keys to shards.

<ins>Example in MonogDB</ins>

Router node checks where is the data from the Config node. The router node routes the query to the correct node.

![](images/Pasted%20image%2020221120142445.png)

**Data transparency**: In theoery, the application should not know where the data is physically. In practice, the developers need to be aware of communication costs to avoid expensive data movement.

**Database partitioning**

DBMS executes query fragments on each partition and combines the results to produce a single answer. Can partition a database physically (shared nothing) or logically (shared disk).

<ins>Naive partitioning</ins> - A table per node. Assumes that each node has enough storage for each table. Ideal if queries never join across tables and access patterns are uniform.

<ins>Vertical partitioning</ins> - Split by attributes into partitions. Must store tuple information to reconstruct the original record. (Similar to a column store)

<ins>Horizonal partitioning (More common)</ins> - Split by partition key and scheme. Choose columns that divide the database equally (size, usage). Examples of partition schemes are hashing, ranges, predicates.

**Logical vs Physical**

<ins>Logical partitioning</ins>

Node communicates with other nodes that responsible for the IDs to retrieve the values.

![](images/Pasted%20image%2020221120145308.png)

<ins>Physical partitioning</ins>

Each shared nothing nodes read and update tuples stored on its own disk.

![](images/Pasted%20image%2020221120145254.png)

**Consistent hashing**

Even if new nodes are added, there is no need to reshuffe data by rehashing.

1. Hash the key into a range between 0 and 1.
2. Use the partition that is closest to the hashed key.

![](images/Pasted%20image%2020221120150129.png)

When a new node is added, only need to move data only from one partition to another.

![](images/Pasted%20image%2020221120150149.png)

**Replication factor**

Store the data in the closest three partitioning based on consistent hashing.

![](images/Pasted%20image%2020221120150345.png)

**Transaction coordination**: Required if the transaction access data on more than one partition.

**Centralized coordination**

<ins>TP monitor (Transaction Processing)</ins>

1. Application server requests coordinator for locks.
2. The coordinator acknowledges and the server can use the data.
3. The server requests to commit.
4. The coordinator ensures safe to commit at the partitions.
5. Transaction is committed.

![](images/Pasted%20image%2020221120151332.png)

<ins>Middleware approach</ins>

![](images/Pasted%20image%2020221120151551.png)

**Decentralized coordination**

1. The server sends a request to some node.
2. The node becomes the leader node, responsible to ensure whether transaction commits.
3. The server sends query to individual partitions.
4. The server requests to commit.
5. The leader ensures safe to commit at the partitions.
6. Transaction is committed.

![](images/Pasted%20image%2020221120151759.png)
