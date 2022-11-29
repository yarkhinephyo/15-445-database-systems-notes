**DBMS storage**

DBMS assumes that data is on non-volatile storage. DBMS components manage swapping in and out of volatile page.

<ins>Volatile</ins>: Random-access, Byte-addressable.

<ins>Non-volatile</ins>: Sequential access, Block-addressable. (Cannot jump to a single memory, must get the whole block first) Maximize sequential access.

**Disk-oriented DBMS**: Disk contains pages of data. Memory manages buffer pool, which fetches pages from the disk if necessary. Execution engine tries to get pages from buffer pool.

**Why not use OS memory mapping?**

If DBMS uses `mmap` to store contents of a file into the address space of a program, OS is responsible for swapping pages in and out. The swapping of pages may not be optimal for the database usage.

- Problem 1 - OS flushes dirty pages anytime, not necessarily when transaction is done.
- Problem 2 - DBMS does not know which pages are in memory.
- Problem 3 - Difficult to validate pages for `mmap`. If there is a problem, there will be `SIGBUS` (using `mmap`) which will have to be handled with signal handlers.
- Problem 4 - OS data structure contention.

**Storage Manager**: Responsible for maintaing a database's files. Tracks what threads are reading/writing at a given time and the available space.

**Hardware page**: Largest block of data that storage device can guarantee failsafe writes (atomic).

**OS Page (4 kB)**: Default size in linux and windows.

**Database page (512B - 16KB)**: Fixed sized <u>block</u> of data for the same type. Metadata of the page should be self-contained. Every page has a unique number so that DBMS can map them to the physical locations.

<ins>Choice of page size</ins> - Larger page will improve sequential access but the data may be at the back of the page.

**Database heap**: Unordered pages with tuples, stored in a random order. Database can locate a page on a disk given a page_id by using linked list of pages or a page directory.

<u>Linked list</u>:  Header page holds pointers to a list of free pages and a list of data pages. To look for a specific page, must sequential scan on the data pages.

<u>Page directory</u>: Special pages that track locations of pages and amount of free space on each page.  Free-space map tracks the free fraction of each page with a few bits for each page. If 3 bits are used for each page, "011" would mean 3/8 of the page is free.

**Slotted page layout**

Most common approach in DBMSs today. Slot array at the beginning, the data at the end of page.

<u>Page header</u> - Tracks number of used slots and the offset of the starting location of the last slot used.

<u>Slot array</u> - Maps slots to the tuples' starting position offsets.

<u>Insert new tuple</u>: Look for free page in page directory. Retrieve page from disk. Check slot array to find empty space.

<u>Update existing tuple</u>: Look for page directory to find location of page. Retrieve page from disk. Find offset in page with slot array. Overwrite existing data.

<u>Delete tuple</u>: The rest of tuples will be compacted. The time of compaction depends on the particular database type.

**Tuple**: Sequence of bytes. Up to the DBMS to interpret. Prefixed with a header that contains meta-data.

<u>Record ID</u>: Unique record identifier for each tuple. Typically page_id + slot.(PostgreSQL - `ctid` column contains page_id + slot pairs)

<u>Tuple header</u>: Contains metadata about tuple such as bit-map for NULL.

<u>Denormalized tuple</u>: Pre-join related tuples and store them in the same page. Updates are more expensive because updating one requires updating the other.

**Problems with slotted pages**

<u>Fragmentation</u> - Deletion of tuples can leave gaps.

<u>Useless disk I/Os</u> - Fetching undesired tuples in the same page from disk.

<u>Random disk I/Os</u> - Updating many tuples will cause random disk I/Os.

**Log-structured page layout**

<u>Log record</u>: Page identifier (Arbitrary), PUT record contains contents, DELETE record mark tuple as deleted.

DBMS stores log records that contain changes to tuples "PUT" and "DELETE" in memory. DBMS keeps appending records as application make changes. So every modification operation does not involve direct disk access -> lower latency.

To read a record, the database scans the log file backwards from newest to oldest.

When in memory page is full of records, it is written to disk asynchronously and becomes immutable.

<u>Read tuple</u>: Index that maps a tuple ID to location of newest log record (Tree index). Read that record.

**Log-structured compaction**: Consolidate multiple pages. Each tuple ID is guaranteed to appear at most once in the page.

<u>Sorted String Tables</u>: DBMS can sort the page based on tuple ID during compaction to improve future look-ups.

**Problems with log-structured approach**: Write-amplifcation. For a single local write of a tuple, there are many writes accumulated across compactions. Compaction is expensive.

**Tuple storage**: Sequence of bytes that only DBMS can interpret. Catalogs contain schema information about tables.

<u>Variable precision numbers</u>: Inexact variable-precision numeric type (FLOAT/REAL/DOUBLE). Uses C++ native types under the hood which discards significands from a certain distance after the decimal point. There may be unexpected rounding errors.

<u>Fixed precision numbers</u>: Numeric types with arbitrary precision and scale (NUMERIC/DECIMAL). Can be more expensive because the database would have to do things (tracking fixed precision) that hardware does not do.

<u>Dates and times</u>: Typically unit time sinces unix epoch.

**Large values**: Tuples larger than a page. DBMS uses separate overflow storage pages. Value in tuple larger than a page is stored as a pointer in the original tuple.

<u>External value storage</u>: Some systems allow large values to be stored in external file (blob). DBMS cannot manipulate the contents of an external file.

**System catalog (DBMS)**: Meta-data about databases in internal catalogs about tables, columns, indexes etc. Typically stored in the format that they use for the tables.
