---
layout: post
title: "Aerospike: My 'Aha!' moments - The predictable Primary Index"
date: 2025-11-13 00:00:00 +0530
categories: aerospike learning
tags: aerospike primary index RIPEMD-160 hash spring
published: true
excerpt_separator: <!--more-->
---

![A women software programmer wonder by looking at the smart aerospike client looking like bot talking to servers](/assets/images/aerospike-banner-primary-index.jpg "A women software programmer wonder by looking at the smart aerospike client looking like bot talking to servers")
Aerospike builds primary index using distributed hashmap and red-black tree(sprigs). This method supports them to maintain the predictable low latency. For each record in the namespace, it creates a 20 byte hash (RIPEMD-160 hash) which helps identifying it's partition and storage location of where the record is stored. So Aerospike can directly reach the record and fetch it with predictable latency.

<!--more-->

I wrote short notes on some important concepts below.

- **User Key** : A unique text key specified by the user for the record
- **Digest** : A 20 byte hash created by RIPEMD-160 hash function using user key, set name, user key type
- **Partition ID** : 12 deterministic bits of the Digest used to identify the partition
- **Partition Map** : Map which helps to identify which node has master or replica of a particular partition.
- **Smart Client** : An aerospike client in consumer side which connects to server. It gets the partition to identify which node it needs to reach for fetching particular record.
- **Sprigs** : A group of red-black trees available in each partition to build primary index. It helps identifying the record metadata using the digest.
- **Primary Index Entry** : Along with digest, it also contains expiry time (void_time), generation, last updated time, storage pointer, replication state. It has fixed size of 64 bytes.

Based on my understanding, I explained the flow of how a client fetches a record.

1. The client computes the digest from namespace, set, and user key.
2. Partition ID is calculated from digest.
3. The partition map tells the client which node holds the master (and replicas) for the partition.
4. Request is routed directly to the correct node.
5. Node looks up the digest in the correct partition’s red-black tree sprig.
6. If present, the tree entry points directly to the storage device and record can be read or written in O(log n) time

I drew a diagram to depict how primary index works in Aerospike.
![Aerospike Primary Index](/assets/images/aerospike-things-you-may-need-to-know-primary-index.jpg "Flow of how aerospike uses primary index to fetch the respective record")

### My experience

Primary index plays vital role in which storage method we should be choosing. For example, in our case, volumes of records and it's impact in size of primary index lead us to start using All Flash storage option. Because with huge volume of records, the space required to store primary index also increases. So using HMA will significantly increase the TOC. An ultra simple formula for calculating the size of primary index given below.

```
Primary Index Size = 64 bytes * Record Count * Replication factor
```

For more detailed planning, please check [capacity planning](https://aerospike.com/docs/database/manage/planning/capacity/) page.

#### Where is my key?

One of shock in initial days was, when you are querying/batch fetch records from Aerospike, you will get your record user key in your response. Yes, it is Aerospike's default behavior that it does not store record user key along with the record. As we have seen above, it creates the digest which used to create your primary index. So you won't get it back it was not stored at all.

When you are fetching the record by your record user key (primary key), you do not need it as you already have it in your hand. But some other times you may prefer to get record user key along with your record. If yes, then you can update your client policy to ask Aerospike to store & retrieve the key. Remember that you have to send this flag for both write as well as read policy.

```java
policy.sendKey = true;
```

In some cases, we experienced inconsistency is storing & fetching keys. Though we are unable to find the exact cause. Below are the workarounds we tried.

- Set `sendKey=true` in your default policies of the client. Ensure that in every place you are using default policy or extending from default policy. Sometimes, developers replace the default policy by unknowingly overriding it by code like below. Anyway sendKey is used as guard against the rare collision in RIPE-MD160 hash.

```java
client.operate(new WritePolicy(), operations);
```

- If the above one does not work, to make getting key predictable, store the key as separate bin in the record. Refer [here](https://support.aerospike.com/s/article/FAQ-Why-isn-t-the-Primary-Key-returned-when-I-read-a-record) for more details

- More details on [Primary Index](https://aerospike.com/docs/database/learn/architecture/data-storage/primary-index/)
- More details on to configure [Primary Index](https://aerospike.com/docs/database/manage/namespace/primary-index)
- [How to retrieve Key in Key-Value Store?](https://aerospike.com/blog/how-to-retrieve-the-key-in-a-key-value-store/)

### All articles in this series

1. [Which storage options to choose?](https://rrameshbtech.github.io/aerospike,/learning,/db/2025/11/09/aerospike-aha-moments.html)
2. [A record which is too big](https://rrameshbtech.github.io/aerospike/learning/2025/11/10/aerospike-aha-moments-2.html)
3. [Is your key too hot?](https://rrameshbtech.github.io/aerospike/learning/2025/11/11/aerospike-aha-moments-3.html)
4. [The predictable Primary Index](#)
