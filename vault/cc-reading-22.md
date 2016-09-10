title: 云计算 阅读材料 22 Message Queues and Stream Processing
categories:
- Technique
date: 2016-04-22 17:24:16
tags:
- CMU
- 云计算
- 讲义
---

In this module, we will look at message queues and stream processing. With the rise of internet services and the availability of continuous streams of real-time data, the challenge is to process these data streams in near-real time.

<!-- more -->

---

Streams should be viewed as an infinite sequence of small messages that arrive continuously with no breaks. The data is not at rest, and the stream processing systems that are responsible for handling these streams should be able to continuously consume and process the data.
We will begin by looking at the main ideas behind message queue systems, the abstractions that it provides which are beneficial for distributed stream processing. Apache’s Kafka is an example of a distributed message queue system that has been gaining a lot of popularity of late.

Next, we look at stream processing systems and understand the motivations of these systems in order to perform real-time processing of data streams. The primary issue in stream processing is that of managing state - certain stream processing workloads are inherently stateless as they can be performed on individual stream messages and can thus be scaled out quite easily as there is no coordination of data or state among the processors working in parallel.

Stateful stream processing workloads require the management of state that is updated by the messages. Stateful stream processes are more challenging to manage and scale out; This is especially true when the frequency of messages arriving in a stream increase or the number of simultaneous streams that need to be processed increase. Methods such as stream windowing allows for processing of multiple stream messages in batches.

Finally, we will put it all together by looking at a few big-data processing archetypes that have been discussed in the industry in the last few years. Proponents of theLambda architecture propose a multi-layered approach with a batch layer to update the state of the data using high throughput, high latency, high accuracy batch processing system, and the very latest data is processed by a low-latency, not-so-high accuracy stream processing system. An alternative is the Kappa architecture, which does away with batch processing entirely and relies on stream processing systems to produce the required results.

## Message Queues

**What is a Message Queue?**

A Message Queue is a technique used for inter-process communication (also known as IPC), or between various components of an application, or across applications. Message queues provide a protocol or interface to enable message passing.

Message Queues for IPCs within a single machine have been made available through the UNIX (`<sys.msg.h>`) and POSIX (`mqueue.h`) system libraries, where they can be used in both synchronous or asynchronous modes. A synchronous message requires the sender and/or receive processes to block while the message is being sent and has been acknowledged by the receiver. Asynchronous messages do not block the sending or receiving processes, and can be buffered for delayed delivery.

For applications that span multiple machines, a number of proprietary and open-source message queue systems have been developed such as IBM’s Websphere MQ, the Java Message Service (JMS), RabbitMQ. All of these systems provide the ability to send, manage and receive messages.

Employing message queues can add complexity to the application design and increase the number of services that need to be managed when compared to traditional request/response messaging. However, at a certain scale, and for applications such as stream processing, distributed message queue systems are needed to manage the added complexity and scale.

![Figure 5.50: An interface connects to senders (or producers), which create messages and to receivers (or consumers) which can receive the messages.](/images/14613603866906.jpg)

**Characteristics of Message Queues**

Message queues and their novelty can be understood and appreciated when contrasted with traditional RPC and request-response mechanisms. The primary features of a message queue are as follows:

**Storage**: Unlike traditional request/response systems that rely on basic TCP/UDP protocols over sockets, message queues typically store messages in some type of buffer until it has either been read by a destination process, or is explicitly removed from the message queue.

**Asynchronicity**: In contrast to request/response systems, the buffering of messages allows message queues to expose a degree of asynchronicity in applications, allowing source processes to send messages and let them accumulate in a queue while destination processes pick them up to process. This allows for applications to function under certain failure scenarios, such as intermittent connectivity or failure of either the source or destination processes.

**Routing**: Message queues may also provide routing functionality, where multiple processes can read or write messages in the same queue, allowing for broadcast or unicast communication paradigms.

**Advantages of Message Queues**

The primary advantage of a message queue is that it provides loose-coupling between various entities in a distributed application. This allows for asynchronous non-blocking communication that provides a higher level of tolerance against failures of processes.

![Figure 5.51: Message queues provide an interface that allows for programs to communicate with each other.](/images/14613604273104.jpg)

**There are no direct connections between programs**. Message queuing allows for indirect program-to-program communication. Senders do not necessarily need to know the receivers of messages and vice-versa. This allows for the logic of routing messages (and potentially work) to be decided by the message queueing system.

**Communication between programs can be independent of time**. Message queues typically buffer messages between programs so they do not have to block or interrupt their execution to send or receive messages. They can do other work, and process the reply either when a message arrives or at a later time. When writing a messaging application, you do not need to know (or be concerned) when a program sends a message, or when the target is able to receive the message. The message is not lost; it is retained by the queue manager until the target is ready to process it. The message stays on the queue until it is removed by a program. This means that the sending and receiving application programs are decoupled; the sender can continue processing without waiting for the receiver to acknowledge receipt of the message. The target application does not even have to be running when the message is sent. It can retrieve the message after it is has been started.

