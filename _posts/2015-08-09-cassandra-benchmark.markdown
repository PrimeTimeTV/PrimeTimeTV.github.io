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
========
The EC2 instances used to run Cassandra are m3.xlarge. They have 4 vCPUs and memory 15 GiB. All instances are running in same VPC and private subnet marks. Internal storage is 1x30 GB of SSD. Both Cassandra commit logs and data files were stored on this filesystem. All 4 instances are running in single data center 'ap-southeast'

Software
========
The testing is run on Apache Cassandra 2.0.8 with 4 instances. This is the minimum guarantee of 99% availabilty of our service. 
The Apache Cassandra is deployed on Linux and Oracle JVM 1.7.0_54. The Cassandra is started with 2 seeds and more 2 nodes are joined in 30 seconds


