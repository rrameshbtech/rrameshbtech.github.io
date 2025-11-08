Hello friends!

In my recent works I have been working with Aerospike database. It was a fun ride and had some interesting learnings. Though I was working NOSQL databases for quite some time, Aerospike was a new experience for me. It has some unique features and characteristics that set it apart from other databases. At some point, I thought if I knew what ever I learned through my battles with Aerospike, how much it would have changed the game for me. This post is the result of that thought process. If you are starting with Aerospike or planning to use it, here are some things I wish you knew which could help your ride smoother.

## A quick intro

A NOSQL database designed for predictable high performance using their Hybrid Memory Architecture, easy scaling out by the ability to add nodes with zero downtime, reducing the total cost of ownership for in-memory DB like performance by effective usage of SSDs using their HMA architecture.

In the era of AI where context is the king, I think Aerospike is a undeniable choice to consider for your database needs.

## Which storage options to choose?

Aerospike provides multiple storage options where we need to choose the right one based on our use case.

1. **Hybrid Memory Architecture (HMA)**: The default Aerospike configuration, where indexes are stored in RAM and data is stored on SSDs. This provides a good balance between performance and cost.
2. **In-Memory**: Both indexes and data are stored in RAM. This option provides the highest performance but is more expensive compared to other options.
3. **All in Flash**: Both indexes and data are stored on SSDs. This option is more cost-effective than in-memory but may have slightly higher latency.

