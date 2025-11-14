---
layout: post
title: "Aerospike: My 'Aha!' moments - A record which is too big"
date: 2025-11-11 00:00:00 +0530
categories: aerospike learning
tags: aerospike storage record size chunk
published: true
excerpt_separator: <!--more-->
---

![A women having bigger data blocks in hand and wondering how it will fit in 1 MB limit gate of Aerospike](/assets/images/aerospike-banner-1mb.png "A women having bigger data blocks in hand and wondering how it will fit in 1 MB limit gate of Aerospike")
In my experience with NOSQL DBs, size of the records were in a talk rarely. Most of the NOSQL DBs allow storing adequately bigger records. Even though we store denormalized data and indexes there weren't much hassle(Though there could be some performance impact when the size grows). But in Aerospike, the default maximum size of a record is 1MB. Yes. It is. If you cross that line then you will get "Error Code 13: Record Too Big" error.

Though you can change this limit by configuring the `max-record-size` parameter in the Aerospike configuration file, increasing this limit will have cascading effects on the performance. For example, larger the record takes more disk I/O during replication. So it is good to stick to the default as much as possible.

<!--more-->

### My experience

When we are using NOSQL DBs as primary data store, there will be scenarios where we need to store larger records. In our case, we needed to store the indexes between multiple domain objects to support our querying needs. Lets take an example of Grocery store application which maintains multiple outlets. By the end of the day, we need to take the report of sales done in every store.

To do this we may need to create indexes (secondary indexes in some DBs) which will keep the mapping between store and sales. But in most of the cases, especially when you volume of sales is huge, it will impact the performance. So it is good to create our own indexing mechanism which stores day-wise sales for each store. Please find the sample index below.

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

When I was trying to build such indexes, I faced the "Error Code 13: Record Too Big" error multiple times. To solve this problem, I had to rethink my indexing strategy. Instead of storing all sales for a store in a single record, I decided to store them in chunks. I may try splitting using natural dividers like per hour, per category etc but some chunks can cause the risk of growing bigger than expected especially during the peak hours.

So I decided to split them based on fixed size. Aerospike has a suggested approach called "Adaptive Map" to address this issue. You can find their sample implementation [here](https://github.com/aerospike-examples/adaptive-map). But we implemented a different flavour of it using ["filter expressions"](https://aerospike.com/docs/database/learn/tutorials/quick-start-expression-index/#_top) for some internal reasons.

Read here why [Aerospike has the max size limit for record](https://discuss.aerospike.com/t/how-to-solve-the-problem-error-code-13-record-too-big/2129/8).

### All articles in this series

1. [Which storage options to choose?](https://rrameshbtech.github.io/aerospike,/learning,/db/2025/11/09/aerospike-aha-moments.html)
2. [A record which is too big](#)
3. [Is your key too hot?](https://rrameshbtech.github.io/aerospike/learning/2025/11/11/aerospike-aha-moments-3.html)
4. [The predictable Primary Index](https://rrameshbtech.github.io/aerospike/learning/2025/11/12/aerospike-aha-moments-4.html)
5. [Resurrection of the record](https://rrameshbtech.github.io/aerospike/learning/2025/11/13/aerospike-aha-moments-5.html)
