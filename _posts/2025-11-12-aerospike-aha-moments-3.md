---
layout: post
title: "Aerospike: My 'Aha!' moments - Is your key too hot?"
date: 2025-11-12 00:00:00 +0530
categories: aerospike learning
tags: aerospike key busy
published: true
excerpt_separator: <!--more-->
---

![Software professional trying to hold aerospike logo shaped hot key](/assets/images/aerospike-banner-hot-key.jpg "Software professional trying to hold aerospike logo shaped hot key")
It is quite common that one record is being read/written by more than one client at the same time, especially when you have denormalized data models. In most of the DBs this impacts the latency & some time the request may even timeout. But in Aerospike, if there are too many concurrent operations on the same record, Aerospike will throw "Error Code 14: AS_ERR_KEY_BUSY" error. I presume that this is their fail fast mechanism to avoid performance degradation due to lock contentions.

<!--more-->

Aerospike maintains queue for the pending transactions. This helps them to maintain the sequential consistency for the record updates. But when the pending transactions cross a threshold limit, Aerospike will start throwing this error. You can configure the transaction pending queue limit by using `transaction-pending-limit` parameter in the Aerospike configuration file. The default value is 20.

Read more about this limit [here](https://aerospike.com/docs/database/reference/config/#namespace__transaction-pending-limit)

### My experience

In our application, we have denormalized domain objects which are being updated by huge incoming traffic while on the other end these objects are being read by multiple clients. In our cause this issue was more frequent because of below reasons as per our analysis,

1. **All Flash storage** : Since we were using All Flash storage option, the read and write latencies were slightly higher comparing to [HMA](https://aerospike.com/glossary/hybrid-memory-model/). This increased the chances of concurrent operations on the same record.
2. **Strong consistency** : We were using strong consistency for our read and write operations. This means every write need to wait for acknowledgement from all the replicas before returning success. This increased the time taken for write operations, leading to more chances of concurrent writes on the same record.
3. **Un-replicated Records** : Due to above strong consistency requirement, it produced un-replicated records in some scenarios especially when timeout happens before updating the replicas. Whenever any client try to read/write to such records, Aerospike will try to re-replicate them first before serving the request. This increases the chances of concurrent operations on the same record.

Below are some of the strategies we used to mitigate this issue,

- **Increase transaction-pending-limit** : We increased the `transaction-pending-limit` parameter in the Aerospike configuration file from 20 to 100. This helped to reduce the frequency of "AS_ERR_KEY_BUSY" errors.
- **Increase timeouts** : We increased the [client-side timeouts](https://aerospike.com/docs/develop/client/java/policies/#basic-timeout-settings) for the read and write operations. Though this might help to reduce this error directly by reducing "un-replicated records" issue, it is required even for the previous strategy to work as the bigger transaction pending queue demands bigger timeout as the operations waiting in the tail of the queue needs more time to get processed.
- **Buffered Writes** : During our discussion with Aerospike experts, they suggested buffering the write operations based on the objects and execute them in once. As every system/component of system has different needs it needs to be strategized accordingly and implemented.

### All articles in this series

1. [Which storage options to choose?](https://rrameshbtech.github.io/aerospike,/learning,/db/2025/11/09/aerospike-aha-moments.html)
2. [A record which is too big](https://rrameshbtech.github.io/aerospike/learning/2025/11/10/aerospike-aha-moments-2.html)
3. [Is your key too hot?](#)
4. [The predictable Primary Index](https://rrameshbtech.github.io/aerospike/learning/2025/11/12/aerospike-aha-moments-4.html)
