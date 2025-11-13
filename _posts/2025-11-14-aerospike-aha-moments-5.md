---
layout: post
title: "Aerospike: My 'Aha!' moments - Resurrection of the record"
date: 2025-11-14 00:00:01 +0530
categories: aerospike learning
tags: aerospike primary index tombstones tombraider defragmentation cold restart durable delete
published: true
excerpt_separator: <!--more-->
---

![A software professional surprised to see the aerospike shared records resurrecting from aerospike server racks](/assets/images/aerospike-banner-resurrection-of-record.jpg "A software professional surprised to see the aerospike shared records resurrecting from aerospike server racks")
Yes. You read it correct. It's not just Aerospike, I knew DBs like Cassandra also has this behavior. Not if any other DB also behave similar. This is because how they handle delete of their records. Though your deleted records will not always resurrect but it can come back in situations like cold restart. So it is good to know how Aerospike handles deletes to handle it effectively based on your use case.

<!--more-->

By default when you delete a record in Aerospike just removes the respective primary index from your memory. It does not really delete your data. So you data will still be there in your write blocks. So during cold restart, when Aerospike tries to rebuild primary index from data, it recreates the index of the deleted record. Confusing? Lets see in short what happens when you update, delete & durable delete.

### When you update a record

Aerospike create a new data record in the write block with new updated bins, latest LUT & generation and update your primary index's storage location to this newly added record. Still your old version of the record still be there in the disk. Refer first 2 boxes in below diagram.

### When you delete a record

As we knew, primary index is the ledger of you records, when you delete a record it really delete the primary index from the memory and does not touch the real data in the disk. Refer box 2 & 3 in below image.

As it just deleted only primary index, when cold restart runs, it will not have primary index in most cases. So it rebuilds the primary index by iterating through the records stored in write blocks (disk). So it might bring back the deleted records in below steps.

- Read the record in the write block.
- If no primary index available, add a new primary index (from metadata of the record) and updates storage location.
- If already exists, check the lut & generation, if the record just read has latest lut/gen then update the storage location to current record else ignore.

As in the above steps there is no way to identify whether a record has been already deleted, it will update the primary index with latest version available for the record.

### When you durably delete a record

As the default delete may not work as expected in some situations and it can bring back the deleted record during situations like cold restart, Aerospike also supports durable deletes. Let's check out how it really works. To enable durable delete, set `durableDelete` to `true` in your write policy.

```java
WritePolicy deletePolicy = new WritePolicy();
deletePolicy.durableDelete = true;
// Durably delete a record with the write policy
client.Delete(deletePolicy, key);
```

When a durable delete executed

1. Create a new data in write block with only metadata with latest LUT & Generation & no bins. It is called tombstone.
2. Update the primary index of the deleted record with to tombstone created in the above step.

As durable delete does not delete the primary index, but it creates a record with no bins (tombstone) and replaces that as the latest version, during cold restart the primary index for the record will be created and linked to the tombstone, not to any older version of the record. So whenever you try to access that record for any operation, as it is linked to tombstone it will not any invalid operations like read or write.

![Aerospike methodology to delete records](/assets/images/aerospike-durable-delete.jpg "Aerospike methodology to delete records. Delete just removes the index. Durable delete adds a tombstone instead")

### But why my older versions of record still available

Due to various reasons like performance optimization, strong consistency and various other reasons, when you update or delete a record, Aerospike does not really update you existing record in the write blocks(disk), instead it just creates new version of the record with latest bins & just changes your storage pointer. So the older versions of your record still remains until [defragment](https://aerospike.com/docs/database/manage/namespace/storage/defrag/) kicks in & does it's job.

You can control when to run defragmentation from your namespace level configuration. Especially [defrag-lwm-pct](https://aerospike.com/docs/database/reference/config#namespace__defrag-lwm-pct) helps to set the percentage of live records. If the live records percentage touches or goes below this level then defragmentation will kick-in.

### Will the tombstone stay forever?

No. Aerospike runs a special background job named [Tomb raider](https://aerospike.com/docs/database/learn/architecture/durable-deletes#tomb-raider) which will scan the tombstones & relevant primary index when there is no need for it. Mostly it removes tombstones in below scenarios

1. No older version of the record present. It might have been already removed by defragmentation.
2. [tomb-raider-eligible-age](https://aerospike.com/docs/database/reference/config#namespace__tomb-raider-eligible-age) exceeds
3. Tombstone is successfully shipped by XDR
   and in some more scenarios.

Refer below pages for more information

- [Durable delete](https://aerospike.com/docs/database/learn/architecture/durable-deletes)
- [When can durable deletes come back?](https://support.aerospike.com/s/article/When-can-durable-deletes-come-back)
- [Will my tombstones will stay forever?](https://support.aerospike.com/s/article/Durable-delete-tombstones-never-removed-in-system-where-load-is-extremely-low)
- How [cold restart](https://aerospike.com/docs/database/manage/database/cold-start/) works?
- How [Defragmentation](https://aerospike.com/docs/database/manage/namespace/storage/defrag/) works?
- [Do record has metadata to rebuild primary index?](https://discuss.aerospike.com/t/primary-index/4912/7)
- How Cassandra handles [deletes with tombstones](https://cassandra.apache.org/doc/latest/cassandra/managing/operating/compaction/tombstones.html)?

### My experience

Though we may not need durable delete in every use case, based on our experience we prefer to keep enabling durable deletes enabled. When we enable it at the default policy level, I have often noticed that, it's been overridden un-intentionally
by sending newly created custom policy which does not extend from default policy. So it is good to educate the developers on how to use policies will help to avoid unexpected behavior.

We faced a scenario where our cluster performance is getting hit after a specific feature deployed in upper environments. It was also overflowing our XDR CDCs in Kafka. Then after some careful analysis that we found [ship-nsup-deletes](https://aerospike.com/docs/database/reference/config#xdr__ship-nsup-deletes) was enabled. By default, it is set to false. So no expired or evicted records send through XDR. But in our case it was set to `true` by mistake. So XDR was tracking all the expired records which are significant in our case.

### All articles in this series

1. [Which storage options to choose?](https://rrameshbtech.github.io/aerospike,/learning,/db/2025/11/09/aerospike-aha-moments.html)
2. [A record which is too big](https://rrameshbtech.github.io/aerospike/learning/2025/11/10/aerospike-aha-moments-2.html)
3. [Is your key too hot?](https://rrameshbtech.github.io/aerospike/learning/2025/11/11/aerospike-aha-moments-3.html)
4. [The predictable Primary Index](https://rrameshbtech.github.io/aerospike/learning/2025/11/12/aerospike-aha-moments-4.html)
5. [Resurrection of the record](#)
