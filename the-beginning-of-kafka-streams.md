# The beginning of Kafka Streams

## The beginning of Kafka Streams

KIP -  [Kafka Improvment Proposal](https://cwiki.apache.org/confluence/display/KAFKA/Kafka+Improvement+Proposals). Every new feature, every public API change must have KIP so Kafka community can discuss pros and cons of it. [KIP-28 Add a processor client](https://cwiki.apache.org/confluence/display/KAFKA/KIP-28+-+Add+a+processor+client) was a KIP that started Kafka Streams.

Cumbersome consumer, tricky producer

{% embed data="{\"url\":\"https://gist.github.com/wojda/7071eada07753b9bc6b8a8a2e45b000b\#file-kafka-streams-from-scratch-vanilla-consumer-and-producer-scala\",\"type\":\"rich\",\"title\":\"Kafka Streams from scratch - vanilla consumer and producer.scala\",\"description\":\"Kafka Streams from scratch - vanilla consumer and producer.scala · GitHub\",\"icon\":{\"type\":\"icon\",\"url\":\"https://gist.github.com/fluidicon.png\",\"aspectRatio\":0},\"thumbnail\":{\"type\":\"thumbnail\",\"url\":\"https://avatars3.githubusercontent.com/u/3869128?s=400&v=4\",\"width\":400,\"height\":400,\"aspectRatio\":1},\"embed\":{\"type\":\"reader\",\"html\":\"<script type=\\\"text/javascript\\\" src=\\\"https://gist.github.com/7071eada07753b9bc6b8a8a2e45b000b.js\\\"></script>\",\"aspectRatio\":0}}" %}



## Kafka Streams DSL 

### Processor API

### High Level DSL

1. Provides higher-level operations on the data
2. New DSL - consume -&gt; transform \(map\) -&gt; produce
3. Single thread - simplicity
4. Partitioning - Scalability by sharding data \(Kafka koncept\)
5. Resiliency, Fault Tolerance - Commiting offset, Consumer rebalancing \(Kafka concepts\)

## Stateful operatation - Join unbounded data sets

1. The root of all evil - state
   1. external database
   2. local state
   3. memory vs file system
   4. RocksDB
2. Local state FTW
   1. Logging - Kafka topic as a backup \(Kafka concept\)
   2. `Standby replicas` - reduce time of fail-over 
3. Windowing
4. At least once delivery guarantee
   1. don't start until restored
   2. commit offset when producer acked \(Kafka concept\)
5. Resiliency, Fault Tolerance \(Kafka concepts\)
6. Scalability
   1. Partition everything! \(Kafka concept\)
   2. Dark side of Kafka Scalability - Rekeying

## Stateful operation - lookup table

1. Source of enrichments - kafka topic
2. KTable
   1. Deploying new instance - don't start until restored
3. Global KTable
4. Source of enrichments - database, http service
5. Async calls in Kafka Streams
   1. Async calls == Blocking calls
   2. At least once delivery guarantee
   3. Failure tolerance

## Http Api for Materialized View

Kafka Streams allows you query any local state you create in your topology. Add your favourite Http server and expose that state via Http Api.



## Production Readiness

1. Observability
   1. Kafka Clients metrics - jmx 
   2. Custom Kafka Streams metrics - jmx
   3. Issue with stateful operations - how to monitor unseccessful join?
2. Testability
   1. Limited unit testing
   2. MockedStreams
   3. Embedded Kafka
3. Deployment
   1. Eventual Readiness - don't start until restored
   2. Reuse the same tools as for every microservice \(for instance: docker + k8s\)
   3. Continious Delivery \(?\)

## The good, the bad and the ugly

**The good**

The beuty of Kafka Streams is simplicity. The library itself is very primitive and noncomplex. However it still provide backpressure, fault tolerance, scalability and resiliency. All of those just by reusing Kafka protocol. It takes all building blockes provided by Kafka adds local store and high level dsl and returns a so powerful tool.

1. Simplicity
2. Control
3. Easy to debug \(no cluster - no pain\)
4. Community - open sourced, stackoverflow, jira, KIPs, Confluent
5. Performance
6. Backpressure 
7. Small code base
8. Scala Api contributed by Lightbend
9. Out of the box metrics
10. At least once delivery guarantee
11. Exactly once delivery guarantee \(\*\)
12. Event Time, Clock Time, Ingestion Time

**The Bad**

1. Lack of obserbability around join operations
2. Rekeying
3. Frameworkish library 
4. kafka-to-kafka only 
   1. Possible solution - [Kafka-Connect](https://www.confluent.io/product/connectors/)
5. Implemented in Java 6 mindset
   1. Ubiquitous `null`
   2. Painful to contribute if you know Scala or Java 8

**The Ugly**

1. Async calls - blocking calls
   1. naive implementation
   2. Akka Streams as a perfect solution
2. Maintainability 
   1. [changing topology is a breaking change](https://stackoverflow.com/a/48119828)
   2. [Restore topic names contain operation number](https://issues.apache.org/jira/browse/KAFKA-6273)

## Q&A

**Q: Why bother about Kafka Streams when Spark exists?** A: "Library vs Framework" + "Unix design principle of many small programs working together"



## Materials

1. [Kafka Consumer Javadoc](https://kafka.apache.org/10/javadoc/index.html?org/apache/kafka/clients/consumer/KafkaConsumer.html)
2. [Kafka Producer Javadoc](https://kafka.apache.org/10/javadoc/index.html?org/apache/kafka/clients/producer/KafkaProducer.html)
3. [Book - Kafka Streams in Action](https://www.manning.com/books/kafka-streams-in-action)
4. [Event sourcing using kafka](https://blog.softwaremill.com/event-sourcing-using-kafka-53dfd72ad45d)
5. [https://www.confluent.io/blog/distributed-real-time-joins-and-aggregations-on-user-activity-events-using-kafka-streams/](https://www.confluent.io/blog/distributed-real-time-joins-and-aggregations-on-user-activity-events-using-kafka-streams/)

