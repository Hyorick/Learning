# KAFKA

=====================================

https://www.youtube.com/watch?v=B7CwU\_tNYIE

https://www.youtube.com/watch?v=cJMMzOr5BDA Devoxx Ukraine 2020. Data processing with Kafka Streams and Spring Framework. 

https://www.youtube.com/watch?v=inrQUHLPFd4\&list=PLGRDMO4rOGcNLwoack4ZiTyewUcF6y6BU

### Kafka Streams (KStream<key, Value> in Spring) (library Generaliste : prend en charge JMS ou Kafka Template) (Spring Cloud Stream dependency)
Kafka Streams is a client library for processing and analyzing data stored in
Kafka. It builds upon important stream processing concepts such as properly
distinguishing between event time and processing time, windowing support,
and simple yet efficient management and real-time querying of application
state .

- Kafka Streams is just a library used by your application;
- Uses Kafka Producer/Consumer APIs;
- Provides API for processing the data in Kafka;
- Contains solutions of the data streaming processing problems:
aggregations, state, recovery, scaling, etc-

Sans Kafka Streams (soit Kafka Template pour acceder à Kafka ou JMS pour accéder au cluster pour JMS )

**Out of box**:
- **Fault-tolerance**
- **Scaling**
- Exactly-once processing semantics (default is at-least-once)
- **Aggregation operations**: **windowing**, **count**, ...
- Late-arrival records handling
- Ability to extend default functionality

Stream (kStream) a pair of key-value 
Ex: ("Alice", 1), ("Bob", 2) ...

Table (kTable): a snapshot of all values of all latest values for all values in a stream

represent latest value for each key in a stream. At any time you can from Stream to Table and vice versa


#### Windows (KSql)
windows are created per record key

#### Data processing types

2 types:

- **Stateless**:
    - Send a message to another system
    - Write a log

Examples of operators: **filter**, **map**, **forEach**, **print**

 - **Stateful**:
    - Count amount of messages
    - **Join** few **streams**
    - Any operation when you need to "remember" something outside of a single message scope
    - **Stores state** in a... state store

Examples of operators: **aggregate**, **count**, **reduce**

##### State stores

- **Stored** in a **local file** system using ***RocksDB***. Default **location** - `/tmp`
- **Backed up** to a «changelog topic» in Kafka to restore a **local copy after
restart**
- Application identifies a state store using the `"application.id"` **property**

### The different concepts of time in Kafka Streams

Approaches for definition of a message's time, 3 types:
- **Event time** - moment of time when an **event happened on a source**
(e.g. time when a **measurement was captured** on by a sensor)

- **Processing time** - moment of time when a **message is processed** by
an **application**

- **Ingestion time** - moment of time when a message **got into Kafka
topic**

!! With Kafka Streams, you **choose one of the mentioned approaches** by
implementing `TimestampExtractor` *interface*

#### Stream time
As a result, **time** will **only advance** when a **new record arrives** at the
**processor.** This concept is called `Stream time`.

!! Kafka cluster derives time **from the records** (messages) **not from your actual computer clock**

**Expired window, why?**
- !!`Stream time` is **tracked per partition** **not per key**! (qui fait que **certaines données collectés ou généré en** ***offline*** **ne sont pas envoyé (skipped)** à **Kafka une fois la connexion réétablie**)
- There is KIP-540 to address this problem

**Expired window**: solutions

- **Custom operator** that implements enhanced window expiration/suppress
logic (see `SuppressOperatorWithStreamTimeTrackedByKey.java` in the source code
after the talk)
- Wait for the solution from the Kafka team (see **KIP-540**)

**Expired window**: one more solution

Create **as many** `partitions` **as there are** `keys` and make sure your partitioning strategy is **one-to-one**.

 
### DevOps: Things to know when deploying to cloud

- **Spring Cloud Actuator**'s `/health` endpoint tracks Kafka Streams
threads liveness starting from version 3.0.0 only
- Application doesn't die on it's own even if Streams threads are dead
- Better to have the same persistent `/tmp` folder between application
redeploys because it helps Kafka Streams to recreate state stores
immediately as opposed to rebuilding from the **changelog topic**
- Consider deploying to the same node
- Use `StatefulSet` in **Kubernetes**

==================================================

What the developer must handle manually

1\. **Consumer concurrency** (Kafka does NOT do this for you): !!! on peut **avoir la concurrency** du coté des **consumer**, en utilisant des **thread pool**



## **!!! Concurrency != Parallelism (TODO)**

* **Concurrency (interleaving tasks)**

