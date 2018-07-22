# Introduction

## Purpose

This article is written to show a real world example of reusability and composability - Kafka Streams library. I'll show how the library was built step by step and a new tool that already reause Kafka Streams as a building block.

**Kafka Concepts -&gt; Kafka Clients -&gt; Processor API -&gt; High Level Dsl -&gt; KSQL**

As a lazy programmer and a huge fan of open source software I always try to work with "building blocks". Github is full of technologies, frameworks, libraries that provide a small modules and reusable components. Everyone knows Unix pipe, the best concept of building blocks and reusability [since 1973](https://en.wikipedia.org/wiki/Pipeline_%28Unix%29).

```bash
cat file | grep streams | wc -l
```

I am a Software Developer that always must know all details of used tools. Reading a documentation from cover to cover is a must. If it is open sourced, that is even better, show me the code. However sometimes reading the documentation and the code is not enough. Sometimes you must write the same feature from scratch to understand the problem and the final solution. That is ultimate understanding.

I recommend the same in this case, there is no better way of explaining Kafka Streams design than writing it from scratch. Shall we start?