**Work can be carried out by small, self-contained programs**. Message queuing allows you to use the advantages of using small, self-contained programs. Instead of a single, large program performing all the parts of a job sequentially, you can spread the job over several smaller, independent programs. The requesting program sends messages to each of the separate programs, asking them to perform their function; when each program is complete, the results are sent back as one or more messages.

**Communication can be driven by events**. Programs can be controlled according to the state of queues. For example, you can arrange for a program to start as soon as a message arrives on a queue, or you can specify that the program does not start until there are, for example, 10 messages above a certain priority on the queue, or 10 messages of any priority on the queue.

**Applications can assign a priority to a message**. A program can assign a priority to a message when it puts the message on a queue. This determines the position in the queue at which the new message is added. Programs can get messages from a queue either in the order in which the messages are in the queue, or by getting a specific message. (A program might want to get a specific message if it is looking for the reply to a request that it sent earlier.)

**Recovery support**. Many message queues offer persistence and logging, allowing for a recovery of state and messages in the queue during failures.

**Multiple queues (topics in Kafka)**. Some systems allow for multiple queues that can be defined and configured by the application developer. This allows for messages to be routed to the necessary entities based on a publisher/subscriber relationship. Apache Kafka is an example of this.

**Ease of scalability**. Message queues can horizontally scale to deal with an increase in the message load, as opposed to tightly coupled systems, where scaling and managing the communication traffic and endpoints is harder.

## Message Queues: Case Study

Previously, we looked at Message Queue systems in general. We saw that general purpose message queue systems for interprocess communication have been around for a while, and more specialized message queues for client-server based systems or web services architectures have emerged. While there are several different systems with unique designs and features, we will actually look at a system that has been built from ground-up for scalability and elasticity: Apache Kafka.

**Apache Kafka**

Kafka is responsible for handling messages originating from a set of programs (known as producers), and sending them to a set of machines that may be interested in the said messages ( known as consumers). The messages are published by the producers into a Kafka topic. Consumers can listen to specific topics by subscribing to them and messages will be delivered to the consumers by Kafka. Hence, Apache Kafka can be described as an open-source distributed publish-subscribe messaging system.

![Figure 5.52 : A kafka cluster](/images/14613605854908.jpg)

Topics: Topics represent a user-defined category to which messages are published. An example topic one might find at an advertising company could be AdClickEvents. All consumers of data can read from one or more topics. Internally, each topic is maintained as a partitioned commit log as illustrated in Figure 5.53. It is important to note that a topic can consist of multiple partitions and a Kafka cluster can handle multiple topics.

![Figure 5.53 : Message Queuing in Kafka](/images/14613605990756.jpg)

Formally, each partition is an ordered, immutable sequence of messages that is continually appended to a commit log. The messages in the partitions are each assigned a sequential id number called the offset that uniquely identifies each message within the partition, and cannot be used to order messages in a topic across partitions.

The partitions in the log serve several purposes. First, they allow the log to scale beyond a size that will fit on a single server. Each individual partition must fit on the servers that host it, but a topic may have many partitions so it can handle an arbitrary amount of data. Second they act as the unit of parallelism, allowing for individual partitions of the log to be distributed among multiple machines.

Producers not only have control over the topic of a message, they can explicitly control the partition that a message is sent to, if needed, using a semantic partitioning function (much like the partitioning functions used in MapReduce). By default, messages in a particular topic are round-robin distributed over the partitions for that topic.
The Kafka cluster retains all published messages - whether they have been consumed or not - for a configurable period of time (the default is 7 days). Messages that are past this retention period are automatically purged by Kafka to create free space for new messages.

Kafka also keeps track of the progress of each consumer in reading the messages of the log file for a particular topic, known as the offset for a consumer. Most consumers advance this offset value linearly as they consume messages, but consumers are in control of this offset variable and can, move forwards or backwards to read older or newer messages, if needed.

This combination of features means that Kafka consumers are very cheap—they can come and go without much impact on the cluster or on other consumers.

**Guarantees provided by Apache Kafka**

Kafka provides a set of high-level guarantees that application developers can rely on:

1. Messages sent by a producer to a particular topic partition will be appended in the order they are sent. That is, if a message M1 is sent by the same producer as a message M2, and M1 is sent first, then M1 will have a lower offset than M2 and appear earlier in the log.
2. A consumer instance sees messages in the order they are stored in the log.
3. For a topic with replication factor N, we will tolerate up to N-1 server failures without losing any messages committed to the log.

