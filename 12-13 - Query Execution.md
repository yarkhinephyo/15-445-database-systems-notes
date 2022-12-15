**Processing models**

<u>Iterator Model (Most common)</u>

Each query plan operator implements a "Next" function. On each invocation, an operator implements a loop that calls next on its children to retrieve their tuples and process them. The operator itself returns either a single tuple or a null marker.

Also known as pipeline model. Allows a single tuple at top of query plan to process as much as possible before going to the next tuple.

![](images/Pasted%20image%2020221006130454.png)

<u>Materialization Model</u>

Each operator dumps out all the tuples anytime it is invoked. The DBMS can push down hints such as "LIMIT" to avoid scanning too many tuples.

The outputs can either be whole tuples (NSM) or subsets of columns (DSM).

![](images/Pasted%20image%2020221006131612.png)

Fewer function calls than the Iterator Model, which will speed up the processing. Good for OLTP workloads that access a small number of tuples at a time. Not suited for OLAP queries as large intermediate results will have to be spilled onto disk between operators.

<u>Vectorization Model</u>

"Next" function that returns batches instead of a single tuple. The size of batch depends on the hardware or query properties. Makes uses of SIMD instructions.

Ideal for OLAP because it greatly reduces the number of function invocations per operator (as compared to the normal iterator model).

**Access Methods**

The way DBMS access the data in a table.

<u>Sequential Scan</u>

For each page in the table, retrieve from the buffer pool and iterate over each tuple. The DBMS maintains cursor that tracks the last page and slot examined.

Optimizations - Prefetching (Pages into buffer pool), Buffer pool bypass (Side buffer for scan-like queries to prevent buffer pool pollution), Parallelization (Scan using multiple threads), Heap Clustering (Store pages in the order of primary key), Late Materialization (Delay stitching together tuples in DSM till upper parts of query plan), Approximate Queries (Lossy. Skip subsets of data to only approximate), Zone maps (Store summaries of each page to avoid checking if the value does not fit in)

<u>Index Scan</u>

DBMS has to try and recognize which index is more useful for a query if a query has multiple predicates over one table. In the example below, if there are only 2 people in the CS department, it will be best to scan using that index.

```
SELECT * FROM students
WHERE age < 30
	AND dept = 'CS'
	AND country = 'US'
```

In multi-index scans, DBMS can get record IDs for each index simultaneously and combine the sets based on the query's predicates such as union or intersect.

**Modification queries**

Operators that modify the database such as "INSERT", "UPDATE" are responsible for modifying the target table and the indexes.

<u>Update/Delete</u> - Child operators pass record IDs for target tuples.

<u>Insert</u> - Materialize tuples inside operator, or... operator inserts tuples passed in from child operators (better option).

**Update Query Problem (Halloween problem)**

Updating a query by removing from index and reinserting into index may continuously update the same tuple.

Anomaly where an update operation changes the location of the tuple and the scan operator visit the tuple multiple times.

<u>Solution</u> - Track modified record IDs.

![](images/Pasted%20image%2020221011164032.png)

**Expression evaluation**

"WHERE" clause as an expression tree. Each node represent expression types such as comparisons, conjunctions, arithmetic operators.

To evaluate the expression at runtime, the DBMS maintains a context handle that contains metadata for the execution such as the current tuple and the table schema.

![](images/Pasted%20image%2020221006141004.png)

High end systems do JIT evaluation on expressions. For example, predicate "1 = 1" will be compiled as true.

**Parallel Database System**: Resources are physically close to one another. Communication is assumed to be cheap and reliable.

**Distributed Database System**: Resources can be far from one another. Communication cost and problems cannot be ignored.

**Process models**

<u>Approach 1 - Process per DBMS worker</u>

Application hits dispatcher process which picks one of the worker processes to carry out the query.

Not optimal because we rely on OS scheduler for the order of workers. Shared memory is also required for global data structures which adds overhead. One good thing is that a process crash does not take down the entire system.

![](images/Pasted%20image%2020221025105046.png)

<u>Approach 2 - Thread per DBMS worker (More common)</u>

DBMS manages its own scheduling. Less overhead for context switch and does not have to manage shared memory.

The dispatcher thread can directly forward to another thread so that the outside application does not need to reconnect.

However, thread crash can kill the entire system.

![](images/Pasted%20image%2020221025105409.png)

DBMS decides where, when and how to execute it. How many tasks, which CPU cores to use, where to store the output.

**SQLOS**: User level OS layer that runs inside of DBMS that determines which tasks are scheduled onto which threads and manages I/O scheduling and database locks. The DBMS does not do any "syscall" directly. This allowed SQL server to be ported over to Linux from Windows.

<u>Scheduling of threads</u>

Non-preemptive thread scheduling, meaning the DBMS has to stop the threads after a quantum (4 ms) as the SQLOS cannot stop threads through interrupts.

![](images/Pasted%20image%2020221025110811.png)

**Process models (Continued)**

<u>Approach 3 -  Embedded DBMS</u>

DBMS runs inside the same address space as the application. (Typically, the applications communicate with DBMS which is on a separate process) Application is responsible for threads and scheduling.

![](images/Pasted%20image%2020221025111232.png)

**Inter- vs Intra-query parallelism**

<u>Interquery</u> - Execute multiple queries on separate workers. Very little coordination required if queries are read-only. If not, concurrency control protocols are required.

<u>Intraquery</u> - Execute the operations of a single query in parallel. Decreases latency for long running queries. This is done by executing query operators in parallel.

**Intra-query parallelism types**

<u>Approach 1 - Intra-Operator (Horizontal)</u>

Decompose operators into independent <u>fragments</u> that perform the same function on different subsets of data. The DBMS inserts an Exchange Operator that coalesce/split results from multiple children/parent operators. Does not exist in relational algebra.

![](images/Pasted%20image%2020221025113243.png)

The types of Exchange Operator are as follows. "Gather" combine the results from multiple workers. "Distribute" split a single stream from child into disjoint sets and send them to above. "Repartition" coalesce multiple input streams and produce multiple output streams (Different number of input and output streams).

In the example below, "Gather" is completed first before probing is done in parallel.

![](images/Pasted%20image%2020221025113919.png)

<u>Approach 2 - Inter-Operator (Vertical)</u>

One worker per operator so that the operators can run all the time. More common in streaming systems (continuous queries). Also known as Pipeline Parallelism.

<u>Approach 3 - Bushy parallelism</u>

Workers execute multiple operators from different segments of a query.

The diagram shows an example of four-way join.

![](images/Pasted%20image%2020221025114757.png)

Using additional threads in parallel would not help if the disk is the main bottleneck. If workers are accessing many parts of the disk, would introduce thrashing instead (Disk brings thing in and out of memory non-stop). 

**I/O Parallelism**

Split DBMS across multiple storage devices to improve bandwidth and latency. Configure OS/hardware to store the DBMS files across multiple storage devices. This is transparent to the DBMS.

**Database partitioning**

Split the database into chunks and assign them to different storage devices.

More popular. Split single table into disjoint physical segments that are stored separately.

**Parallel grace hash join**

Use a separate worker to perform the join for each level of buckets after partitioning.

![](images/Pasted%20image%2020221025112544.png)
