# Partitioning
The main reason for wanting to partition data is **scalability**. Different partitions can be placed on different nodes in a shared-nothing cluster. Thus, a large dataset can be distributed across many disks, and the query load can be distributed across many processors.
Thus, one goal of partitioning is to spread the data and the query load evenly across nodes.


## Partitioning and Replication
Partitioning is usually combined with replication so that copies of each partition are stored on multiple nodes.

## Partitioning of Key-Value Data
If the partitioning is unfair, so that some partitions have more data or queries than others, we call it *skewed*.

`hot spot`: A partition with disproportionately high load

Let’s assume for now that you have a simple key-value data model, in which you always access a record by its primary key.

### Partitioning by Key Range
One way of partitioning is to assign a continuous range of keys (from some minimum to some maximum) to each partition.
The ranges of keys are not necessarily evenly spaced, because your data may not be evenly distributed.

- Range scans are easy, and you can treat the key as a concatenated index in order to fetch several related records in one query
- Certain access patterns can lead to hot spots. If the key is a timestamp, all the writes end up going to the same partition (the one for today), so that partition can be overloaded with writes while others sit idle

### Partitioning by Hash of Key
Because of this risk of skew and hot spots, many distributed datastores use a hash function to determine the partition for a given key.

Once you have a suitable hash function for keys, you can assign each partition a range of hashes (rather than a range of keys), and every key whose hash falls within a partition’s range will be stored in that partition.

However, we lose the ability to do efficient range queries.

#### Concatenated Index Approach
`Cassandra` achieves a compromise between the two partitioning strategies. A table in Cassandra can be declared with a **compound primary key** consisting of several columns. Only the first part of that key is hashed to determine the partition, but the other columns are used as a concatenated index for sorting the data in Cassandra’s SSTables.

The concatenated index approach enables an elegant data model for one-to-many relationships. For example, on a social media site, one user may post many updates. If the primary key for updates is chosen to be `(user_id, update_timestamp)`, then you can efficiently retrieve all updates made by a particular user within some time interval, sorted by timestamp.

### Skewed Workloads and Relieving Hot Spots
Hashing a key to determine its partition can help reduce hot spots. However, it can’t avoid them entirely: in the extreme case where all reads and writes are for the same key, you still end up with all requests being routed to the same partition.

Today, most data systems are not able to automatically compensate for such a highly skewed workload, so it’s the **responsibility of the application to reduce the skew**.

One solution is to add a random number to the end of the key.
- Just a two-digit decimal random number would split the writes to the key evenly across 100 different keys, allowing those keys to be distributed to different partitions.
- However, this increases the load for read requests, as data with same user are split across different key

## Partitioning and Secondary Indexes
What if **secondary indexes** are also involved?

A secondary index usually doesn’t identify a record uniquely but rather is a way of searching for occurrences of a particular value:
- find all actions by user 123
- find all articles containing the word hogwash
- find all cars whose color is red, and so on.

### Partitioning Secondary Indexes by Document
Imagine you are operating a website for selling used cars.

Each listing has a unique ID — `document ID` — and you partition the database by the document ID.

What if you want to let users search for cars, allowing them to filter by `color` and by `make`? You need a `secondary index` on color and make.

If you have declared the index, the database can perform the indexing automatically.
For example, whenever a red car is added to the database, the database partition automatically adds it to the list of document IDs for the index entry `color:red`.

Downside:
- If you want to search for red cars, you need to send the query to all partitions, and combine all the results you get back.

### Partitioning Secondary Indexes by Term
Rather than each partition having its own secondary index (a **local index**), we can construct a **global index** that covers data in all partitions.

With the **term-partitioned** indexing, read is more efficient, but writes are slower and complicated. Because of this, updates to global secondary indexes are often asynchronous.

## Rebalancing Partitions

`Rebalancing`: The process of moving load from one node in the cluster to another.

### Strategies for Rebalancing

#### How not to do it: hash mod N

When partitioning by the hash of a key, it's best to divide the possible hashes into ranges and assign each range to a partition.

***Why can't we just use mod?***

The problem with the mod N approach is that if the number of nodes N changes, most of the keys will need to be moved from one node to another.

```
hash(key) = 123456 -> mod 10: 6, mod 11: 3
```
We need an approach that doesn’t move data around more than necessary.

#### Fixed number of partitions
Fortunately, there is a fairly simple solution: create many more partitions than there are nodes, and assign several partitions to each node.

If a node is added to the cluster, the new node can steal a few partitions from every existing node until partitions are fairly distributed once again. Note that 
- The **entire** partition should be moved between the nodes.
- The number of partitions does not change, nor does the assignment of keys to partitions.
- The change of assignment is not immediate: it takes some time to transfer a large amount of data over the network

#### Dynamic Partitioning
For databases that use key range partitioning, a fixed number of partitions with fixed boundaries would be very inconvenient.

If you got the boundaries wrong, you could end up with all of the data in one partition and all of the other partitions empty.

For that reason, key range–partitioned databases such as HBase and RethinkDB create partitions dynamically
- When a partition grows to exceed a configured size, it is split into two partitions so that approximately half of the data ends up on each side of the split.
- If lots of data is deleted and a partition shrinks below some threshold, it can be merged with an adjacent partition.

A caveat is that an empty database starts off with a single partition, since there is no a priori information about where to draw the partition boundaries.

To mitigate this issue, `HBase` and `MongoDB` allow an initial set of partitions to be configured on an empty database (this is called **pre-splitting**). In the case of key-range partitioning, pre-splitting requires that you already know what the key distribution is going to look like.

#### Partitioning proportionally to nodes
`Cassandra` and `Ketama` make the number of partitions proportional to the number of nodes. In other words, to have a fixed number of partitions per node.

This approach also keeps the size of each partition fairly stable.

### Operations: Automatic or Manual Rebalancing
***Does the rebalancing happen automatically or manually?***

There is a gradient between two approaches:
- automatic rebalancing: the system decides automatically when to move partitions from one node to another, without any administrator interaction
- fully manual : the assignment of partitions to nodes is explicitly configured by an administrator, and only changes when the administrator explicitly reconfigures it
 
## Request Routing

There remains an open question: 
***When a client wants to make a request, how does it know which node to connect to?***

This is an instance of a more general problem called service discovery, which isn’t limited to just databases.

There are a few different approaches to this problem.

1. Allow clients to contact any node. If the node doesn't own the partition, forward the request to another node. (`Cassandra`, `Riak`)
2. Send all requests from clients to a routing tier first, which determines the node that should handle each request and forwards it accordingly. (`Helix`, `HBase`, `SolrCloud`, `Kafka`)
3. Require that clients be aware of the partitioning and the assignment of partitions to nodes.

Many distributed data systems rely on a separate coordination service such as *ZooKeeper* to keep track of this cluster metadata.

Each node registers itself in ZooKeeper, and ZooKeeper maintains the authoritative mapping of partitions to nodes. Other actors, such as the routing tier or the partitioning-aware client, can subscribe to this information in ZooKeeper.

### Parallel Query Execution

`Massively parallel processing` (MPP) relational database products, often used for analytics, are much more sophisticated in the types of queries they support.

The MPP query optimizer breaks this complex query into a number of execution stages and partitions, many of which can be executed in parallel on different nodes of the database cluster.