Notice that the delivery guarantees are not very strict. This includes the fact that consumers may obtain the same message twice, in rare instances.

**Architecture of Apache Kafka**

![Figure 5.54 : Kafka Architecture](/images/14613606629918.jpg)

The servers that transfer messages from publishers (producers) to subscribers (consumers) are known as the Kafka brokers. The Kafka brokers are responsible for message persistence and replication. The partitions of each topic are distributed over the brokers, and each broker stores one or more partitions.

The brokers are organized in a decentralized manner, as there is no fixed master broker. In order for the brokers to achieve consensus about the state of the system, Apache Zookeeper is employed. Zookeeper provides a highly-available quorum consensus service in the form of a file-system like API. Zookeeper is used in Kafka for the following tasks:

1. Detecting the addition and removal of brokers and consumers in the system,
2. Triggering a rebalance of partitions when the number brokers or consumers change, and
3. Maintaining the consumption relationship and keeping track of the consumed offsets for each partition.

As mentioned earlier, the smallest unit of parallelism in Kafka is a partition in a topic. This means that all messages of a partition are consumed by one consumer at any given time. This saves Kafka from doing expensive coordination among multiple brokers if it were to guarantee order across partitions.

Partitions are replicated among multiple brokers for fault tolerance. One of the brokers is designated as the leader for a particular partition, and all reads and writes for that particular partition go to the master replica by default. A message is considered committed only when all the replicas have committed the message to their log. Only committed messages are forwarded to the consumers. Producers can choose to block until a message is committed by Kafka or choose to continuously stream messages in a non-blocking fashion. There are multiple techniques employed by Kafka to speed up the log replication process, and details can be read here: http://kafka.apache.org/documentation.html

Since Kafka brokers are expected to process large amounts of messages, there are two “liveness” properties that Kafka monitors for every node in the cluster:

1. Each node maintains a session with Zookeeper via a heartbeat mechanism.
2. Each slave must replicate the master’s updates and should not fall “too far” behind. The replica lag is a configurable property in a Kafka cluster.

**Producer Interaction**

Producers can send messages to a Kafka cluster using the Kafka API. Producers are aware of the topics and partitions configured. Kafka and a producer typically directs messages to the appropriate broker that is responsible for handling the particular partition of a message. The API also allows for metadata requests to be made. This allows producers to query and locate the appropriate broker for a topic and partition. As mentioned earlier, the partitioning of a topic is configurable, and random load-balancing or content-aware semantic partitioning of a topic can be employed.

In addition, producers interacting with a Kafka broker have the option of asynchronous communication of messages as well as request batching, which accumulates messages and sends them in batches. These are also configurable, both in terms of the number of messages to batch, or a fixed latency bound, allowing the application to trade off latency for throughput.

**Consumer Interaction**

Kafka consumers issue fetch requests from the brokers for individual partitions that need to be consumed. A consumer can specify an offset in the Kafka log with each request and receives a chunk of messages that begin from that position. Consumers can rewind to any offset and request messages, provided those messages are within the retention window for the Kafka cluster.

Kafka consumers issue fetch requests from the brokers for individual partitions that need to be consumed. A consumer can specify an offset in the Kafka log with each request and receives a chunk of messages that begin from that position. Consumers can rewind to any offset and request messages, provided those messages are within the retention window for the Kafka cluster.

**Kafka Use Cases**

Messaging Queue: Unsurprisingly, Kafka can be used as a replacement for traditional messaging queues such as ActiveMQ or RabbitMQ. Kafka is particularly significant as it is designed from the ground-up for high availability and scalable message delivery, with configurable latency and throughput requirements.

Website Activity Tracking: Kafka was originally built by LinkedIn to build a user activity pipeline and make real-time decisions for content and Ad placement for LinkedIn users. In this scenario, topics can be constructed by user interaction type (e.g. a topic for page views and scrolling information, another one for search terms and another topic for user clicks). Various back-end services such as real-time processing and monitoring of user activity can subscribe to the relevant topics and process the streams as they come in.

Log Aggregation: Kafka can be used to aggregate the logs from multiple services and make them available in a central location for processing. In comparison to log-centric systems like Scribe or Flume, Kafka offers equally good performance, stronger durability guarantees due to replication, and much lower end-to-end latency.

## Stream Processing Systems

The frameworks that we have looked at so far (MapReduce, Spark, GraphLab), were primarily designed to perform batch computations. Their inputs are typically large distributed datasets, which are processed for several hours to yield a large, useful output. The use of these frameworks was originally restricted to data scientists and programmers, who used them for specific, large queries where this high latency was tolerable. However, as the use of Big Data gained prevalence within enterprises, there was a move towards ad hoc querying of data, with expected latencies of minutes, not hours. Tools like Pig, Hive, Shark & SparkSQL allowed many businesses to ask complex questions of their data, without relying on a large pool of highly trained programmers. The cloud drove this adoption even further, providing an elastic supply of compute resources for the duration of an ad hoc query.

