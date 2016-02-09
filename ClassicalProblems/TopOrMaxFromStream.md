# Max/MostFrequent K items in stream (Kinesis/Kafka)
The stream message should include:
1. PrimaryKey (which could be or not be the identifier of 'item')
2. Payload
3. Timestamp


## Stream implementation: using sequence append log both for Kinesis/Kafka

## Processing pattern:
1. All time update: use key-value table to track all time statistics, have adder/remover to react to message append/delete
  * The table could only store keys with count greater than certain threshold, as most items should have very small count values.
2. Time Interval Based: use bath processing to generate result in batch

## Scale:
1. If the data stream is not so large that one host can process within SLA, it could be done on one host of a hostclass.
(using heap for max/min problem, use map for top count problem)

2. If one host cannot process fast enough due to:
  * Too many data points, cannot process fast enough
  * Too many data, cannot load everything into memeory, has to be swapped with file system based or external persistency layer
We can vertically scale by computing/resource improvement.
We can also horizontally scale using
  * Split/shard the stream by shard/parition using its primary key if primary key is the term/id we are sort/count by
  * Redirect to a second layer of streams using mapped key if we need to map the key.
  * For each shard/parition, we can have worker to pull data from that specific shard
  * If data in the shard increase big enough, we just split the shard and have more hosts to work with each shard

## Reducing and generate results
1. For max/min K problem, each worker can finish its local buffer batch and update the reduce table.
  * If the key is pretty sparse, depends on the business usecase we may able to store only the keys with reasonable big numbers and purge the small items.
2. For top K problem, each worker can finish its local buffer and produce its count output in output files or a key-value table.
