---
author: siri
layout: post
featimg: markdown/sample.png
title: Benchmarking Cassandra in PrimeTime
date:   2015-02-01 00:00:00
tags: cassandra
draft: true
---

Benchmarking Cassandra in PrimeTime
===================================

PrimeTime using Apache Cassandra since the beginning. We use Apache Cassandra as a main storage for storing almost everything in PrimeTime Server. Before we made a decition, we need to run a test to validate our service performance and scalability on Apache Cassandra. 

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

Testing
-------
The testing command is
```
./cassandra-stress -d "{list of node IP}" -e QUORUM -n 1000000 -l 3 -t 100 -o INSERT -r -R NetworkTopologyStrategy

```