Soon, the expectation of latencies got even lower. Big Data began to be received in real-time, and was often only valuable for a short duration. For example, search engines required the best combination of advertisements to be served within milliseconds for each query; social media websites detected trends and trending topics/hashtags and system monitoring tools detected complex patterns across several large infrastructure components. To be able to provide such low latencies, a new class of stream processing frameworks began to take shape. These had fundamentally different requirements and constraints from the batch/interactive processing systems of the past.

This led to the advent of stream processing systems.

**Stream Processing**

The stream processing paradigm applies a series of operations on each element of data emitted by an infinitely long input data source. The series of operations are generally pipelined, which adds dependencies between operations. Within the processing application, state information is often read from and written to a small, fast data source. The output of a pipeline of stream operations is also a data stream. This can be used to trigger other applications, or be buffered and stored to stable storage. The basic conceptual architecture of such a system is shown below.

![Figure 5.54: A stream processing system must process data in-stream, with a separate pipeline for storage, if needed, which does not lie on the “critical path”](/images/14613608039934.jpg)

**Eight Rules for Stream Processing**

Stonebraker et. al. described 8 basic rules for stream processing systems.

1) Keep the data moving: A real-time stream-processing framework must be able to process messages “in-stream”, without having to store them on disk, which adds unacceptable latency on the critical path. Additionally, these systems should be active (event-driven) and not passive (whereby applications need to poll the results to detect conditions of interest).

![Figure 5.55: A stream processing system must process data in-stream, with a separate pipeline for storage, if needed, which does not lie on the “critical path”](/images/14613608370355.jpg)

2) Streams should support querying using SQL: SQL has emerged as a widely used and familiar standard for querying data. However, traditional SQL operates on a fixed amount of data, whereby reaching the end of the table tells the query it is completed. In streaming scenarios, the data increases continuously. Stonebraker et. al. perceived the need for a StreamSQL language, with variable-length time-based sliding windows defining the scope of a query. Windows could be defined using time, number of messages, or any arbitrary parameters. Additional operators may be needed to merge messages from multiple streams.

![Figure 5.56: StreamSQL should process subsets of the data, and allow relations to be expressed across windows](/images/14613608561029.jpg)

3) Handle stream imperfections: In real-time systems, data can be lost, arrive late, and/or out-of-order. A stream processing system cannot wait indefinitely for data, but also may not have the flexibility to ignore/miss any data. Such systems must be resilient against stream imperfections, with mechanisms like configurable timeouts, and “slack times”, during which a late-arrival can be accepted.

4) Generate predictable outcomes: The outcome of any stream processing system must be deterministic and repeatable by replaying the stream. This is particularly hard when the system is operating on multiple concurrent streams, or when messages arrive out-of-order. Messages must be produced in ascending time-order, irrespective of their arrival time. This property also enables fault-tolerance, by making it reasonable to replay streams upon which processing failed.

5) Integrate stored state: Stream-processing applications must often combine the present with the past. For example, when recommending an ad to a user, Google must combine the current information about the search term and the current state of the ad market, with past information about the user’s click habits. Integrating stored state and streaming data also allows seamless switching, whereby an algorithm can be tested on historical data, and then switched-over to the live stream when it works satisfactorily. The data should be stored in the same system address space as the application, perhaps using an embedded database, allowing the use of a uniform language that deals with stored and streaming data.

6) Guarantee high availability: Stream processing systems work in real-time, and often cannot tolerate restart-recoveries. Such systems must allow a hot-switchover to a backup/shadow, which must be regularly synchronized with the primary. The integrity of the data must be guaranteed, in accordance with rule 4.

7) Support partitioning and auto-scaling: Distributed processing is the standard model of operation for all such large systems. A good stream processing architecture should be non-blocking and exploit modern multi-threaded architectures. Further, it should be able to handle the scaling out/in of the system on its own, by adding or removing machines, either based on increased/decreased data volumes or processing delays/complexities. Additionally, it must automatically and transparently perform load balancing over the available machines. The end-user should not have to deal with any of this complexity.

8) Make sure it can keep up: All system components should be designed for high-performance, with a minimal number of operations happening out-of-core. The system must be tested and benchmarked based on the target workload, and the throughput and latency goals must be validated.

**Evolution of Stream Processing Engines**

Aurora (2002) was one of the earliest stream processing systems, also developed by Stonebraker et. al. at MIT and Brown University. Aurora treated the stream processing problem as a directed acyclic graph (DAG).

