**Bifurcated environment**

![](images/Pasted%20image%2020221214211741.png)

**Star schema**: Main analytical queries are done against Fact Tables and supplementary information is joined from the Dimension Tables. One Fact Table with outer ring of Dimension Tables.

There is only one dimension of tables going out. For example, "product" and "category" instances are in the same table.

![](images/Pasted%20image%2020221129120243.png)

**Snowflake schema (More common now)**: Dimension Tables are normalized into more than one dimensions.

Snowflake schemas take up less storage space (with less redundant information) but incur consistency violations. Snowflake schemas may require more joins for a query.

![](images/Pasted%20image%2020221129120330.png)

**Join data from different partitions**

<u>Push model</u> - Send the query to the node which contains the data. Perform as much filtering as possible first. More common in a shared nothing system.

![](images/Pasted%20image%2020221129121906.png)

<u>Pull model</u> - Bring the data to the node executing the query. More common for a shared disk system.

 ![](images/Pasted%20image%2020221129121852.png)  
 
**Long running OLAP crash**
 
 For shared-nothing DBMSs, as soon as a node goes down, the query is aborted. 
 
 In shared-disk cloud servers with frequent snapshots to disks, if a node goes down, another can take over and continue the query.

**Query plan fragments**

1. <u>Physical operators</u> - A single query plan is generated and broken into partition-specific fragments.
2. <u>SQL</u> - Rewrite the original query into partition-specific queries. Local optimization is done at each node.

**Distributed join scenarios and algorithms**

Join algorithm involves two tables. Multiple scenarios can occur.

1. One of the two tables is already replicated at every node. No need to transfer data over the network before the join. Each node can join the data in parallel.
2. Both tables are partitioned on the join attribute. This also means no need to transfer data.
3. One table is not partitioned on the join key. If this table is small, it is broadcast over the network to all the nodes. Each node can now join the data in parallel.
4. Both tables are not partitioned on the join key. DBMS copies the table and shuffles around nodes such that data at each node have the same range of partition keys. Then the join algorithm is carried out.

**Semi-join**: Join query where the results only contain the columns from the left table.

**Managed DBMS**: No significant modification for a DMBS to be aware that it is running in a cloud environment.

**Cloud-native DBMS**: The DBMS is designed explicitly to run in a cloud environment. It is usually based on a shared disk architecture.

**Serverless database**: Shared-disk architecture. The compute nodes are only spinned up when necessary. If idle, the buffer pool and page tables are evicted to the disk before the node is shut down.

**Data lake**: Dump data without defining a schema or re-formatting them. In a catalog service, there is information about where to find the data. The data is only parsed during the query. This is ELT as opposed to ETL. Anyone can consume the data and transform to any format that they want instead of enforcing immediately.

![](images/Pasted%20image%2020221129125940.png)