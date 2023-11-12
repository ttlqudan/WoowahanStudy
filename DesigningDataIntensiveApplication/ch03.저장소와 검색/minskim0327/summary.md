## Data Structures That Power Your Database

Think of the world's smallest database, with the two functions below.

```bash
db_set () {
    echo "$1,$2" >> database
}
db_get () {
    grep "^$1," database | sed -e "s/^$1,//" | tail -n 1
}
```
`db_set` is an append-only write function, thus providing a very efficient performance.
On the other hand, `db_get` has a terrible performance if you have a large number of records in your database. In algorithmic terms, the cost of a lookup is `O(n)`: if you double the number of records n in your database, a lookup takes twice as long. That’s not good.

In order to efficiently find the value for a particular key in the database, we need a different data structure: an *index*.
An index is an additional structure that is derived from the primary data. Many databases allow you to add and remove indexes, and this doesn't affect the contents of the database; it only affects the performance of queries.
Any kind of index usually slows down writes, because the index also needs to be updated every time data is written. Thus, there is an important tradeoff: well-chosen indexes speed up read queries, but every index slows down write queries.

### Hash Indexes
Let’s say our data storage consists only of appending to a file, as in the preceding example. Then the simplest possible indexing strategy is this: keep an in-memory hash map where every key is mapped to a byte offset in the data file—the location at which the value can be found,

This is an append only approach (log), and this technique is used by storage engines like `Bitcask`. This is well suited for workload with lots of writes, but not too many distinct keys.

How do we avoid eventually running out of disk space? Solution is `compaction`: throwing away duplicate keys in the log, and keeping only the most recent update for each key.

Since compaction makes segments much smaller, we can also merge several segments together at the same time as performing compaction, done in the background thread.
In order to find the value for a key, we first check the most recent segment's hash map, and  so on. One thing to note is that as writes are appended to the log in a strictly sequential order, a common implementation choice is to have only one writer thread.

An append-only log seems wasteful at first glance: why don’t you update the file in place, overwriting the old value with the new value? But an append-only design turns out to be good for several reasons:
- Appending and segment merging are sequential write operations, which are generally much faster than random writes, especially on magnetic spinning-disk hard drives.
- Concurrency and crash recovery are much simpler if segment files are append-only or immutable. For example, you don’t have to worry about the case where a crash happened while a value was being overwritten, leaving you with a file containing part of the old and part of the new value spliced together.

However, the hash table index also has limitations.
- The hash table must fit in memory, so if you have a very large number of keys, you’re out of luck.
- Range queries are not efficient. For example, you cannot easily scan over all keys between kitty00000 and kitty99999—you’d have to look up each key individually in the hash maps.

### SSSTables and LSM-Trees

In hash indexes, keys were not sorted. What if we require that the sequence of key-value paris is *sorted by key*?
This format is called *Sorted String Table*, or *SSTable* for short.

SSTables have several big advantages over log segments with hash indexs:
1. Merging segments is simple and efficient. If mergesort algorithms is used, we can also gurantee that the newly merged segment file is sorted.
If there are `n` segments, with average count of key-value pairs in each document as `m`, then the average time complexity is `mlg(n)`.
What if same key appears in several documents? We know that all the values in one input segment must be more recent than all values in other segment, so if multiple segments contain the same key, we can keep the value from the most recent segment.
2. Searching is more efficient, as you no longer need to keep an index of all keys in memory.
3. Allows compression, thus saving disk space and reducing the I/O bandwith use.

#### Constructing & Maintaing SSTables
1. When a write comes in, add to an in-memory tree data structure, called *memtable*.
2. When the memtable gets bigger than some threshold, write it out to disk as an *SSTable* file. While the **SStable** is being written out to disk, writes can continue to a new memtable instance.
3. Make read request, first from the memtable, then from the most recent disk segment, and so on.

This only suffers from one problem: if database crashses, the most recent writes in memtable are lost. In order to avoid this, we can keep a separate log on disk to which every write is immediately appended.

#### Making an LSM-tree out of SSTables.
The algorithm described here is essentially what is used in **LevelDB** and **RocksDB**. Simimlar storage engines are used in **Cassandra** and **HBase**, both of which were inspired by Googles' *BigTable paper*.
Lucene, an indexing engine for full-text search used by Elasticsearch and Solr, uses a similar method for storing its term dictionary

#### Performance Optimzations
The LSM-tree algorithm can be slow when looking up keys that do not exist in the database: you have to check the memtable, then the segments all the way back to the oldest (possibly having to read from disk for each one) before you can be sure that the key does not exist.