Think of one person cooking dinner:

You chop vegetables

Then you stir the sauce

Then you check the oven

Then you go back to chopping


**You’re not doing tasks simultaneously, but you’re managing multiple tasks in the same time period.**



**➡️ Concurrency = switching between tasks**  

**➡️ It’s about structure, not speed.**



**In programming:**

**One thread handles multiple tasks by switching between them**

**Tasks appear to run together, but not literally at the same instant**


* **Parallelism (simultaneous execution)**

Now imagine three people cooking:

One chops

One stirs

One checks the oven


**All tasks happen at the exact same time.**



**➡️ Parallelism = true simultaneous execution**  

**➡️ Requires multiple CPU cores or multiple threads running at once**



**Concurrency and parallelism can exist independently**

| Scenario | Concurrency | Parallelism |
| --- | --- | --- |
| One thread switching tasks | ✅ Yes | ❌ No |
| Multiple threads on one CPU core | ✅ Yes | ❌ No |
| Multiple threads on multiple cores | ✅ Yes | ✅ Yes |
| GPU doing 10,000 operations at once | ❌ No (not switching tasks) | ✅ Yes |


In **Java terms**

**Concurrency**

ExecutorService

CompletableFuture

Virtual threads (Java 21)


One thread handling many tasks by switching


**Parallelism**

Multiple threads running on multiple cores

ForkJoinPool

Parallel streams (stream().parallel())

**Kafka concurrency**

* One consumer thread polls messages
* It can hand them to worker threads
* Tasks overlap in time but not necessarily simultaneously

**Kafka parallelism**

* Multiple consumers in the same group
* Each on different partitions
* Running on different CPU cores

Truly processing messages at the same time

What you should know about **Kafka for deep dives in interviews**



1\. **Scalability** (nombre de **partitions >** au nombre de **brokers max théorique**)

\-Aim for < 1MB per message

\-One Broker up to 1TB data \& 10k messages/s

How to scale:

\-more brokers

\-choose a good partition **key**



How to handle a **hot partition?**