The stream input is a sequence of unbounded tuples (a1, a2, … an) over time that flow from upstream (start) to downstream (output). An entire application can be built by adding different combination of processing boxes and drawing links between them. Aurora was a single-node system, which lacked many of the scalability requirements of a stream processing engine. This was rectified in Aurora* (2003), which simply combined many Aurora nodes over a network. Thus, scalability was achieved by partitioning the different stages of the stream processing job over different physical nodes. Finally, the Medusa project (2003) added federation support to Aurora, allowing collaboration and sharing by multiple users.

Borealis (2005) was the next extension of the Aurora project, which added support for high availability using active replication. The replicas were carefully synchronized to provide data consistency.

Apache Storm (2011) was a stream processing engine developed by Twitter. Here, the processing nodes (Bolts) could subscribe to streams from different sources (Spouts), thereby enabling a simple subscriber model of computation. Storm provides guaranteed message processing, regardless of node failures, and enables exactly-once semantics ensures data is neither undercounted nor overcounted. Apache S4 (2011) was a similar subscription system developed at Yahoo!. It is symmetrical, in the sense that all nodes are equal and there is no centralized control, in the hopes of making it scalable. S4 did not support dynamically adding or removing nodes to/from a running cluster. Apache Samza (2013) is another multi-subscriber system in this mould, that we will explore in more detail.

Storm, Samza and S4 follow the traditional streaming model, known as record-at-a-time-processing. In this model, stateful operators process arriving records, using the new data to modify internal state and then emit new records. Fault tolerance and recovery is done by replication, either by making multiple copies of processing elements, or by buffering and storing backups of messages upstream, and resending them downstream, in case of failures. Also, as the layout of the DAG grows more complex, it is hard to ensure consistency across different paths.

Finally, combining these frameworks with batch systems is non-trivial, and is often done using the lambda architecture (discussed later).
Another approach to designing stream-processing systems is provided by Spark Streaming (2012), which provides “micro-batching”. Micro-batching converts stream computations into a set of extremely fast computations, with latencies from hundreds of milliseconds to a few seconds. At the cost of increased latency, this makes it easier to provide fault tolerance and exactly-once semantics on the result of each micro-batch.

Selecting the correct framework to use for a task is a factor of the expected latency, fault tolerance and message delivery guarantees, as well as the skillset of the users and desired development costs. On the next page, we will explore the internals of these frameworks in more detail, by studying Apache Samza.

## Streaming Architectures : A Case Study

Now that we have seen how streaming architectures evolved we can look at one specific framework- Apache Samza.

**Apache Samza**

The Samza project was developed at LinkedIn as a distributed stream processing framework. It converts an input stream of messages to a modified output stream, based on stateful or stateless processing. Samza was developed alongside Kafka (discussed earlier), which was a low-latency distributed messaging system. Samza enabled the real-time processing of these Kafka messages.
Samza is divided into 3 layers:

1. A streaming layer that provides partitioned, replicated and durable streams
2. An execution layer that schedules and coordinates tasks on a cluster
3. A processing layer that transforms the input stream and generates a new output stream, changes databases, triggers events, and in general, reacts to the input messages.

![Figure 5.57: The three layers of a Samza application](/images/14613613162450.jpg)

The streaming and execution layers are pluggable. A default implementation uses Kafka as the streaming message broker. The input and output streams are immutable message sequences, which can be partitioned across nodes. Within a partition, messages are globally ordered and uniquely identifiable by the offset within the stream. The default execution layer uses YARN, though Mesos is another common resource manager that could be used. Using YARN makes it easier for a Samza application to ensure fault tolerance, simplify deployment, and use the built-in logging and resource isolation features. Using YARN with HDFS also allows Samza to benefit from data locality.

Samza also uses cgroups process single-core containers that run JVMs to execute one or multiple tasks within a single job. Cgroups is a Linux kernel feature that enables a collection of processes to have a collective bound on their CPU, memory and filestystem access. In Samza, each container is logically executed as single-thread when processing a message, in the sense that only a single task within a container is executing at any point in time. The processing is done by custom code written using the Samza API.

To gain more parallelism, Samza simply spawns more containers. For this reason, developers are discouraged from using multithreading within their job’s code. Samza uses multiple threads internally for communications and processing, however the single thread runs as an event loop that handles message I/O, checkpointing, windowing and flushing metrics.

![Figure 5.58: The three layers of a Samza application](/images/14613613386567.jpg)

Samza clients initiate Samza jobs into YARN. Samza has its own Application Master which negotiates for resources with the YARN Resource Manager (RM). The YARN RM talks to various Node Managers to allocate resources to a Samza application. YARN spawns SamzaContainers (Task Runners) that run custom code that implements the Samza StreamTask API. These are often co-located with the containers for Kafka brokers in order to maximize data locality.