In order to optimize this kind of access, storage engines often use additional *Bloom filters*. A Bloom filter is a memory-efficient data structure for approximating the contents of a set. It can tell you if a key does not appear in the database, and thus saves many unnecessary disk reads for nonexistent keys.

There are also different strategies to determine the order and timing of how SSTables are compacted and merged. The most common options are size-tiered and leveled compaction. LevelDB and RocksDB use leveled compaction (hence the name of LevelDB), HBase uses size-tiered, and Cassandra supports both. 

In leveled compaction, the key range is split up into smaller SSTables and older data is moved into separate “levels,” which allows the compaction to proceed more incrementally and use less disk space. 

In size-tiered compaction, newer and smaller SSTables are successively merged into older and larger SSTables. 

### B-Trees
B-trees break the database down into fixed-size blocks or pages, traditionally 4 KB in size (sometimes bigger), and read or write one page at a time.
Each page can be identified using an address or location, which allows one page to refer to another—similar to a pointer, but on disk instead of in memory.

Some things to note are:
- `branching factor`: the number of references to child pages in one page of the B-tree
- updating value for an existing key: search for the leaf page containing that key, change the value in that page, and write the page back to disk
- add a new key: find the page whose range encompasses the new key and add it to that page. If there isn’t enough free space in the page to accommodate the new key, it is split into two half-full pages, and the parent page is updated to account for the new subdivision of key ranges
- These algorithms ensures that the tree remains **balanced**: a tree with *n* keys always has a depth of *O(lg n)*


#### Writes in B-Trees
The basic underlying write operation of a B-tree is to overwrite a page on disk with new data.

Think of overwriting a page on disk as an actual hardware operation. On a magnetic hard drive, this means moving the disk head to the right place, waiting for the right position on the spinning platter to come around, and then overwriting the appropriate sector with new data.

Some operations require several different pages to be overwritten. For example, if you split a page because an insertion caused it to be overfull, you need to write the two pages that were split, and also overwrite their parent page to update the references to the two child pages.
This is dangerous because if the database crashes after only some of the pages have been written, you end up with a **corrupted index**.

In order to make the database resilient to crashes, it is common for B-tree implementations to include an additional data structure on disk: a write-ahead log (**WAL**, also known as a redo log).
This is an append-only file to which every B-tree modification must be written before it can be applied to the pages of the tree itself.

An additional complication of updating pages in place is that careful concurrency control is required if multiple threads are going to access the B-tree at the same time—otherwise a thread may see the tree in an inconsistent state. This is typically done by protecting the tree’s data structures with **latches** (lightweight locks).

### Comparing B-Trees and LSM-Trees
LSM-trees are typically faster for writes, whereas B-trees are thought to be faster for reads. Reads are typically slower on LSM-trees because they have to check several different data structures and SSTables at different stages of compaction.
A B-tree index must write every piece of data at least twice: once to the write-ahead log, and once to the tree page itself
LSM-trees are typically able to sustain higher write throughput than B-trees, partly because they sometimes have lower *write amplification* (although this depends on the storage engine configuration and workload), and partly because they sequentially write compact SSTable files rather than having to overwrite several pages in the tree.
A downside of log-structured storage is that the compaction process can sometimes interfere with the performance of ongoing reads and writes.

An advantage of B-trees is that each key exists in exactly one place in the index, whereas a log-structured storage engine may have multiple copies of the same key in different segments. This aspect makes B-trees attractive in databases that want to offer strong transactional semantics:

### Other Indexing Structures
It is also very common to have secondary indexes.

#### Storing values within the index
The key in an index is the thing that queries search for, but the value can be one of two things: it could be the actual row (document, vertex) in question, or it could be a reference to the row stored elsewhere.

#### Multi-column indexes
The most common type of multi-column index is called a concatenated index, which simply combines several fields into one key by appending one column to another

Multi-dimensional indexes are a more general way of querying several columns at once, which is particularly important for geospatial data.

#### Full-test search and fuzzy indexes

Full-text search engines commonly allow a search for one word to be expanded to include synonyms of the word, to ignore grammatical variations of words, and to search for occurrences of words near each other in the same document, and support various other features that depend on linguistic analysis of the text.

## Transaction Processing or Analytics

- OLTP: online transaction processing
- OLAP: online analytic processing

|    | OLTP | OLAP |
| :-----: | :----: | :----: |
| Main read pattern | Small number of records per query, fetched by key | Aggregate over large number of records |
| Main write pattern | Random-access, low latency writes from user input | Bulk import (**ETL**) or event stream |
| Primarily used by | End user/customer, via web application | Internal analyst, for decision support |
| What data represents | Latest state of data | History of events that happened over time |
| Dataset size | GB ~ TB | TB ~ PB |

