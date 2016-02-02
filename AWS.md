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

##SNS Notification on updates.

##Best practices
1. With write > 100 or read > 300 RPS see (details)[http://docs.aws.amazon.com/AmazonS3/latest/dev/request-rate-perf-considerations.html]
  1.  If mix requests types, use proper name prefix (not date, sequence etc), as internally S3 sort key names using lexical order. Potential overwhelming the I/O capacity.
    * add hash prefix before sequence/date etc, not good for range query.
    * add group + hashprefix
    * reverse the sequence number as prefix.
  1.  If bulk read, swich to use 'CloudFront' CDN.

#SQS

#Kinesis

#Redshift