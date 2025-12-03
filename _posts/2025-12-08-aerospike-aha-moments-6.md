---
layout: post
title: "Aerospike: My 'Aha!' moments - A summary"
date: 2025-12-03 00:00:01 +0530
categories: aerospike learning
tags: aerospike data structure index hma denormalization toc
published: true
excerpt_separator: <!--more-->
---

![The software programmer flying in Aerospike logo shaped jet crossing hurdles successfully](/assets/images/aerospike-key-learning-banner.png "The software programmer flying in Aerospike logo shaped jet crossing hurdles successfully")
Though the learning never stops, I think it is good to summarize my experience with both Aerospike and building a general purpose solution using it. It may recall some of the ideas and concepts discussed in previous articles. So bear with me and buckle on for the ride.

<!--more-->

Let's quickly glance at the context so it will be helpful for you to understand some of the specifics we will be discussing. I could not reveal the exact context, but let me share an abstracted version. Our goal is to replicate a high volume & frequently updating normalized data in near real time from our legacy application to our DB (Aerospike) and generate capabilities based on those data. So this creates a parallel system to our legacy one. These capabilities will be in the form of APIs, Events & more. They are consumed by critical customer facing systems.

![Architecture diagram of the context application](/assets/images/aerospike-context-application.png "Architecture diagram of the context application")

Given above context let me take you through some of our problem statements related to data and how we solved them.

## Replicate at near real time

One of our critical need is to replicate the data from legacy system at real time. Irrespective unavoidable latencies introduced by multiple intermediaries, our goal still stays as near real time. So our needs are

1. Ability to write as faster as possible
2. Ability to store large volume of data
3. Ability to keep lesser total cost of ownership & more

