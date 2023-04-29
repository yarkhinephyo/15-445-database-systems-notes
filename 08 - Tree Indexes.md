**Table Index**: Replica of a subset of attributes stored in a way that allows for efficient access.

DBMS ensures that the contents and the index in sync. DBMS figures out the best indexes to use for each query.

<ins>Trade-off</ins> - Storage and maintenance overhead vs query time.

**B+Tree**

Optimized for systems that read and write large blocks of paper.

- Self-balancing
- Keeps data sorted
- Allows searches, sequential access, insertions, deletions. O(log(n))

- Perfectly balanced
- Every node is at least half-full
- Every inner node (non-leaf node) with k keys has k+1 non-null children

![](images/Pasted%20image%2020220921155941.png)

**B+Tree node**: Every node is a key/value pair. For inner node, value is a pointer to another node. For leaf node, value is the actual value of attribute. The arrays are kept in sorted key order so that binary searches can be performed.

Some DBMS store record IDs as the values. Some DBMS store the tuple data as the values.

![](images/Pasted%20image%2020220921160604.png)

**B Tree vs B+Tree**

B Tree stores values in all nodes in the tree. B+Tree only stores in leaf nodes (which means less efficient storage).

In practice, B+Tree is used because updates are cheaper with <ins>multiple threads</ins>. There is no need to traverse up and down nodes.

**B+Tree insert**

Find leaf node and insert in the sorted order. If there is no enough space, split the node at half way point into two nodes, then update the parent's node to include the middle key (Split may be recursive).

**B+Tree remove**

Find leaf node and delete the entry. If leaf node is less than half full, re-distribute from sibling. If re-distribution fails, merge the leaf node with its sibling and update the parent.

**Clustered indexes**

The table is sorted in the sort order specified by the primary key. The physical layout of tuples matches the order in the index.

Some DBMS will make a hidden row id primary key to use as clustered index if primary key is not specified. Some DBMS does not support clustered index (PostgreSQL).

**Selection conditions**

DBMS can use the B+Tree if query provides any attributes of the search key in the tree. For example if query is `a=5 and b=3` but there is only `b` in the index, it can still be used to narrow down the search.

This is unlike a <ins>Hash Index</ins> which requires all attributes to match the search key.

**Tuning B+Tree**

<ins>Node size</ins> - Larger node sizes for slower storage devices (Can be more than a page).

<ins>Merge threshold</ins> - Delaying a merge operation may reduce the amount of reorganization. May be better to let underflows exist and periodically rebuild the tree.

**Variable length keys**

- Store the keys as pointers to the tuple attribute. (Not relevant now)
- Variable length nodes, will require careful memory management. (Not relevant now)
- Pad the key to be max length of key type.
- (Key map) Embed an array of pointers (offsets within the node) that map to the key/value pairs stored with the node.

![](images/Pasted%20image%2020220921175229.png)

**Intra-node search**

- Linear search of keys from beginning to end.
- Binary search of keys (Need to ensure sorted order).
- Interpolation to approximate the location of the key then linear search.

**Optimizations**

<ins>Prefix compression</ins> - Since sorted keys have similar prefixes (near them), store the common prefixes once and unique suffices for each key.

<ins>Suffix truncation</ins> - Inner nodes do not need entire key. Only store a minimum prefix at inner nodes to correct route queries to the right leaf node.

<ins>Bulk insert</ins> - Build a B+Tree by sorting the keys and building the index from bottom up (Instead of adding keys one by one from the top). This is the fastest way to build a tree.

<ins>Pointer swizzling</ins> - Normally, the nodes use page_ids to reference other nodes. Buffer pool manager provides the memory location of the page. If pages are pinned in memory, raw pointers can be stored instead of page_ids. No need to go through the buffer pool manager anymore.