**Design decisions for data structures**

<ins>Data organization</ins> - How to layout data structure in pages to support efficient access.

<ins>Concurrency</ins> - How to enable multiple threads to access the data structure.

**Hash table**

<ins>Hash function</ins> - Map large key space to smaller domain. We do not want a cryptographic hash functions because they are slower.

<ins>Hashing scheme</ins> - Handle key collisions. Trade-off between large hash table (memory) vs time (compute) to find and insert keys.

**Static hash table**: Giant array with one slot for every element. Just mod the key by number of elements to get the slot number. Assumes perfect hash function.

**Linear probe hashing**: Giant table of slots that resolve collisions by linearly searching for the next free slot in the table. Store key in the index to know when to stop scanning.

<ins>Insert</ins> - Hash the key and insert at the first empty slot.

<ins>Delete</ins> - Approach 1 is to store tombstone marker so that finding a key will not terminate at this slot. Approach 2 is to move up the items after the deleted slot (Need to check hashed keys each time).

**Non-unique keys**: Hash table that can have multiple key-value pairs.

<ins>Separate linked list</ins> - The hash slots point to linked lists corresponding to each key.

<ins>Redundant keys</ins> - Store duplicate key entries together in the hash table. Lookup will be scanning till an empty slot.

**Robinhood hashing**: Variant of linear probe hashing that steals slots from "rich" keys and give them to "poor" keys.

Track the distance of the key-value pair from the original hashed slot position. The key-value pair with higher distance value replaces the position of a pair with lower distance value.

![](images/Pasted%20image%2020220917143721.png)

**Cuckoo hashing**

Multiple hash tables with different hash function seeds.

If no table has free slot, evict element from one of them and re-hash it in another table. If there is a cycle of eviction and rehashing, resize table.

**Dynamic hash tables**: Resize themselves on demand.

**Chained hashing**: Linked list of buckets for each slot in the hash table.

**Extendible hashing**: Split buckets instead of letting buckets grow forever. Multiple slots can point to the same bucket chain.

Slots hold pointers (page_id) to buckets (pages). Global counter tracks how many bits of hashed key is needed to find a slot. Local counter tracks the common bits for each key in the bucket (<= global counter).

If bucket has no more space, increment the local counter and split the keys according to the new common bits. If local counter is more than global counter, double the global counter and resize the slot array.

More explanation of algorithm [here](https://youtu.be/r4GkXtH1la8).

<ins>Implementation</ins> - In practice, if you want buckets to be durable, can allocate a page in the buffer pool which can act as a bucket.

<ins>Problem</ins> - During the doubling of table during extendible hashing, requires a latch on entire slot array which will prevent other threads from accessing the table (bottleneck).

**Linear hashing**: Linear hashing builds on top of chained hashing. Localize the resizing to the splitting bucket instead of doubling the entire array.

<ins>Insert</ins> - Maintains a pointer that tracks the next bucket to split. If <ins>any</ins> bucket overflows, split it at pointer location. Add an entry to the slot array (pointing to the splitted bucket). Create a new hash function to find the location of new entry.

![](images/Pasted%20image%2020220917160031.png)

<ins>Find</ins> - If the hash slot position is less than the split pointer, it means the bucket has been split. Requires using other hashing functions to find the other buckets.

<ins>Delete</ins> - Can do incremental deletes but in practice may be faster to rebuild.
