---
title: "SwimmerDB"
description: "SwimmerDB"
date: 2020-10-06T08:49:55+00:00
lastmod: 2020-10-06T08:49:55+00:00
draft: false
menu: 
  subprojects:
    parent: "subprojects"
weight: 10
toc: true
images: []
---

## Introduction

SwimmerDB is a classical client/server and embedded database built on top of the SWMRStore (pronounced "swimmer-store"). It can also be used together with [FUSE Integration](/subprojects/fuse-integration) to export structures data as a filesystem for certain applications.

The main purpose of SwimmerDB is to showcase Memoria's capabilities as a full stack data engineering framework, so it's not being targeted as a database for specific types of applications like OLTP or analytics. Instead, SwimmerDB will be good where its underlying storage engine is good.

For example, SWMRStore provides strongly serializable read/write transactions with wait-free data paths for readers and writers, which can be extremely fast -- up to tens of millions TPS out of a single server -- assuming that compute-intensive parts of the data stack are [hardware accelerated/offloaded](/subprojects/smart-storage). It can be provide very good and predictable latency for concurrent writers, providing that all writers are "point-like" updates: like, transferring money from one banking account to another, making only O(1) reads to validate the operation. But queries touching more than O(log N) data blocks, like ... 

```sql
UPDATE Table1 SET F1 = V1 where F2 = V2
```

... may introduce significant (and unpredictable) latency for concurrent writers. Depending on an application, it can be a serious issue. 

There are two types of applications where inherent properties of SWMR CoW make it very good choice:

1. Streaming (near real-time) analytics.
2. Atomic distributed fault-tolerant data structures.

With SWMRStore, readers do not interfere with writers in any way (except for a fixed number of locks on the version history), and readers see a point-in-time snapshot of the data. Because the data is immutable, readers can access it up to the full speed of an underling data store. At the same time, writers are still updating the store, so as soon as readers are done with their current snapshots, they can start processing newly created ones.

Atomic data structures are needed to build complex distributed applications. In most cases of distributed applications there is some subset of the state that requires strong consistency to work with. It can be, for example, entire state of the cluster. Or the state of a reliable message queue. ZooKeeper, [Consul](ttps://www.consul.io) and [Etcd](https://etcd.io) provide distributed atomic primitives to store configuration information. [Atomix](https://atomix.io) provides atomic primitives like counters, maps/multimaps, queues, sets and trees for applications.

SwimmerDB is different in that it provides strongly serializable transactional data structures, not just linearizable/atomic ones. It's possible to accumulate any number of updates in a single transaction and release them to the wild atomically. There is no inherent limit to a transaction size or storage size, and data path may be hardware accelerated.

SwimmerDB is between databases and file systems. It's rather light-weight, low-level, close to hardware and can achieve high IO speeds. But at the same time, provide transactional access to structured data.

## Architecture Objectives

SwimmerDB is intended mostly to accommodate design and development of data containers for Memoria and its applications. Once data container is ready, it can easily be ported into different Memoria-based environments. So, by default, there are three configurations covering most of important use cases:

1. Embedded database.
1. Client/Server mode.
1. Raft- or 2PC-based fault tolerant client/server (2-5 servers).

Immutable data model on top of persistent data structures is capable of running true cloud-native data platforms (with active data caching) but this mode of operation is explicitly a *non-goal* for SwimmerDB.

Nevertheless, SwimmerDB can work in a decentralized environment by:

1. preserving liner update history for entire database or subset of data; 
1. providing arbitrary tree-structured version branches of entire data store;
1. optional block-level MerkelTree-based data integrity protection;


So, it's possible to send/receive patches between instances of SwimmerDB. 

TBC...