As we have seen in the [first article](https://rrameshbtech.github.io/aerospike,/learning,/db/2025/11/09/aerospike-aha-moments.html), Aerospike provides 3 storage options. We had to choose the best fit for our use case. Neal Ford & Mark Richards quote in [this article](https://www.oreilly.com/live-events/software-architecture-superstream-software-architecture-trade-offs/0636920083581/) that **everything in software architecture is a trade-off**. Here are our trade-offs and rationale below.

- **In Memory**: Yes, it is faster. But given the volume of data it will shoot up our TOC.
- **HMA**: Faster, Yes. But it still costs us more because of the volume of data. In Aerospike, cost is not just decided by the total size of data but also by volume (number of records). Refer how [primary keys work here](https://rrameshbtech.github.io/aerospike/learning/2025/11/12/aerospike-aha-moments-4.html)
- <span style="background-color:PaleGreen;color:#666">**All Flash**: With some trade-offs in latency, we chose this storage option. This helps us to reduce the total cost of ownership and achieve faster writes with occasional reads (We will see why we read lesser in below sections).</span>

After choosing the storage options, still there are other rotary knobs you can tune to get the perfect result you want. Based on my experience I have given some of them below.

### Where & how to store primary keys?

Aerospike allows flexibility to configure how and where do you store your primary keys. You can configure `mount` and `mount-budget` under `index-type` configuration and `partition-tree-sprigs` based on total number of records.

Ensuring that your primary index is spread as per your use case will help to avoid any bottle-necks later. For example, if all the primary indexes are over loaded in one node then it will become a bottleneck as that particular node's I/O will be overloaded with frequent PI reads/writes.

Please refer [Aerospike documentation](https://aerospike.com/docs/database/manage/planning/capacity#calculating-primary-index-storage) for more details on capacity planning and [All Flash FAQ](https://aerospike.com/docs/database/manage/planning/ssd/faq/).

### Size & count of nodes

We need to find the balance between size and count of nodes. Both are inversely proportional, as the total size of records will be constant. There is no one size fits all. Initially we had larger nodes in lesser counts, which resulted in higher migration and cold-restart durations. Remind that IO is not just from external read/writes but also from other background jobs run in the server. So based on our observations, we migrated to smaller nodes with more count. Remember Aerospike supports scaling out based on number of nodes. So we can increase the nodes linearly based on when & how we need it.

In contrary, if we go too fragmented, it will increase operational complexity in maintenance. So again balancing is the art here.

### Update vs Replace

As All-Flash option is comparatively slower, we need to make sure we tune every knobs possible to keep the latency under control. One such is setting `RecordExistsAction` to `replace` while replicating the data. In our case, we replicate the data. So every time we get the whole record through CDC which we have to directly replaced in Aerospike. By setting `replace` for `RecordExistsAction` configuration, Aerospike can do better as it might avoid reading the existing record. This option is supported in [Aerospike sink connector](https://aerospike.com/docs/connectors/streaming/kafka/inbound/) also.

Read [here](https://aerospike.com/docs/develop/client/csharp/usage/atomic/update/#replace) for more information.

## Serve all the needs

Like we have seen above, we get large volume of normalized data and replicated in a No-SQL DB (Aerospike). Our client does not only fetch the data by primary key but also query, filter, aggregate & so on. As our source data is normalized, for getting a user's basic information we need to fetch data from 5-10 tables(sets) to respond to one request. Initially we started with light transformation approaches due to other constraints, but we ended up failing to meet the non functional requirements. So we had to pivot our strategy and keep the data stored in a format which is native to NO-SQL and supports our data fetching patterns. Yes, we ended up with denormalizing before storing.

I explained our initial data modeling approach and what we missed below. Here are the steps we followed during our initial light weight transformation.

1. Store the source data as one JSON bin.
2. Extract & transform specific fields from source as separate bins.
3. Create secondary indexes on the extracted fields which client may query on.
4. Store the secondary indexes **in memory** so it help querying faster.
   So we ended up storing data like below

```java
// UserPersonalInformation
{
  "PK": "user-key-1",
  "source": {
    "firstName": "Test",
    "lastName": "User"
    // ..more fields
  },
  "name": "Test User", // create secondary index for this field
  "email": "test@domain.com"
}

// UserAddresses
{
  "PK": "user-key-1",
  "source": {
    "isActive": "yes",
    "lane": "Thendral nagar",
    "city": "Coimbatore"
    // ..more fields
  },
  "city": "Coimbatore" // create secondary index for this field
}
```

Some of the factors we less focused which costed us heavily later are,

1. Though secondary index is stored in memory and reading the index is faster, in Aerospike, querying uses [scatter gather approach](https://aerospike.com/blog/working-with-query-result-streams/#query_execution). So even if you send a query to one node, it broadcasts to all other nodes gets the node responses and build the final response.
2. Even though Aerospike query supports more than one condition, the nodes run query with one condition only then after collecting the cumulative response from all nodes it does further filtering. So it costs extra time and processing for the node.
3. In real time, when using normalized data structure, one data read does not mean one db read. Based on the relationships, the number of time a table being read to serve one client API request might grow up to thousands. So calculating real db record level read will show us the original latency.

### Denormalize incrementally

So we decided to denormalize the source data and store it in format which is closer to support our reading patterns. But this immediately popped up the next challenge of handling the additional over head of denormalizing. Especially when we have to

1. Ensure to check the incoming data is latest and whether can we update a specific fragment of data
2. When aggregating
3. Creating additional indexes and eliminating older ones

These all required additional reads and updates. So after a lot of pondering we decided to take incremental approach to denormalize rather than reading the existing record to make any decision and then update. Yes, this is very much contextual and depends on a lot of other factors too. For example, if a `bin`(column) need to keep the number of active posts by a user. In posts CDC processor,

1. Increment `activePosts` by 1 if it is a new post.
2. Decrement `activePosts` by 1 if an existing post is deactivated
3. Do not change, if the incoming is a change to active post (A post record with `op-type=update` and `status=active`).

We heavily rely on Aerospike filter expressions to decide whether a particular record should update or not. Refer [the documentation](https://aerospike.com/docs/develop/client/java/usage/atomic/expressions/#write) for more details. For example, below code writes to address only if the `op-time` of CDC record is latest than the `addressUpdatedAt` in denormalized record.
_Note: addressUpdatedAt(last updated CDC's op-time) is the additional metadata we track in denormalized record._

```java
WritePolicy policy = client.getWritePolicyDefault();
// Build the expression
policy.filterExp = Exp.build(
        Exp.or(
            Exp.binExists("addressUpdatedAt"),
            Exp.lt(Exp.intBin("addressUpdatedAt"), Exp.val(opTime))
        )
    )
);
// Update the record
client.operate(null, key, writeAddressOperation);

// Close the connection to the server
client.close();
```

The sample user record after denormalization looks like below.

```java
// User
{
    "name":{
        "firstName": "Test",
        "lastName": "User"
    }
    "address": {
        "lane": "Thendral nagar",
        "city": "Coimbatore"
        // ..more fields
    },
    "activePosts": 5,
    "addressUpdatedAt": 123857362
}
```

### HMA for denormalized data

As now we have clear segregation of data, we revisited our storage option again. Because with the denormalized data, the volume (number of records) got reduced significantly in most of the cases. So we decided to reconsider HMA for denormalized data. Now we can have 2 namespaces as given below.

1. **replicated-namespace**: It contains the replicated data in normalized structure and keeps our TOC low.
2. **denormalized-namespace**: It will have the sets which are denormalized and read actively by the end consumer applications.

This helps us to get better performance with comparatively lesser TOC. Find the high level architecture after implementing denormalization.

![Architecture of application with denormalization](/assets/images/Aerospike-denormalized.png "Architecture of application with denormalization")

With the above fine tunings we were able to get the results we wanted. But yes, it does not stop here and we have to keep aligning to the incoming changes like Kent Beck quotes in his book [Extreme Programming](https://www.amazon.in/Extreme-Programming-Explained-Embrace-Change/dp/0321278658) that software development is like doing small adjustments to your steering even you are driving on a highway. So that you will be on track.

## All articles in this series

1. [Which storage option to choose?](https://rrameshbtech.github.io/aerospike,/learning,/db/2025/11/09/aerospike-aha-moments.html)
2. [A record which is too big](https://rrameshbtech.github.io/aerospike/learning/2025/11/10/aerospike-aha-moments-2.html)
3. [Is your key too hot?](https://rrameshbtech.github.io/aerospike/learning/2025/11/11/aerospike-aha-moments-3.html)
4. [The predictable Primary Index](https://rrameshbtech.github.io/aerospike/learning/2025/11/12/aerospike-aha-moments-4.html)
5. [Resurrection of the record](https://rrameshbtech.github.io/aerospike/learning/2025/11/13/aerospike-aha-moments-5.html)
6. [A Summary](#)
