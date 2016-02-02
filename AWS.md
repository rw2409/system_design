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

## Query/Scan Items:

##Best practices
* When bulk uploading data, you can achieve higher throughput by spreading evenly across hash key values.
* Uniform Workload: generating requests evenly across your table in order to avoid throttling.
* Spread expensive Scan or Query requests out over time to avoid bursts


#S3
#SQS
#Kiniesis
#Redshift
