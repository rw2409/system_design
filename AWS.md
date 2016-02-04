#Dynamo DB
## Tables
1. Primary Key
  1. Simple (PartitionKey Key): PartitionKey used as hashfunction input, has to be unique
  2. Composite (PartitionKey, SortKey): **PartitionKey** used as hashfunction input, items with same hashkey stored together, but must have different **SortKey**
[Details on item distribution](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DataModel.html#DataModel.Partitions). 

2. Read/Write Capacity
  1. Read Capacity Unit: The number of strongly consistent reads per second of items up to 4 KB in size per second. For eventually consistent reads, one read capacity unit is two reads per second for items up to 4 KB.
  2. The number of 1 KB writes per second.

Capacity increases in proportion with item size, read/write request rate, consistency(strong read = 2*eventually consistent read), local secondary index will also consumes read/write capacity. [Details](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/WorkingWithTables.html)

## Items
* Collection of attribute names and values
* No need to define attribute names and data types while table creation.
* Each item max size 400KB
* Attribute data types:
  * Scalar types – Number, String, Binary, Boolean, and Null.
  * Document types – List and Map.
  * Set types - String Set, Number Set, and Binary Set.

## Secondary Indexes
Secondary index consists of attributes that are projected, or copied, from the table into the index.
1. Global secondary index: an index with a partition key and sort key that can be different from the table
  1. "global" because queries on the index can span all of the data in a table, across all partitions.
2. Local secondary index: an index that has the same partition key as the table, but a different sort key
  1.  "local" in the sense that every partition of a local secondary index is scoped to a table partition that has the same partition key value
[See Detail Comparisons](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/SecondaryIndexes.html)

## Read/Write Items:
* GetItem: by default eventually consistency read, set consistentRead to be true for strong consistency read
* PutItem:
* UpdateItem:
* DeleteItem:
* BatchGetItem: max 16MB data or 100 items
* BatchWriteItem: max 16MB data or 25 items
* Atomic Counters
* **Conditional Update(idempotent): specify the condition(s) in the ConditionExpression parameter.** 
(OptimisticLocking Implementation)[http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DynamoDBMapper.OptimisticLocking.html]

##Examples
* DocumentStore (Dynamo DB + S3)\
* Potentially one revision (S3 object) can be split into multiple Dynamo DB items. And use two extra table (item and revision (with a set of primary keys of individual items)).

##Best practices
* When bulk uploading data, you can achieve higher throughput by spreading evenly across hash key values.
* Uniform Workload: generating requests evenly across your table in order to avoid throttling.
* Spread expensive Scan or Query requests out over time to avoid bursts

#S3
##Buckets
* Bucket name cannot be reused.
* 100 buckets per accounts
* lives in the region specified
* DNS naming conventions
* not create/delete buckets in high availability code path
* SNS Notification on updates.

##Best practices
1. With write > 100 or read > 300 RPS see (details)[http://docs.aws.amazon.com/AmazonS3/latest/dev/request-rate-perf-considerations.html]
  1.  If mix requests types, use proper name prefix (not date, sequence etc), as internally S3 sort key names using lexical order. Potential overwhelming the I/O capacity.
    * add hash prefix before sequence/date etc, not good for range query.
    * add group + hashprefix
    * reverse the sequence number as prefix.
  1.  If bulk read, swich to use 'CloudFront' CDN.

#SQS
##Properties
* multiple readers/writers
* SQS message size 256KB, bigger data could store pointer to S3 object
* At least once delivery, consumers needs idempotent
* message order is **NOT guaranteed**, needs version number, sequence information or timestamp
* delay queue to be used to postpone messages
* short polling behavior: samples a subset of servers. (subset of all messages returned)

##Concepts
* QueueName: URL
* MessageID: generated upon sendMessage() response, char(100), can be used to identify message but not delete
* Receipt Handle: char(1024)
  * Each time you receive a message from a queue, you receive a receipt handle for that message. 
  * The handle is associated with the act of receiving the message, not with the message itself.
  * To delete the message or to change the message visibility, you must provide the receipt handle and not the message ID. 
  * This means you must always receive a message before you can delete it (you can't put a message into the queue and then recall it).
* Visibility Timeout
  * SQS does not delete message after received, instead it blockes the message with visibility timeout
  * 120,000 limit of inflight messages (received but not yet deleted)
  * If the message has different visibility timeout, we can forward to multiple queues with different timeout
* Retention period: messages automatically deleted after retention period.
* Delay Queue:  Delay queues allow you to postpone the delivery of new messages in a queue for a specific number of seconds.
* Dead Letter Queue: A dead letter queue is a queue that other (source) queues can target to send messages that for some reason could not be successfully processed.
  * A primary benefit of using a dead letter queue is the ability to sideline and isolate the unsuccessfully processed messages. 

#Kinesis
##Use cases
Real-time aggregation of data followed by loading aggregated data into DW/Map-Reduce cluster
  * log data
  * social media/market data feeds
  * click stream

Kinesis ensures durability and elasticity
 * near realtime 1s available after data intake
 * no data loss prior to expiration
 * multiple data consumers supported

##Concepts
* Kinesis Stream
  * ordered sequence of data records
  * each record in the stream has a sequence number
  * composed of one or more shards, each of which provides a fixed unit of capacity
  * add/remove shards to increase/decrease with capacity

* Shards
  * capacity: 5 reads or 2 MB per second
  * 1000 writes or 1 MB per second

* Data Record
  * partition key: byte(256), MD5 hash(parition key)->128 bit integer to map to shards
  * sequence number: each data record has a unique sequence number, sequence number for the same parition key generally increase over time
    *The sequence number is assigned by Streams after writing to the stream with client.putRecords or client.putRecord
  * A data blob (Immutable) can be up to 1MB

* Retention Period
  * default 24 hours, max 7 days

* Producers: Data generator
* Consumers: Kinesis Stream applications
	* Each consumer must have unique name that is scoped to the AWS account and region
	* The name is used to create a DynamoDB table to store application state

#Redshift
## High level architecture: Cluster of nodes
* Leader node: receives queries from client, parse queries, develops query execution plans
* Leader node then coordinates the parallel execution of these plans with the compute nodes, aggregates the intermediate results from these nodes, and finally returns the results
* Compute nodes execute the query execution plans and transmit data among themselves to serve these queries.

## DB type: PostgreSQL
* online analytic processing (OLAP) and  business intelligence (BI) oriented
* E.G, online transaction processing (OLTP) applications typically store data in rows, Amazon Redshift stores data in columns, using specialized data compression encodings for optimal memory usage and disk I/O. 
* E.G, Secondary indexes and efficient single-row data manipulation operations are disabled
* Non implemented features: indexes, foreign keys, constraints, triggers, sequences etc

# Distributed Key-value store options: Dynamo DB/Redis/HBase/Cassandra
**CAP Theroem** : trading off about C(Consistency) A (Availability) and P (Partition Tolerance)
When network parition happens:
CP: CP system will continue running with full consistency; it will sacrifice A by shutting down other (non-quorum) nodes entirely

## Dynamo DB
* Pros: hosted, seamingless scale, no need to worry about consistency, availability, scalability
* Cons: expensive, dependency on AWS

## Build high scalable key-value storage with Shareded Redis Cluster
1. Single Node to load everything in RAM
    * -Issue1: No redundancy, single point of failure
    * -Issue2: Not able to host all data
2. Issue1: Master-Slave replica
    * +both master/slave can do non-blocking sync-up, keep serving traffic while syncing
    * -Potential problem of consistency when network partition happens
    * ?Any way to recover/promote a slave
3. Shard/Partition the data to different Redis clusters
    * + In theory can support infinite data
    * + Need to implement consistant hashing to reduce re-map effort (Details? in implementing consistant hashing: what hash function?)
    * 2015 ready: Redis Cluster Solution

## Hbase: Big table on HDFS(GFS)
  * Non-Relational database run on top of HDFS(Hadoop Distributed File system)
  * Implementation of BigTable including bloom filter, compression, in-memory operation
  * Non-Structured, column database
  * Good for storing large quantities of sparse data (find 50 max in a billion records)
  * Tables in Hbase can be used as input/output of MapReduce jobs
  * CP system in CAP

### Original Idea: BigTable:
* Based on GFS (Google File System)

### Use examples:
1. Facebook messaging system
2. Big Table: Google Reader, Maps, SearchHistory etc

### [Data Model](http://hbase.apache.org/book.html#conceptual.view)
[Good analogy](http://jimbojw.com/wiki/index.php?title=Understanding_Hbase_and_BigTable)
1. Table: Sorted Map
  1. Row Key : only search index in the table, usually reversed for domain name to keep locality
  2. Support single index, range by timestamp, or scan
  3. Sorted by byte array values
2. Column family: Multi-dimensional value Map
  1. Column families are specified when created, cannot be modified, Expensive to add new column families
  2. Each Column family may have any number of columns, denoted by a column "qualifier" or "label".
  3. When asking HBase/BigTable for data, you must provide the full column name in the form "<family>:<qualifier>". 
  4. Each row may have any number of different columns, there's no built-in way to query for all columns in all rows.
3. Timestamp:
  * Last dimension represented in HBase/BigTable is time. 
  * All data is versioned either using an integer timestamp (seconds since the epoch), or another integer of your choice.
4. Cell: a cell is identified by its rowkey/column pair
Each column family may have its own rules regarding how many versions of a given cell to keep. In most cases, applications will simply ask for a given cell's data, without specifying a timestamp. HBase/BigTable will return the most recent version (the one with the highest timestamp) since it stores these in reverse chronological order. 

### [Physical Model](http://hbase.apache.org/book.html#regions.arch)
1. Region: a continous range of data in table, smallest distributed/load balancing unit
  * A table starts with one region and grow to split
  * Smallest store (HRegions) and load balancing unit: differnt HRegions could be stored on different HRegionServers, but the same HRegion will not split
2. Store: smallest store unit, one HRegion have 1+ Stores, each store contains all data of a column family
  * Each Store is one memStore and 0 or more Store File
  * Store file is HFile(B Tree) on HDFS

3. HRegionServer = HBase RegionServer
  * One physical node runs **exactly one** HRegionServer
  * One HRegionServer manages **mulitple** HRegions
  * One HRegion instance has HLog and multiple Store

4. HLog: Each HRegionServer manages one HLog, logs from differnt HRegion are commingled,  WAL(Write Ahead Logging) before writing
  * Single HLog good for write
  * Whe failover, need to split logs for differnt HRegions

### W/R operation
  1. Write Sequence:
    1. Connect to HRegionServer to submit write
    2. Write Ahead Log (WAL) and memstore
    3. When memstore reaches certain threshold, HRegionServer will flashcache into StoreFile
    4. Flash file may grow and merge and finally HRegion will split to regions.
    5. HMaster will assign new HRegion Server
  2. Read operation:
    1. first check memStore
    2. if not in check StoreFile

### HMaster:
  * One single master of a HBase Cluster
  * HMaster will load balancing between Region Server
  * HMaster never servers outside traffic, only HRegionServer answers
  * When HRegion is down, HMaster will notice by ZooKeeper and process corresponding HLog
  * When HMaster is down, HRegion can still work but MetaData cannot be udpated.
  * [Master/Slave Model but not every request needs HMaster](https://blogs.apache.org/hbase/entry/hbase_who_needs_a_master)

##Cassandra?
##[About meta data](http://blog.csdn.net/liuaigui/article/details/6749188)

#Open source projects:
* Thrift: Thrift is an interface definition language and binary communication protocol that is used to define and create services for numerous languages.
* Hive(SQL): BI Reporting SQL
* Pig: ETL Tools (Data flow)
* ZooKeeper: Scheduling/Coordinating between MapReduce jobs

#Kinesis difference & Kafka & Storm

#What is good for Redis/HBase/Cassandra/Dynamo?
#TODO:
##Kinesis code read
##SWF, how to define workflow, SWF code read
##Spring MVC
