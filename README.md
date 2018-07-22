# Kafka Streams From Scratch

#### Intro

1. Cumbersome consumer, tricky producer. Vanilla clients
2. "[Add processor client](https://cwiki.apache.org/confluence/display/KAFKA/KIP-28+-+Add+a+processor+client)" - first Kafka Streams KIP \(Kafka Improvment Process\)

#### New DSL

1. Provides higher-level operations on the data
2. New DSL - consume -&gt; transform \(map\) -&gt; produce
3. Single thread - simplicity
4. Scalability - more threads, Kafka protocol, single consumer group
5. Resiliency, Fault Tolerance - Kafka protocol
6. At least once delivery guarantee

#### Stateful operatation

1. Join unbounded data sets
2. The root of all evil - state
   1. external database
   2. local state
   3. memory vs file system
   4. RocksDB
3. Local state FTW
   1. Logging - Kafka topic as a backup
   2. `Standby replicas` - reduce time of fail-over 
4. Windowing
5. At least once delivery guarantee
   1. don't start until restored
6. Resiliency, Fault Tolerance
7. Scalability
   1. Partition everything!

#### Dark side of Kafka Scalability

1. Rekey before join
2. More operations - more^2 topics

#### Data Enrichment

1. Source of enrichments - kafka topic
2. KTable
   1. Deploying new instance - don't start until restored
3. Global KTable
4. Source of enrichments - database, http service
5. Async calls in Kafka Streams
   1. Async calls == Blocking calls
   2. At least once delivery guarantee
   3. Failure tolerance

#### Other Features

1. Stream or Table
2. Interactive Queries + External Api for local state
3. Processor API
4. Exactly-Once Semantic \(\*\)
5. Event Time Processing

#### Production Readiness

1. Observability
2. Testability
   1. Limited unit testing
   2. MockedStreams
   3. Embedded Kafka
3. Deployment
   1. Eventual Readiness - don't start until restored
   2. Continious Delivery \(?\)

#### The good, the bad and the ugly

**The good**

1. Simplicity
2. Control
3. Community - open sourced, stackoverflow, jira, KIPs
4. Performance
5. Small code base
6. Scala Api contributed by Lightbend
7. Out of the box metrics
8. At least once delivery guarantee
9. Exactly once delivery guarantee \(\*\)

**The Bad**

1. Lack of obserbability around join operations
2. Cost of `rekeying`
3. frameworkish library 
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
   2. [Restore topic names contains operation number](https://issues.apache.org/jira/browse/KAFKA-6273)

### Q&A

Q: Why bother about Kafka Streams when Spark exists? A: "Library vs Framework" + "Unix design principle of many small programs working together"

### Materials

1. [Kafka Consumer Javadoc](https://kafka.apache.org/10/javadoc/index.html?org/apache/kafka/clients/consumer/KafkaConsumer.html)
2. [Kafka Producer Javadoc](https://kafka.apache.org/10/javadoc/index.html?org/apache/kafka/clients/producer/KafkaProducer.html)
3. [Book - Kafka Streams in Action](https://www.manning.com/books/kafka-streams-in-action)
4. [Event sourcing using kafka](https://blog.softwaremill.com/event-sourcing-using-kafka-53dfd72ad45d)

