Requirements:For web applications, it is common to have a large number of servers running the same application, with a load balancer in front to distribute the incoming requests. In this scenario, we want to check and alarm in case an exception is thrown in any of the servers. We want a system that checks for appearance of specific words, "Exception", "Disk Full" etc. in the logs of any of the servers. How would you design this system?

# High level architecture
Four tiers split:
 1. log entry mapper/generator
 2. log entry aggregation/reducer
 3. metrics data persistance layer
 4. application layer to metrics persistance layer to take actions

* use Streaming/Messaging (Kafka, Kinesis) queue based async data publishing to connect tier 1 and tier 2.
* Each layer can use horizontal scaling with increasing number of nodes

## log/Metrics publishing layer: light weight publisher to publish log/metrics entry
    {
        key : ServiceName,
        timeStamp: currentTime
        payload: {
            hostName: "foo",
            metricName: "bar",
            logEntryValue: "logEntry"
        }
    }
  * There should be a local agent that constantly tailing/grepping logs for the key words and publishes messages.
  * The local agent can also generate system metrics such as CPU Usage, disk Usage, service metrics etc and publish to the queue.
  * Potentially, the log file is too big to transfer over the stream.
  * Local agent such as scripts, Logstash

## log consuming layer
this part should dequeue messages, and publishes to certain consumers, for different type of payLoad, emit different data events or actions.
* Certain data such as metrics may need to run aggregation for the smallest time granularity such as 'Minute'

## metrics data persistance layer
* The persistant layer could potentially use key-value store such as Cassandra or Dynamo with primary key as metrics key, and time-interval as sort key
* Certain entries or events can also be translated to Jason and persisted in ElasticSearch, indexed.

## Log application layer
* Ticket/Alarming system can scheduled to query specific metric/logscan entry and cut alarms.
* Advanced log monitoring/visualization could be done using Kibana, querying elastic search
