# Kafka Streams

## The beginning of Kafka Streams

### Cumbersome Consumer and tricky producer

{% embed data="{\"url\":\"https://gist.github.com/wojda/7071eada07753b9bc6b8a8a2e45b000b\#file-kafka-streams-from-scratch-vanilla-consumer-and-producer-scala\",\"type\":\"rich\",\"title\":\"Kafka Streams from scratch - vanilla consumer and producer.scala\",\"description\":\"Kafka Streams from scratch - vanilla consumer and producer.scala · GitHub\",\"icon\":{\"type\":\"icon\",\"url\":\"https://gist.github.com/fluidicon.png\",\"aspectRatio\":0},\"thumbnail\":{\"type\":\"thumbnail\",\"url\":\"https://avatars3.githubusercontent.com/u/3869128?s=400&v=4\",\"width\":400,\"height\":400,\"aspectRatio\":1},\"embed\":{\"type\":\"reader\",\"html\":\"<script type=\\\"text/javascript\\\" src=\\\"https://gist.github.com/7071eada07753b9bc6b8a8a2e45b000b.js\\\"></script>\",\"aspectRatio\":0},\"caption\":\"Consume-transform-produce implemented with vanilla Kafka clients\"}" %}

### KIP-28 Add a processor client

KIP -  [Kafka Improvment Proposal](https://cwiki.apache.org/confluence/display/KAFKA/Kafka+Improvement+Proposals). Every new feature, every public API change must have KIP so Kafka community can discuss pros and cons of it. [KIP-28 Add a processor client](https://cwiki.apache.org/confluence/display/KAFKA/KIP-28+-+Add+a+processor+client) was a KIP that started Kafka Streams.

## Stateless Operations

### Processor API

```java
public interface Processor<K, V> {
    void init(ProcessorContext context);
    void process(K key, V value);
    void close();
    @Deprecated
    void punctuate(long timestamp);
}
```

Process method is called on every received message.

Q: Why process method returns `void`? //TODO

Q: What's the purpose of ProcessorContext? //TODO



Example implementation, processor that accepts values in String type and upper case them.

```scala
class UpperCaseProcessor extends Processor[String, String] {
  var context: ProcessorContext = _ //screeeeeeeeam
  
  override def init(context: ProcessorContext): Unit = this.context = context

  override def process(key: String, value: String): Unit = {
    val newValue = value.toUpperCase
    context.forward(key, newValue)
  }

  override def punctuate(timestamp: Long): Unit = ()
  override def close(): Unit = ()
}
```

Wiring of the processor, topology and KafkaStreams.

```scala
    val processorSupplier = () => new UpperCaseProcessor()

    val topology = new Topology()
      .addSource("Source", deserializer, deserializer, "input")
      .addProcessor("Processor", processorSupplier, "Source")
      .addSink("Sink", "output", serializer, serializer, "Processor")

    new KafkaStreams(topology, config).start()
```

### High Level DSL

1. Provides higher-level operations on the data
2. New DSL - consume -&gt; transform \(map\) -&gt; produce
3. Single thread - simplicity, no synchronisation
4. Partitioning - Scalability by sharding data \(Kafka koncept\)
5. Resiliency, Fault Tolerance - Commiting offset, Consumer rebalancing \(Kafka concepts\)

### Scalability, Resiliency, Fault tolerance

Scalability - partitioning \(Kafka Concept\). 

Resiliency - consumer rebalancing \(Kafka Concept\). 

Fault tolerance - commit offset at the end, consumer rebalancing.

Issue: How to implement backpressure? Solution: Consumer.pause\(partition, topic\) method && internal buffer.

## Stateful Operations

1. Join unboanded data sets
2. Windowing
3. State - the root of all evil
   1. external store
   2. local state
      1. faster \(lower latency\)
      2. better isolation
      3. flexible
   3. memory vs file system
   4. RocksDB 

### Scalability, Resiliency, Fault tolerance

Issue - Local state provides fast access, however it is ephemeral. If I use in-memory option and my application/node is restarted, all state is gone. Solution - Kafka Streams solves this issue by adding new feature called `logging`. Every local store has a backup Kafka topic, and every event is sent to that topic before the state give back control to application. That topic is used to restore a local store in case of a state lose.

Issue - If I consume from input topic but my local state was not restored yet, I may lose data. Solution - Kafka Streams does not start consuming from input topic until local store is restored. 

Issue - How Kafka Streams knows that local state was restored? Solution - it checks the last partition's offset on startup. When the message with the same offset is consumed, restoring is finished.

Issue - Does every Kafka Stream node have data from all stores? No, Kafka Stream node has data only from partitions assigned to it. Backup topics have the same number of partitions as a input topic, so Kafka Streams assign the same partitions of input topics and backup topic to the same node. 

Issue - Every time one node goes down a rebalancing is triggered. Partitions are assigned to other running node and local state recovery starts. If my local store contains gigabytes of data, cluster healing can take significant amount of time. Solution - Kafka Streams provides a feature called `standby replicas`. If it is enabled, each node store data from assigned partitions and  from additional partition. Data from additional partition are not used for processing. If a node goes down, Kafka Streams will assigned orphaned partitions to healthy nodes with standby replicas. Thanks that there is no delay.

Issue - for join the same partitions must be assign to the same thread. solution - custom Consumer Coordinator. **Explain Consumer rebalancing protocol.**

**Issue - if we join topic with different number of partitions we get different keys in different partitions.** Solution - there is no solution for it, joined topics must have the same number of partitions otherwise Kafka Streams throws an exception on startup. Quote from Kafka Streams Developer Guide:

> Input data must be co-partitioned when joining. This ensures that input records with the same key, from both sides of the join, are delivered to the same stream task during processing. **It is the responsibility of the user to ensure data co-partitioning when joining**.

Issue - if a new consumer joined, partitions will be rebalanced and local states will be recreated from scratch. Is there a way of minimizing the need of recreating local state? solution - custom Consumer Coordinator instead of default round-robin.

Issue - if we change the key in topology before join operation, there is no guarantee we will get all data. Solution - Rekeying. **Explain dark side of rekeying.**

## Lookup table

1. Source of enrichments - kafka topic
2. KTable
   1. Deploying new instance - don't start until restored
3. Global KTable
4. Source of enrichments - database, http service

## Async calls

1. Async calls == Blocking calls
2. Async calls =&gt; different order =&gt; different offset order =&gt; data loss
3. Failure tolerance, retries?
4. Akka Streams as a perfect solution

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
12. Event Time, Clock Time, Ingestion Time via `TimestampExtractor`
13. Supports our-of-order and delayed messages

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
   1. \*\*\*\*[**KIP-311: Async processing with dynamic scheduling in Kafka Streams**](https://cwiki.apache.org/confluence/display/KAFKA/KIP-311%3A+Async+processing+with+dynamic+scheduling+in+Kafka+Streams)\*\*\*\*
2. Maintainability 
   1. [changing topology is a breaking change](https://stackoverflow.com/a/48119828)
   2. [Restore topic names contain operation number](https://issues.apache.org/jira/browse/KAFKA-6273)



