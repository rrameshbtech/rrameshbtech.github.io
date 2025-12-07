---
layout: post
title: "Aerospike: My 'Aha!' moments - The predictable Primary Index"
date: 2025-11-13 00:00:01 +0530
categories: aerospike learning
tags: aerospike primary index RIPEMD-160 hash spring
published: true
excerpt_separator: <!--more-->
---

![A women software programmer wonder by looking at the smart aerospike client looking like bot talking to servers](/assets/images/aerospike-banner-primary-index.jpg "A women software programmer wonder by looking at the smart aerospike client looking like bot talking to servers")
Aerospike builds primary index using distributed hashmap and red-black tree(sprigs). This method supports them to maintain the predictable low latency. For each record in the namespace, it creates a 20 byte hash (RIPEMD-160 hash) which helps identifying it's partition and storage location of where the record is stored. So Aerospike can directly reach the record and fetch it with predictable latency.

<!--more-->

I wrote short notes on some of the important concepts below. It will help in understanding how it works.

- **User Key** : A unique text key specified by the user for the record.
- **Digest** : A 20 byte hash created by RIPEMD-160 hash function using user key, set name and the user key type.
- **Partition ID** : 12 deterministic bits of the Digest which is used to identify the partition.
- **Partition Map** : Map which helps to identify which node has master or replica of a particular partition.
- **Smart Client** : An aerospike client at the consumer which connects to server. It knows which node it should reach for fetching a specific record with the help of partition map.
- **Sprigs** : A group of red-black trees available in each partition to build primary index. It helps identifying the record metadata using the digest. It helps to maintain the shortest traverse to get the location of a record.
- **Primary Index Entry** : Along with digest, it also contains expiry time (void_time), generation, last updated time, storage pointer, replication state. It has the fixed size of 64 bytes.

Based on my understanding, I explained the flow of how the smart client fetches a record.

1. The client computes the digest from namespace, set, and user key.
2. Partition ID is calculated from digest.
3. The partition map tells the client which node holds the master (and replicas) of the given partition.
4. The request is routed directly to the master node.
5. The node looks up the digest in the respective partition’s red-black tree sprig.
6. If present, the tree entry points directly to the storage location through the pointer and record can be read with O(log n) time.

I drew a diagram to depict how primary index works in Aerospike.
![Aerospike Primary Index](/assets/images/aerospike-things-you-may-need-to-know-primary-index.jpg "Flow of how aerospike uses primary index to fetch the respective record")

### My experience

Primary index plays vital role in choosing which storage method we should be using. For example, in our case, volumes of records and it's impact in size of primary index lead us to start using All Flash storage option. Because with huge volume of records, the space required to store primary index also increases which impacts our TCO. So choosing HMA seemed costlier. I have given an ultra simple formula for calculating the size of primary index given below.

```
Primary Index Size = 64 bytes * Record Count * Replication factor
```

For more detailed planning, please check [capacity planning](https://aerospike.com/docs/database/manage/planning/capacity/) page.

#### Where is my key?

One of the shock in initial days was, when you are querying/batch fetch records from Aerospike, you will not get your record user key in your response. Yes, it is Aerospike's default behavior that it does not store record user key along with the record. As we have seen above, it creates the digest which used to create your primary index. So you won't get the user key back as it was not stored at all.

When you are fetching the record by your record user key (primary key), you do not need it as you already have it in your hand. But in some other times you may prefer to get record user key along with your record. If yes, then you can update your client policy and ask Aerospike to store & retrieve the key. Remember that you have to send this flag for both write as well as read policies.

```java
policy.sendKey = true;
```

In some cases, we experienced inconsistency in storing & fetching the keys. We were unable to find the exact reason as there were one specific cause. But I would recommended to start with below one.

- Set `sendKey=true` in your default policies of the client. Ensure that in every place you are using default policy or extending from the default policy. Sometimes, developers replace the default policy unintentionally by overriding it with the code like below. "sendKey" is also used as guard against the rare collision in RIPE-MD160 hash.

```java
client.operate(new WritePolicy(), operations);
```

- If the above one does not work, to make getting key predictable, store the key as separate bin in the record. Refer [here](https://support.aerospike.com/s/article/FAQ-Why-isn-t-the-Primary-Key-returned-when-I-read-a-record) for more details

- More details on [Primary Index](https://aerospike.com/docs/database/learn/architecture/data-storage/primary-index/)
- More details on to configure [Primary Index](https://aerospike.com/docs/database/manage/namespace/primary-index)
- [How to retrieve Key in Key-Value Store?](https://aerospike.com/blog/how-to-retrieve-the-key-in-a-key-value-store/)

### All articles in this series

1. [Which storage option to choose?](/aerospike,/learning,/db/2025/11/09/aerospike-aha-moments.html)
2. [A record which is too big](/aerospike/learning/2025/11/10/aerospike-aha-moments-2.html)
3. [Is your key too hot?](/aerospike/learning/2025/11/11/aerospike-aha-moments-3.html)
4. [The predictable Primary Index](#)
5. [Resurrection of the record](/aerospike/learning/2025/11/13/aerospike-aha-moments-5.html)
6. [A Summary](/aerospike/learning/2025/12/02/aerospike-aha-moments-6.html)