![Figure 5.59: A Samza job is split into tasks, which can be grouped within a container. As there is only one thread per container, only one task is active at any time](/images/14613613543819.jpg)

Samza relies on horizontal scaling for performance improvements. This is done by increasing the number of tasks within a job. Each task operates on a single partition from the job’s input streams. Thus, to be able to run more parallel tasks, a stream must be broken into a larger number of partitions. This has been described in the earlier section on Kafka. For each input topic, there is at least one StreamTask instance initiated for each partition. Each stream task independently processes a single partition.

![Figure 5.60: Samza applications run on YARN in isolated containers](/images/14613613699810.jpg)

Of course, the streaming example shown above simply transforms an incoming stream into an output. There are many stream processing applications where the computation performed on any input message is independent of all other messages. Examples include data filtering based on rules or simple time-based modifications.

However, more interesting use cases for stream processing require connecting multiple streams, performing message aggregation or making decisions based on a window of data. All of these scenarios require the storage of state information. Samza implements durability using the KeyValueStore abstraction. Each StreamTask instance stores state on a separate embedded data store on the same machine. By default, Samza uses RocksDB, which provides low-latency and high-throughput and is write-optimized. Using an embedded database reduces the overhead of relying on expensive network calls to query data.

![Figure 5.61: Ensuring durability of a task’s local state using an embedded data store](/images/14613613857411.jpg)

This implementation can be thought of as sharding a remote database and co-locating each shard with a unique data partition. To ensure that failures do not lead to a loss of state, any modification to a local database is emitted using a separate changelog stream, which is a separate Kafka topic. A separate background process runs log compaction to reduce the amount of data in the changelog.

![Figure 5.62: Each local embedded database writes to a changelog output stream](/images/14613614001397.jpg)

Thus, tasks can be easily scaled out by launching a new container with its own DB and writing to another parallel changelog stream. In case of any failures, a new container can be launched and restored to a consistent state by consuming from the output changelog of the failed partition.

![Figure 5.63: Failure recovery in Samza](/images/14613614145769.jpg)

## Big Data Processing Architectures

In this unit, we have covered the basic ideas behind distributed programming and analytics engines, as well as the challenges of large-scale, big data processing. We saw a number of frameworks, including MapReduce, Spark, GraphLab, as well as Stream Processing frameworks. To cap our discussion, we now present some of the ongoing attempts to define an architectural paradigm to help build systems that can handle both real-time and historical data: The Lambda and Kappa Architectures.

For example digital assistants such as Apple’s Siri often use complex machine learning algorithms for speech recognition and for understanding user queries. Incoming user queries can be considered to be a stream which needs to be responded to in low latency. However, over time the historical data of user queries can be used to re-train the machine learning system for better speech recognition and query understanding. The latter can use some sort of batch processing system to update the machine learning models for future queries.

**Lambda Architecture**

The Lambda Architecture is a data-processing architecture designed to handle massive quantities of data by taking advantage of both batch and stream processing methods. Lambda attempts to balance latency, throughput, and fault-tolerance by using batch processing to provide comprehensive and accurate views of batch data, while simultaneously using real-time stream processing to provide views of online data.

The Lambda architecture describes a system consisting of three layers: the batch layer, the speed (or real-time) processing layer, and a serving layer for responding to queries.

![Figure 5.67: A stream processing system must process data in-stream, with a separate pipeline for storage, if needed, which does not lie on the “critical path”](/images/14613618236246.jpg)

The flow of data in the lambda architecture can be represented in the Figure (5.67). The steps are as follows:

1. All data entering the system is dispatched to both the batch layer and the speed layer for processing.
2. The batch layer manages the master dataset, and pre-computes the batch views.
3. The serving layer indexes the batch views so that they can be queried in a low-latency, ad-hoc manner.
4. The speed layer builds real-time views of the incoming data in a manner that is much faster than the batch+serving layers, but may not be as accurate.
5. Any incoming query can be answered by merging results from batch views and real-time views.

**Batch Layer**

The batch layer has a number of important functions within the lambda architecture:

1. First, the batch layer typically manages the master dataset. The master dataset is the source of truth in the Lambda Architecture, and could be used to reconstruct any data that is served by the system in case of failures in any of the layers of the system. Data in the batch layer is typically organized as an immutable, append-only log, which contains new data as it arrives into the system.
2. Second, the batch layer builds the batch views of the data. The batch layer aims at perfect or near-perfect accuracy by being able to process all available data when generating the batch views. This means it can fix any errors by recomputing against the complete data set, then updating the existing batch views. Output from the batch layer is stored in the serving layer.

Apache Hadoop is the de facto standard batch-processing system used in most high-throughput architectures, and is the typical choice for implementing the batch layer. The processing can be done using MapReduce or any of the higher-level batch processing systems that are built on top of Hadoop.

