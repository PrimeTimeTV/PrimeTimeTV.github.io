---
author: siri
layout: post
featimg: cassandra-benchmark/ring-topology.png
title: Benchmarking Cassandra in PrimeTime
date:   2015-08-09 21:45:00
tags: cassandra benchmark aws
draft: false
---

Benchmarking Cassandra in PrimeTime
===================================

PrimeTime using Apache Cassandra since the beginning. We use Apache Cassandra as a main storage for storing almost everything in PrimeTime Server. Before we made a decision, we need to run a test to validate our service performance and scalability on Apache Cassandra. 

To benchmark a Apache Cassandra, we use "cassandra-stress" tool. Since our service is implemented and build on AWS, so we configured the Cassandra to optimize as much as possible with AWS such as Ec2Snitch. The measure are not tested on real production environment. Our production environment is classified and not allowed to expose in the public domain.

Hardware
--------
The EC2 instances used to run Cassandra are m3.xlarge. They have 4 vCPUs and memory 15 GiB. All instances are running in same VPC and private subnet marks. Internal storage is 1x30 GB of SSD. Both Cassandra commit logs and data files were stored on this filesystem. All 4 instances are running in single data center 'ap-southeast'

Software
--------
The testing is run on Apache Cassandra 2.0.8 with 4 instances. This is the minimum guarantee of 99% availabilty of our service. 
The Apache Cassandra is deployed on Linux and Oracle JVM 1.7.0_54. The Cassandra is started with 2 seeds and more 2 nodes are joined in 30 seconds

Cassandra Configuration
-----------------------
Cassandra keyspace is created with a Replication Factor of 3 and single availability zone. If single instance is outage, the Cassandra ring still has a 2 copied of data and continue serve requests. Read and Write consistency, we use QUORUM level so our read/write data always 2 nodes. 

Cassandra Keyspace Configuration

```
Keyspace: Keyspace1:
  Replication Strategy: org.apache.cassandra.locator.NetworkTopologyStrategy
  Durable Writes: true
    Options: [ap-southeast:3]
  Column Families:
    ColumnFamily: Standard1
      Key Validation Class: org.apache.cassandra.db.marshal.BytesType
      Default column value validator: org.apache.cassandra.db.marshal.BytesType
      Cells sorted by: org.apache.cassandra.db.marshal.BytesType
      GC grace seconds: 864000
      Compaction min/max thresholds: 4/32
      Read repair chance: 0.1
      DC Local Read repair chance: 0.0
      Populate IO Cache on flush: false
      Replicate on write: true
      Caching: KEYS_ONLY
      Default time to live: 0
      Bloom Filter FP chance: default
      Index interval: 128
      Speculative Retry: NONE
      Built indexes: []
      Compaction Strategy: org.apache.cassandra.db.compaction.SizeTieredCompactionStrategy
      Compression Options:
        sstable_compression: org.apache.cassandra.io.compress.LZ4Compressor
```

Benmarking
-------
I performed the testing on 1 million rows insert and read records. Each row contains 4 columns. Use random key generator for rowkey.

The testing command is

```
./cassandra-stress -d "{list of node IP}" -e QUORUM -n 1000000 -l 3 -t 100 -o INSERT -r -R NetworkTopologyStrategy
```

Insert operation

```
Averages from the middle 80% of values:
interval_op_rate          : 10057
interval_key_rate         : 10057
latency median            : 1.1
latency 95th percentile   : 4.1
latency 99.9th percentile : 162.0
Total operation time      : 00:01:43
```
The results shown that 1 million records are inserted into Cassandra in 103 seconds. The latency at 95th percentile is only 4.1 milliseconds.

Read operation

```
Averages from the middle 80% of values:
interval_op_rate          : 13201
interval_key_rate         : 13201
latency median            : 1.8
latency 95th percentile   : 7.7
latency 99.9th percentile : 68.1
Total operation time      : 00:01:22
```
The results shown that 1 million records are read from Cassandra in 82 seconds. The latency at 95th percentile is 7.1 milliseconds.

Cost
----
The total cost are calculated from EC2 instance costs. Since there is no inter-region so network traffic costs will not be included.

EC2 4 instances of m3.xlarge On-Demand is 1274.46$ a month. If we sign a 1 year hard-contract, it will be 110.80$ a month.

Assume that we use On-Demand so total cost per minutes is 0.029$ per minute.

AWS cost calculator
[http://calculator.s3.amazonaws.com/index.html](http://calculator.s3.amazonaws.com/index.html)

Final Thoughts
--------------
Assume that one customer play a movie, it will require the following API

* Login
* Get home catalog
* Get content detail
* Play
* DRM verification

Minimum requests for playing a movie of single customer are 5 requests. Let's says 3 requests required insert operations. So it will be 5 read and 3 insert. Our acceptable response time for each requests are 0.2 seconds.


Total concurrent write records in 200 ms are `((1,000,000 * 200) / 103,000) = 1,942` records
Total concurrent read records in 200 ms are `((1,000,000 * 200) / 82,000) = 2,410` records


Total concurrent users (write) are `(1,942/3) = 647` users
Total concurrent users (read) are `(2,410/5) = 482` users


So with 4 instances of Cassandra, we will able to support concurrent 482 users in 200 ms or 144,600 users in 1 minute at cost 0.029$


**Note:** This is only roughly calculate for estimating quick solution for seting up the architecture. There are other constraint such as number of Threads and other optimization that are not writen in this blog.

