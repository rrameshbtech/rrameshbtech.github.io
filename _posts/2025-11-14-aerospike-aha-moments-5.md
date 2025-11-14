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
Yes. You read it correct. It's not just in Aerospike, I knew DBs like Cassandra also has this behavior. Not if any other DB has similar behavior. This happens because how they handle the delete operation of the records. Though your deleted records will not always resurrecting but it can come back in situations like cold restart in Aerospike. So it is good to know how Aerospike handles deletes to mange it effectively based on your use cases.

<!--more-->

By default when you delete a record in Aerospike, it just removes the respective primary index from your memory. It does not really delete your data. So your data will still be there in the write blocks. So during cold restart, when Aerospike tries to rebuild the primary index from data, it recreates the index of the deleted records also. Confusing? Lets see in short what happens when you update, delete & durable delete.

## How it works?

![Aerospike methodology to delete records](/assets/images/aerospike-durable-delete.jpg "Aerospike methodology to delete records. Delete just removes the index. Durable delete adds a tombstone instead")

### When you update a record

Aerospike create a new data record in the write block with new updated bins, latest LUT & generation then it updates your primary index's storage pointer to the newly created record. Still the older version of the record present in the disk. Refer first 2 boxes of above diagram.

### When you delete a record

As we knew, primary index is the ledger of your records, when you delete a record it only deletes the primary index from the memory and does not touch the real data in the disk. Refer box 2 & 3 in above image.

As it just deletes only primary index, when the cold restart runs, it will not have primary index in most cases. So it rebuilds the primary index by iterating through the records stored in write blocks (disk). So it might bring back the deleted records. Check below steps to understand how it happens.

- Read the record in the write block.
- If no primary index available for record just read, add a new primary index (from metadata of the record) and updates storage pointer.
- If already exists, check the lut & generation, if the record just read has latest lut/gen then update the storage pointer to the current record else ignores.

As in the above steps there is no way to identify whether a record has been already deleted or not, it will update the primary index with latest version of the available record.

### When you durably delete a record

As the default delete may not work as expected in some situations and it can bring back the deleted record during situations like cold restart, Aerospike also supports durable deletes. To enable durable delete, set `durableDelete` to `true` in your write policy.

```java
WritePolicy deletePolicy = new WritePolicy();
deletePolicy.durableDelete = true;
// Durably delete a record with the write policy
client.Delete(deletePolicy, key);
```

When a durable delete executed

1. Create a new data in write block with only metadata, latest LUT & Generation. It will not have any bins. It is called tombstone.
2. Update the primary index of the deleted record to the tombstone created in the above step.

As durable delete does not delete the primary index, but it just creates a tombstone and replaces that as the latest version. So during the cold restart the primary index for the record will be created and linked to the tombstone, not to any other version of the record which has bins. So whenever you try to access that record for any operation, as it is linked to tombstone it will not allow any invalid operations like read or write.

### But why my older versions of record still available

Due to various reasons like performance optimization, strong consistency and various other reasons, when you update or delete a record, Aerospike does not really update you existing record in the write blocks(disk), instead it just creates newer version of the record with latest bins & updates your storage pointer. So the older versions of your record still remains until [defragmentation](https://aerospike.com/docs/database/manage/namespace/storage/defrag/) kicks in & does it's job.

You can control when to run defragmentation from your namespace level configuration. Especially [defrag-lwm-pct](https://aerospike.com/docs/database/reference/config#namespace__defrag-lwm-pct) helps to set the percentage of live records in a write block. If the live records percentage touches or goes below this level then defragmentation will kick-in.

### Will the tombstone stay forever?

No. Aerospike runs a special background job named [Tomb raider](https://aerospike.com/docs/database/learn/architecture/durable-deletes#tomb-raider) which will scan the tombstones & relevant primary index then removes the index and tombstone when there is no need for it. Mostly it removes the tombstones in below situations.

1. No older version of the record present. It might have been already removed by defragmentation.
2. [tomb-raider-eligible-age](https://aerospike.com/docs/database/reference/config#namespace__tomb-raider-eligible-age) exceeds
3. Tombstone is successfully shipped by XDR
   and in some more scenarios.

Refer below pages for more information

- [Durable delete](https://aerospike.com/docs/database/learn/architecture/durable-deletes)
- [When can durable deletes come back?](https://support.aerospike.com/s/article/When-can-durable-deletes-come-back)
- [Will my tombstones stay forever?](https://support.aerospike.com/s/article/Durable-delete-tombstones-never-removed-in-system-where-load-is-extremely-low)
- How [cold restart](https://aerospike.com/docs/database/manage/database/cold-start/) works?
- How [Defragmentation](https://aerospike.com/docs/database/manage/namespace/storage/defrag/) works?
- [Do record has metadata to rebuild primary index?](https://discuss.aerospike.com/t/primary-index/4912/7)
- How Cassandra handles [deletes with tombstones](https://cassandra.apache.org/doc/latest/cassandra/managing/operating/compaction/tombstones.html)?

### My experience

Though we may not need durable delete in every use case, based on our experience we prefer to keep enabling durable deletes enabled. When we enable it at the default policy level, I have often noticed that, it's been overridden un-intentionally by sending newly created custom policy which does not extend from the default policy. So it is good to educate the developers on how to use the policies will help to avoid unexpected behavior.

We faced a scenario where our cluster performance is getting impacted after a specific feature deployed in upper environments. It was also overflowing our XDR CDCs in Kafka. Then after some careful analysis, we found that the [ship-nsup-deletes](https://aerospike.com/docs/database/reference/config#xdr__ship-nsup-deletes) parameter was enabled unexpectedly. By default, it is set to false. So no expired or evicted records send through XDR. But in our case it was set to `true` by mistake. So XDR was tracking all the expired records which are significant in our case.

If you still prefer that I do not want my XDR to send deleted records, you can try [XDR Filter](https://aerospike.com/docs/database/manage/xdr/filters/#setting-the-filter) which will help to you avoid unnecessary records being flowed through XDR CDC.

### All articles in this series

1. [Which storage options to choose?](https://rrameshbtech.github.io/aerospike,/learning,/db/2025/11/09/aerospike-aha-moments.html)
2. [A record which is too big](https://rrameshbtech.github.io/aerospike/learning/2025/11/10/aerospike-aha-moments-2.html)
3. [Is your key too hot?](https://rrameshbtech.github.io/aerospike/learning/2025/11/11/aerospike-aha-moments-3.html)
4. [The predictable Primary Index](https://rrameshbtech.github.io/aerospike/learning/2025/11/12/aerospike-aha-moments-4.html)
5. [Resurrection of the record](#)