**Serving Layer**

Output from the batch layers (the batch views) are stored in the serving layer and made available for querying by applications. It is tightly tied to the batch layer. The serving layer is typically distributed among many machines for scalability. The serving layer typically comprises of some type of database and is typically NoSQL in nature. The serving layer requirements are as follows:

+ Batch writable - The batch views for a serving layer are produced from scratch. When a new version of a view becomes available, it must be possible to completely swap out the older version with the updated view. As a result, serving layer systems do not need to be optimized for fast random writes, unlike traditional database systems.
+ Scalable - A serving layer database must be capable of handling views of arbitrary size. As with the distributed filesystems and batch computation framework previously discussed, this requires it to be distributed across multiple machines.
+ Random reads - A serving layer database must support random reads, with indexes providing direct access to small portions of the view. This requirement is necessary to have low latency on queries.
+ Fault-tolerant - Because a serving layer database is distributed, it must be tolerant of machine failures.

Common examples of datastores used in the serving layer include Apache Hive, HBase and Impala.

**Speed Layer**

The speed layer processes data streams in real time with the lowest possible latency to generate real-time views of the data. Essentially, the speed layer is responsible for filling the "gap" caused by the batch layer's lag in providing views based on the most recent data.

This layer's views may not be as accurate or complete as the ones eventually produced by the batch layer, but they are available almost immediately after data is received, and can be replaced when the batch layer's views for the same data become available. Using incremental or stream processing approaches, that we discussed previously in this unit, the processing can be done in a more efficient manner if the computation can be expressed as a function of the previous real-time view and the recent data, to produce the updated real time views.

![Figure 5.68: A stream processing system must process data in-stream, with a separate pipeline for storage, if needed, which does not lie on the “critical path”](/images/14613619267098.jpg)

Stream-processing technologies typically used in this layer include Apache Samza, Apache Storm, SQLstream and Apache Spark. Output is typically stored on fast NoSQL-style databases for low latency querying.

**Kappa Architecture**

![Figure 5.69: A stream processing system must process data in-stream, with a separate pipeline for storage, if needed, which does not lie on the “critical path”](/images/14613619520000.jpg)

The Kappa architecture, popularized by LinkedIn is shown in Figure 5.69. One of the important motivations of the Kappa architecture was to avoid maintaining two separate code bases for the batch and speed layers. The key idea is to handle both real-time data processing and continuous data reprocessing using a single stream processing engine.

Data reprocessing is an important requirement for making visible the effects of code changes on the results. As a consequence, the Kappa architecture is composed of only two layers: stream processing and serving.

The stream processing layer runs the stream processing jobs. Normally, a single stream processing job is run to enable real-time data processing. Data reprocessing is only done when some code of the stream processing job needs to be modified. This is achieved by running another modified stream processing job and replaying all previous data. Finally, similar to the Lambda architecture, the serving layer is used to query the results.

Lambda v.s. Kappa architecture is an ongoing debate in the big-data processing community. The choice of architectures depends on some characteristics of the application that is to be implemented. A simple approach suggested by Ericsson is the following:

1. If the algorithms used for the historical and real-time data are identical, then it is generally better to go with the Kappa approach. Some form of batch computation may be necessary to bootstrap the views, depending on the amount of historical data present and the rate of arrival of new data.
2. In certain types of applications, such as machine-learning applications, the output of the batch and real-time systems differ in accuracy because of the amount of data that it is computed upon. This makes it very difficult to merge the batch and real-time processing results into a consistent view, and a Lambda-based architecture may be better for the application.
3. In some cases, the batch algorithm can be optimized thanks to the fact that it has access to the complete historical dataset, and then outperform (in terms of processing throughput) the real-time algorithm. Here, choosing between Lambda and Kappa becomes a choice between favoring batch execution performance over code base simplicity.

## Real-time architectures in practice

Many web-scale enterprises have now begun to use message queues and stream processing frameworks within their applications. This is primarily because these companies have begun to gain significant value from the use of fresh data. Now that we have covered the basic concepts of message queues, stream processing and lambda architectures, it may be interesting to look at a real-world example of how a low-latency data-intensive architecture provides value to a large company, in this case, LinkedIn. Specifically, we will illustrate how incorporating Kafka and Samza allowed LinkedIn to feed many different real-time systems.

**Incorporating Kafka and Samza at LinkedIn**

As a young technology company, LinkedIn often found itself building and adopting a large number of disruptive technologies, each to solve a specific requirement. Their services ran on a custom backend that use LinkedIn’s own low-latency distributed key-value store (Voldemort), a distributed document-store (Espresso) and an Oracle RDBMS. Front-end web services, analytics, emails and notifications were driven by tasks of different complexity. These included batch jobs using Hadoop, queries on a large data warehouse, a separate infrastructure for searches, all supported by a layer of logging, monitoring and user/metric tracking.

