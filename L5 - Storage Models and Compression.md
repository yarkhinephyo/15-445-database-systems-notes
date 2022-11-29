**Different workloads**

<u>OLTP</u> - Fast operations which update/read small data. The application that people usually build first.

<u>OLAP</u> - Complex queries (read) with a lot of data for aggregates. Reads span multiple entries.

<u>HTAP</u> - OLTP and OLAP together.

**N-ary storage model (NSM)**

All attributes for a single tuple contiguously on a page. Querying a single attribute across multiple tuples will cause entire pages (with unnecessary attributes) to be brought in.

<u>Advantages</u> - Fast insert/update. Good for queries that need entire tuple.

<u>Disadvantages</u> - Not good for scanning subset of the attributes.

**Decomposition storage model (DSM)**

A single attribute across multiple tuples are stored contiguously in a page.

<u>Disadvantages</u> - Slow for point queries due to stitching.

**Tuple identification in DSM**

<u>Fixed-length offsets</u> - Every value is the same length. Same offsets in different columns match the same tuple.

<u>Embedded tuple IDs</u> - Each value is stored with its tuple ID. Instead of arithmetic to calculate offsets, another lookup is required for ID.

**Data compression**

Trade off is speed vs compression ratio.

Compression works because datasets tend to have skewed distributions (Items repeated many times). High correlation between attributes of the same tuple.

<u>Three goals</u>

- Must produce fixed length compressed data.
- Postpone decompression as much as possible. We want to operate on compressed data without decompressing first.
- Must be lossless. (Lossy compressions must be done at application level)

**MySQL InnoDB compression (Block Level)**

MySQL pages are 16kB so the compressed pages must be certain sizes. Certain changes to the data are added to the "mod log" (such as updates). If "mod log" is full, changes are applied.

![](images/Pasted%20image%2020220913124415.png)

The rest are column level compression.

**Run-length encoding**

Compress runs of the same value in a single column into triplets. (Value, Start position, Number of elements) Requires the columns to be <u>sorted</u> to maximize effectiveness.

**Bit-packing encoding**

Store data as smaller data size than declared. For example, storing LONGs as INTs.

**Mostly encoding**

Bit-packing variant that uses <u>special maker</u> to indicate when a value exceeds largest size and maintain a lookup table to store them.

**Bitmap encoding**

Stores a separate bitmap for each unique value. If two unique value -> two bitmaps. If the cardinality is very high, this encoding may not be appropriate.

**Delta encoding**

Record the difference between values that follow each other in the same column. Combine with RLE for even better compression.

**Incremental encoding**

Type of delta encoding that avoids common prefixes. Look at the prefix of the immediate previous value (Not from the beginning). Works well with sorted data.

**Dictionary encoding**

Maps variable length values to small integer identifiers. Most widely used scheme. Some queries require the original data, some queries only need the dictionary.

Strings will be compared as keys in the background which will make queries faster.

<u>Why not hash?</u> Dictionary needs to support encode and decode. Hash function will not work for decoding purpose.

<u>Order-preserving</u> - Encoded values need to support the same collation as original values (such as range query). This is addressed by sorting the dictionary via value between assigning the keys in order.