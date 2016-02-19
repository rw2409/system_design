#System learning case:
## DocumentStore and Reporting
1. Data is saved to Document Store (Key-Value, DynamoDB).
  * Optimistic Locking
  * Send SNS event which goes into a SQS queue (order is not guaranteed, therefore downstream sequencing behavior on kinesis does not solve all ordering problems)

2. Mapping on start-up cache all these mapping configs
  * Schema to mappings: multi-map
  * OutputModel to destinations: multi-map
  * OuptputModel to outputFields: map
  * Implementation:
    * Use spring to find all Beans that are annotated/implemented with certain annotation or classes.
      * E.G: on start-up find all classes annotated with Muninn Mapping annotation therefore find all source schemas, destinations and output model classes.
    * Reflection can find all annotations on given class.
      * E.G: find all destination types for given Muninn Model classes

3. Mapping and sending to Kinesis Stream
  1. dequeue SQS messages and deserialize the message to retrieve document id.
  2. Call Document Store to retrieve document(both verion and latest version) as source, find all the mappings registered as beans (Spring) that accepts the document
  3. Runs mapping to generate output model POJO.
  4. For each output model POJO, send to Kinesis using documentID (Examples, RemovalOrder, RoutingDocument).
    * Each Document Type has its own KinesisStream, by default of 2 shards, this data is configurable.
    * What happens for dynamic stream size increase? It's fine as this is a blocking write, the messages will be in the source queue

4. Consuming Kinesis Stream
  1. Consumer as another ThreadPool.
  2. For each tuple of (Model class, destination) create a KinesisConnectorExecutor for class/destination with factory pattern
  3. Each KinesisConnectorExecutor(abstract class) is subclassed by S3/Redshift/ElasticSearch Executor
    * Each specific Executor create KinesisConnectorRecordProcessorFactory with specific pipeline and config
    * Each KinesisConnectorRecordProcessor is created by KinesisConnectorRecordProcessorFactory, the factory includes destination specific logic.
    * Each destination specific transformer, filter, emitter is defined in specific pipeline
    * The KinesisConnectorExecutor extends KinesisConnectorExecutorBase, which has a KCL Woker
    * Each KCL Worker syncs shard and lease information, tracking shard assignment and processing data from shards using RecordProcessor passed from KinesisConnectorRecordProcessor.
      * [Resharding/Scaling/Parallel Model](http://docs.aws.amazon.com/kinesis/latest/dev/kinesis-record-processor-scaling.html): each process(worker) has multiple shardConsumer for a shard.
      * [Status Tracking in DynamoDB](http://docs.aws.amazon.com/kinesis/latest/dev/kinesis-record-processor-ddb.html): each row is a shard.
      * [De-dups](http://docs.aws.amazon.com/kinesis/latest/dev/kinesis-record-processor-duplicates.html): first/last sequence schema for S3, elastic search ID and version always uses the latest sequence.

  4. KinesisMessageTransformer to deserialize from Record to KinesisMessage
    * KinesisMessage is a SharedType between consumer/producer, output model including JsonData
      * KinesisMessage(Kinesis producer of muninn svc) -(Json)-> Record(Kinesis) -(Json)-> KinesisMessage(muninn Kinesis Consumer)

  5. KinesisConnectorRecordProcessor<KinesisMessage, S3/Redshift/ElasticSearch Record> each includes pipeline and config, for pipeline:
    * Buffer: buffer pending steam records
      * Mostly uses BasicInMemoryBuffer 10M size
    * Filters: skip records that is not needed
      * all uses tombstone records
      * only elastic search generate tombstone records
    * Transformer (KinesisMessageTransformer mentioned earlier):
      * Transform from KinesisMessage to output record
      * RedshiftTransformer: KinesisMessage->byte[], use dynamoDB table to track last updated time.
      * Each transformer udpate ThreadLocal shardData with current KinesisMessage messageTime.
        * this messageTime is set to be local machine time while enqueing to Kinesis **Potential issue for time out of sync **
    * Emitter: emit destination record
      * RS actually leverages simple S3 emitter by chunck
      * Each emitter also has reference to TimeCompleteUpdator which reads data from ThreadLocal ShardData (shardId, consumerId and lastRecordTime) and update to DynamoDB table

    Also each KinesisConnectorRecordProcessor override pocessRecords function (List<Records> records, IRecordProcessorCheckpointer checkpointer) in that:
      * the processRecords function to set ThreadLocal data with KinesisConsumerName and shardID, (consumerName, ShardID) before processing any records.

5. Monitoring and alarming:
	* Input QueueSize alarming.
	* Kinesis QueueSize

6. Notes
  * Kinesis delete messages automatically, no need for client to delete them.
	* KCL uses DynamoDB to track each worker thread checkpoint into the stream, not related to our customization completion time check.
	* There is one thread on each host for each KinesisConsumer (model+destination), are different destination threads for the same model accessing the same shard of the model stream? However if shard number is smaller than total nodes, there will be some idle threads not accessing the shard.