![Figure 5.70: Data Integration disaster at LinkedIn](/images/14613620196530.jpg)

A few years ago, engineers at LinkedIn realized that their system had grown too complicated. Communications between services, front-ends and back-ends were all-to-all, since each component had its own latency and throughput requirements. The addition of a single service required connections to a large number of components, which in turn led to very fragile endpoints. This also led to siloes, as individual service owners were less likely to share their data.

The first change at LinkedIn was a move towards using a centralized Kafka-driven data pipeline. This allowed each service to produce data and emit it using a specific topic. Accessing data from another service now became as simple as subscribing to that particular topic. Since the pipeline is internally run on a distributed cluster, it is expected to be inherently scalable.

![Figure 5.71: A single, distributed, asynchronous data pipeline](/images/14613620351967.jpg)

Another service was now needed to manage membership of nodes within a data center. Since scaling up/down was now simply a matter of adding/removing nodes from the system, a new service called Zookeeper was used to manage membership within the cluster, distribute ownership of partitions and topics within the broker tier, and correctly handle offsets on the consumer tier.

![Figure 5.72: High-velocity data processing using Kafka topics](/images/14613620484519.jpg)

The changes in their architecture were also driven by a move towards more reactive systems. In their original architecture (around 2010), all user activity logs were scraped by a batch process every few hours and this was used to generate new graphs and dashboards on the front-end. This architecture allowed LinkedIn to have hourly or daily updates of view counts, user recommendations, and trending topics.

![Figure 5.73: Before-- Offline processing of user activity logs](/images/14613620629105.jpg)

However, latencies of a few hours were no longer acceptable; neither to the end-users who wanted to know page view statistics and trends from the last five minutes, nor to advertisers and data analytics folks who needed to be able to display the most relevant information in real-time. Advertisements are no longer driven solely by offline analysis of clickstreams in data warehouses, but in fact, are increasingly impacted by real-time models that are updated online.

![Figure 5.74: Left : Traditional Ad serving ; Right : Model ad serving](/images/14613620763812.jpg)

This is what stateful streaming using Samza enables. Traditionally, the source of truth would always arrive from one of the central data stores. However, databases cannot serve such a large number of simultaneous requests, to match the complex real-time queries of recommendation and complex analytics systems. This also placed a great demand on the bisection bandwidth of the data center. Instead, LinkedIn’s approach was to use the same stream of data to update multiple backends. Thus, as each consumer node had a local embedded database, a single update event would simply trigger all the relevant updates in real-time on this local database.

A simple example of a User Profile update event is shown below. As the profile is updated, it writes to a specific UserProfileUpdate topic, which is consumed by the Newsfeed (to show in real-time that the user has moved to a new company), the search datastore (so that it doesn’t return stale data), and also consumed to be used for offline processing (e.g. to capture trends in the number of software engineers joining a particular company per day).

![Figure 5.75: A UserProfileUpdate is consumed at LinkedIn](/images/14613620924992.jpg)

When you are running large web-scale distributed systems like Google, Facebook or Netflix, there is a paradox in design choices. Each of these companies have created a large number of specialized systems such as data stores, networking interconnects, batch/interactive/stream processing frameworks, etc. This is because the one-size-fits-all solutions of the past no longer meet specific hard requirements of today’s users. However, the downside of this approach is a significant amount of “reinventing the wheel”, building custom solutions to problems like failure-detection and recovery, cluster management etc. The move towards such systems is primarily driven by needs of agility, ease-of-management and scalability.

In short, we see here the benefits accrued by LinkedIn in moving towards a log oriented system. This made it easier to be schemaless, allowing easier data integration. By making scalable log message passing a first class citizen within their data center, LinkedIn built a highly modular and flexible system.

## Summary

Message queues are communication mechanisms used to enable indirect, asynchronous communications by partitioning and storing messages on brokers. This allows easy horizontal scaling of the messaging layer.
Kafka is a multi-subscriber message queue developed at LinkedIn. Consumers of this queue can choose to subscribe to topic(s), and is guaranteed to receive messages in the order sent.

Stream processing systems operate on an infinitely-long, often fast-moving set of input records, for e.g. the output of a message queue. To reduce latency, there are a set of simple rules that it could follow.

Stream processing jobs could be stateless (simply applying pre-defined rules to an input) or stateful (applying continuously changing rules based on past data and current status).

Samza is a stream processing framework developed at LinkedIn. By default, Samza is configured runs cgroups containers scheduled over YARN, reads from a Kafka stream, allowing programmers to use a custom API to define streaming tasks. When local state is needed, an embedded RocksDB instance is used.

Lambda and Kappa architectures are two methods of working with data pipelines with different latency requirements.

