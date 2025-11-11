---
layout: post
title: "Aerospike: My 'Aha!' moments - Introduction"
date: 2025-11-10 00:00:00 +0530
categories: aerospike, learning, db
tags: aerospike storage hma hybrid flash ssd
published: true
excerpt_separator: <!--more-->
---

![Thing about Aerospike](/assets/images/aerospike-banner.jpg "A women software professional thinking about Aerospike")
Hello friends!

In my recent works I have been working with Aerospike database. It was a fun ride with lots of interesting learnings. Though I was working with NOSQL databases for quite some time, Aerospike was a new experience for me. It has some unique features and characteristics that set it apart from other databases. At some point, I thought if I knew what ever I learned through my battles, how much it would have changed the game for me. This post is the result of that thought process. If you are starting with Aerospike or planning to use it, here are some things I wish you knew which could help your ride smoother.

<!--more-->

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

![A person who is struggling to choose between 3 options](/assets/images/aerospike-struggling-to-choose-storage-option.png "A person who is struggling to choose between 3 options]")

Though All Flash storage option looked good on paper, in reality it found it is not the best fit. So based on our learnings we re-architected our system to use both HMA and All Flash storage options for different parts of our system. This helped us to achieve the desired performance and cost balance.
So the key takeaway here are

- When you are calculating your usage pattern & expectations we need to be very careful and consider not just application requirements but also DB background operations.
- When you are building a system, not all components of your system is going to handle similar data and not all going to have the same performance requirements.

---

**Note:** This article is growing more than I thought, so I prefer to split it into a series which can help us to ponder and discuss. See ya in the next one.

### All articles in this series

1. [Which storage options to choose?](#)
2. [A record which is too big](#)