### Data Warehousing

Running sepearte database for analytics was called a *data warehouse*.

**OLTP** systems are usually expected to be highly available and to process transactions with low latency, since they are often critical to the operation of the business.
They are usually reluctant to let business analysts run ad hoc analytic queries on an OLTP database, since those queries are often expensive.

A data warehouse, by contrast, is a separate database that analysts can query to their hearts’ content, without affecting OLTP operations. The data warehouse contains a read-only copy of the data in all the various OLTP systems in the company. Data is extracted from OLTP databases (using either a periodic data dump or a continuous stream of updates), transformed into an analysis-friendly schema, cleaned up, and then loaded into the data warehouse. This process of getting data into the warehouse is known as Extract–Transform–Load (**ETL**).

### Stars and Snowflakes: Schemas for Analytics
In analytics, there is much less diversity of data models. Many data warehouses are used in a fairly formulaic style, known as a star schema.

A variation of this template is known as the snowflake schema, where dimensions are further broken down into subdimensions. For example, there could be separate tables for brands and product categories, and each row in the dim_product table could reference the brand and category as foreign keys, rather than storing them as strings in the dim_product table.

Snowflake schemas are more normalized than star schemas, but star schemas are often preferred because they are simpler for analysts to work with.

## Column-Oriented Storage
Although fact tables are often over 100 columns wide, a typical data warehouse query only accesses 4 or 5 of them at one time ("SELECT *" queries are rarely needed for analytics).

Let's assume we are trying to solve the following problem: *Analyzing whether people are more inclined to buy fresh fruit or candy, depending on the day of the week*

How can we execute this query efficiently?

From OLTP point of view: 
Have indexes on `fact_sales.date_key` and/or `fact_sales.product_sk` that tell the storage engine where to find all the sales for a particular date or for a particular product.

```sql
SELECT
  dim_date.weekday, dim_product.category,
  SUM(fact_sales.quantity) AS quantity_sold
FROM fact_sales
  JOIN dim_date    ON fact_sales.date_key   = dim_date.date_key
  JOIN dim_product ON fact_sales.product_sk = dim_product.product_sk
WHERE
  dim_date.year = 2013 AND
  dim_product.category IN ('Fresh fruit', 'Candy')
GROUP BY
  dim_date.weekday, dim_product.category;
```
But then, a row-oriented storage engine still needs to load all of those rows (each consisting of over 100 attributes) from disk into memory, parse them, and filter out those that don’t meet the required conditions.

The idea behind column-oriented storage is simple: don’t store all the values from one row together, but store all the values from each column together instead.

The column-oriented storage layout relies on each column file containing the rows in the same order.

### Column Compression
One technique that is particularly effective in data warehouses is bitmap encoding.

### Sort Order in Column Storage
It wouldn’t make sense to sort each column independently. Rather, the data needs to be sorted an entire row at a time, even though it is stored by column.
One advantage of sorted order is that it can help with compression of columns, and it can also help queries that need to group or filter sales by product within a certain date range.

### Writing to Column-Oriented Storage

The above optimzations, however, has downside of making writes more difficult. Fortunately, we have already seen a good solution earlier in this chapter: LSM-trees. All writes first go to an in-memory store, where they are added to a sorted structure and prepared for writing to disk. It doesn’t matter whether the in-memory store is row-oriented or column-oriented. When enough writes have accumulated, they are merged with the column files on disk and written to new files in bulk.

### Aggregation: Data Cubes and Materialized Views

One aspect of data warehouses that is worth mentioning briefly is materialized aggregates. As discussed earlier, data warehouse queries often involve an aggregate function, such as `COUNT`, `SUM`, `AVG`, `MIN`, or `MAX` in SQL. If the same aggregates are used by many different queries, it can be wasteful to crunch through the raw data every time. Why not **cache** some of the counts or sums that queries use most often?
One way of creating such a cache is a materialized view. A common special case of a materialized view is known as a data cube or OLAP cube.


The advantage of a materialized data cube is that certain queries become very fast because they have effectively been precomputed.

The disadvantage is that a data cube doesn’t have the same flexibility as querying the raw data. For example, there is no way of calculating which proportion of sales comes from items that cost more than $100, because the price isn’t one of the dimensions.

Most data warehouses therefore try to keep as much raw data as possible, and use aggregates such as data cubes only as a performance boost for certain queries.