For more details, refer to the [official documentation](https://aerospike.com/docs/database/learn/architecture/hybrid-storage/).

### My experience

Our use case is little unique where parts of our system needs ultra low latency and some other parts can work with slightly higher latency but needs to store large amount of data. During our initial exploration our main focus was to get the setup which has lower cost of ownership but with required performance. So we choose All Flash storage options with below rationale.

- Number of records we store is huge where a single record size is smaller. So storing indexes in RAM will be costly.
- Our read latency requirements seemed to match with the benchmarks provided by Aerospike for All Flash storage.

But when we started building the system and using with real time traffic, we realized that there were gaps between our expectations and reality. This caused a lot of side effects which we will discuss in coming sections. But 2 key learnings we got from this experience are,

- Not all parts of our data is huge in numbers which was a key deciding factor for us to choose All Flash. Especially our primary outcome we produce or consume more frequently is smaller in numbers (comparing to other data we store) and needs ultra low latency.
- Though the benchmarks provided by Aerospike are ideal, in real time when you are calculating number of reads and writes we have to be very careful. Some of the key factors we would have considered better are,
  - In your application when you read one domain object does not mean one DB read. You may need to read multiple records to construct the domain object.
  - Similarly when you write to a domain object, in most of the cases you may not write all your changes in one go. You may have multiple writes to the same domain object.
  - Apart from your applications, multiple DB background operations does read and write to the DB which will add to your total read and write calculations.

![An image of person who is struggling to choose between 3 options](https://via.placeholder.com/150)

Though All Flash storage option looked good on paper, in reality it found it is not the best fit. So based on our learnings we re-architected our system to use both HMA and All Flash storage options for different parts of our system. This helped us to achieve the desired performance and cost balance.
So the key takeaway here are

- When you are calculating your usage pattern & expectations we need to be very careful and consider not just application requirements but also DB background operations.
- When you are building a system, not all components of your system is going to handle similar data and not all going to have the same performance requirements.

## A record which is bigger than 1MB?

In my experience with NOSQL DBs, size of the records weren't been in a talk. Most of the NOSQL DBs allow storing adequately bigger records. Even though we store denormalized data and indexes there weren't much hassle(Though there could be some performance impact when size grows). But in Aerospike, the default maximum size of a record is 1MB. If you cross that line then you will get "Error Code 13: Record Too Big" error.

Though you can change this limit by changing the `max-record-size` parameter in the Aerospike configuration file, increasing this limit have cascading effects on the performance. For example, larger the record takes more disk I/O during replication. So it is good to stick to the default limit as much as possible.

### My experience

When we are using NOSQL DBs as primary data stores, there will be scenarios where we need to store larger records. In our case, we needed to store the indexes between multiple domain objects to support our querying needs. Lets take an example of Grocery store application which maintains multiple outlets. By the end of the day, we need to take the report of sales happened in every store.

To do this I may need to create indexes (secondary indexes in some DBs) which will keep the mapping between store and sales. But in most of the cases, especially when you volume of sales is huge, it will impact the performance. So it is good to create our own indexing mechanism which stores day-wise sales for each store.

```json
{
  "PrimaryKey": "Store:[StoreID]_Date:[Date]",
  "sales": {
    "SaleID1": { "item": "Item1", "quantity": 2, "amount": 20 },
    "SaleID2": { "item": "Item2", "quantity": 1, "amount": 15 },
    ...
  }
}
```

When I trying to build such indexes, I faced the "Error Code 13: Record Too Big" error multiple times. To solve this problem, I had to rethink my indexing strategy. Instead of storing all sales for a store in a single record, I decided to store them in chunks. I may try splitting using natural chunks like per hour, per category etc but this case the risk of growing bigger especially during peak hours.

So I decided to split based on fixed size. Aerospike has a suggested approach called "Adaptive Map" to address this issue. You can find their sample implementation [here](https://github.com/aerospike-examples/adaptive-map). But we implemented a different flavour of it using ["filter expressions"](https://aerospike.com/docs/database/learn/tutorials/quick-start-expression-index/#_top) for some internal reasons.

Read here why [Aerospike has this limit](https://discuss.aerospike.com/t/how-to-solve-the-problem-error-code-13-record-too-big/2129/8).

## Is your key too hot?

It is quite common that one record is being read/written by more than one client at the same time, especially when you have denormalized data models. In most of the DBs this impacts the latency & some time the request may even timeout. But in Aerospike, if they are too many concurrent operations on the same record, Aerospike will throw "Error Code 14: AS_ERR_KEY_BUSY" error. I presume that this is their fail fast mechanism to avoid performance degradation due to lock contentions.

Usually Aerospike, maintains queue for the pending transactions. This helps them to maintain the sequential consistency for the records. But when the pending transactions cross a threshold limit, Aerospike will start throwing this error. `transaction-pending-limit` parameter in the Aerospike configuration file controls this limit. The default value is 20.

Read more about limit [here](https://aerospike.com/docs/database/reference/config/#namespace__transaction-pending-limit)

### My experience

In our application, we have denormalized domain objects which are being kept updated by huge incoming traffic while on the other end these objects are being read by multiple clients. In our cause this issue was more frequent because of below reasons as per our analysis,

1. **All Flash storage** : Since we were using All Flash storage option, the read and write latencies were slightly higher compared to HMA. This increased the chances of concurrent operations on the same record.
2. **Strong consistency** : We were using strong consistency for our read and write operations. This means every write need to wait for acknowledgement from all the replicas before returning success. This increased the time taken for write operations, leading to more chances of concurrent writes on the same record.
3. **Un-replicated Records** : Due to above strong consistency requirement, we get un-replicated records in some scenarios. Whenever client's try to read/write to such records, Aerospike tries to re-replicate them first before serving the request. This increases the chances of concurrent operations on the same record.

Below are some of the strategies we used to mitigate this issue,

- **Increase transaction-pending-limit** : We increased the `transaction-pending-limit` parameter in the Aerospike configuration file from 20 to 100. This helped to reduce the frequency of "AS_ERR_KEY_BUSY" errors.
- **Increase timeouts** : We increased the [client-side timeouts](https://aerospike.com/docs/develop/client/java/policies/#basic-timeout-settings) for read and write operations. Though this might help to reduce this error by reducing "un-replicated records" issue, it is required even for the previous strategy to work as the bigger queue will take more time to process.
- **Buffered Writes** : During our discussion with Aerospike experts, they suggested buffering the write operations based on the objects and execute them in once. As every system/component of system has different needs it needs to be strategized accordingly and implemented.

## The predictable Primary Index

Aerospike builds primary index using distributed hashmap and red-black tree(sprigs). This method supports them to maintain the predictable low latency. For each record in the namespace, it creates a 20 byte hash (RIPEMD-160 hash) which helps identifying it's partition and storage location of where the record is stored. So Aerospike can directly reach the record and fetch it with predictable latency.

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

## References

- [Aerospike Error Codes](https://aerospike.com/docs/database/reference/error-codes)