\-Remove the key (if you don't need message ordering)  *A faire dans le producer*

\-Compound Key - AdId:N (split across N partitions) or AdId:UserId  *A faire dans le producer*

\-Backpressure - Slow down the producer



2\. **Fault Tolerance** \& Durability

What happens when **a consumer goes down**?

\-. Just read the latest **offset**

\-. If a consumer group, **rebalance**



Le **consumer** **store en local** le **offset** de **lecture** dans la partition et **commit (envoi) régulière** cette **valeur au** Kafka **broker**. !!Si le consommateur rencontre une **erreur en lisant le message** il peut **ne pas commit l'offset au broker**. !! So **the consumer is the one that commits offsets**, not the broker.



The consumer offset is the position in a partition that a consumer has processed up to.

\-**Consumer** decides when it is processing a message.

\-**Kafka broker** **stores** the **committed offset** in a **special topic** (\_\_consumer\_offsets) or allows external storage (rare).


Le **commit** par le **consumer** peut être **automatique** (au bout **d'une période, toutes les 500ms**) ou **manuel** (après avoir **lu avec succès** il fait **ensuite un commit**)



!Commiting means "I **successfully processed everything** **before** **this offset**." !!If you **commit too early**, you **risk losing messages.** So If the **consumer: "read the message BUT failed to process it"** then usually: **"the offset should NOT be committed".**



3\. **Errors** \& **Retries**

**Producers retries, Consumer retries**


4\. Performance **Optimizations**

\-**Batch** messages in **Producer** (produire des messages **en lot**)

\-**Compress** messages in **Producer** (produire des messages **sous forme compressé Ex: .zip**)

5\. **Retention** Policies (**durée** **conservation** des **messages**)

Two settings

1\. retention.**ms** (**default 7 days**) - how long to keep a message

2\. retention.**bytes** (**default 1GB**) - when to start purging based on size



=====================================

This is exactly where many people get confused because there are actually **2 different fault-tolerance systems** in Kafka:


1. **Partition replication fault tolerance** → for *topic data*

Example:

Topic Orders

Partition P1 replicas:

\- Broker 1

\- Broker 4

\- Broker 7

RF = 3   (Replication Fault)

This determines:
* message **durability**
* **partition** **availability**
* **consumer/producers** ability to **read/write**


**2. Controller quorum fault tolerance** → for *metadata/control plane*

These are **separate concepts**.

!!The **controller quorum** is separate from the **broker cluster size (nombre de brokers dans le cluster)**, though **controllers are themselves** Kafka **brokers**.


Example:

Controllers:

\- Broker 2

\- Broker 5

\- Broker 8

This determines:

* who is **leader** for **partitions**
* **topic creation**
* **broker registration**
* **metadata consistency**



**Can we have brokers that are not controllers at all? Yes**



Example: 7 total brokers

**Controllers:**

\- Broker 1

\- Broker 2

\- Broker 3

**Regular brokers only:**

\- Broker 4

\- Broker 5

\- Broker 6

\- Broker 7

Only Brokers **1–3 participate** in the **Raft metadata quorum**.

The **others**:
* store partitions
* serve producers/consumers
* are **NOT involved in controller election**

**Configuration to control this:** KAFKA\_CFG\_PROCESS\_ROLES,  KAFKA\_CFG\_LISTENERS etc

process.roles=**broker**,\*\*controller (\*\*process.roles ou KAFKA\_CFG\_PROCESS\_ROLES)

or

process.roles=**controller**

**or**

process.roles=**broker**


====================================================================================

Nodes (similaire aux **nodes** dans Kubernetes)  => **brokers**

**Partitions** (similaire aux **pods** dans Kubernetes)


| Feature                | **Zookeeper** (old)       | **KRaft** (new since kafka 2.8+)                  |
| ---------------------- | --------------------- | ---------------------------- |
| External dependency    | Zookeeper cluster     | No external system           |
| Metadata replication   | ZK replication        | Raft-based internal          |
| Leader election        | ZK election           | Raft election                |
| Operational complexity | Higher                | Lower                        |
| Metadata consistency   | Eventual (ZK watches) | Strong via Raft              |
| Controller failures    | Needs ZK failover     | Raft quorum handles failover |


#### 4\. Controller quorum size and fault tolerance (on *metadata/control plane)*

**Recommended**: **3, 5, or 7** controllers (**odd number** for Raft majority)

The cluster can **tolerate up to N/2 failures**, where **N** = **number of *controllers***

| Controllers | Max failures tolerated | Notes        |
| ----------- | ---------------------- | ------------ |
| 3           | 1                      | Majority = 2 |
| 5           | 2                      | Majority = 3 |
| 7           | 3                      | Majority = 4 |



#### 5\. Controller configurations

Kafka allows different ways to **configure** the **controller quorum**:


##### a) Dedicated controllers

Some brokers are **controller-only nodes** (don’t store regular partitions, i.e des partitions qui vont contenir des data des appli)(ou ne stoke que des **partitions** qui vont contenir des **métadata**)

**Pros**:
* Reduces load on brokers handling client traffic
* More predictable performance for metadata operations

**Cons**:
* More nodes needed
* Slightly more complex to maintain



Exemple: Some nodes are ONLY controllers. (large production clusters)

&#x20;              CLIENTS

&#x20;                  |

&#x20;   -----------------------------------------

&#x20;   |         |         |         |         |

+---------+ +---------+ +---------+ +---------+

|Broker 1 | |Broker 2 | |Broker 3 | |Broker 4 |

|Data only| |Data only| |Data only| |Data only|

+---------+ +---------+ +---------+ +---------+



&#x20;         Metadata / Raft quorum

&#x20;   --------------------------------

&#x20;   |              |              |

+-------------+ +-------------+ +-------------+

| Controller1 | | Controller2 | | Controller3 |

| NO data     | | NO data     | | NO data     |

+-------------+ +-------------+ +-------------+

**Advantages**
- more stable metadata management
- better for large/high-throughput clusters
- controllers stay responsive under heavy load

**Disadvantages**
- more infrastructure
- slightly more operational complexity


#### b) Combined controllers

Any broker can be a **controller broker** and a **data broker**

**Pros**:
* Fewer nodes
* Simpler deployment

**Cons**:
* High metadata operation load can compete with client traffic



Example: Combined Broker + Controller (small/medium clusters). Every controller is also a normal broker

               CLIENTS

                  |

     --------------------------------

     |              |              |

+-------------+ +-------------+ +-------------+

| Broker 1    | | Broker 2    | | Broker 3    |

| Controller  | | Controller  | | Controller  |

| Partitions  | | Partitions  | | Partitions  |

+-------------+ +-------------+ +-------------+

       (all nodes do both jobs)



Each node:
* stores partitions
* serves producers/consumers
* participates in controller quorum

**Advantages**
* simpler deployment
* fewer machines
* good for smaller clusters

**Disadvantages**
* metadata operations compete with data traffic
* heavy traffic may affect controller responsiveness

