---
title: Redis Introduction
tags:
  - redis
  - tec
date: 2019-12-02
---

## Redis

Redis is a **in-memory, key-value** data store.

### What is in-memory, key-value store

Key-Value store is a storage system where data is stored in form of key and value pairs. When we say in-memory key-value store, by that we mean that the key-value pairs are stored in primary memory(RAM). So we can say that **Redis stored data in RAM in form of key-value pairs.**

In Redis, **key has to be a string but value can be a string, list, set, sorted set or hash.**

### Advantage And Disadvantage of Redis over DBMS

**Database Management systems store everything in second storage which makes read and write operations very slow. But Redis stores everything in primary memory which is very fast in read and write of data.**

Primary memory is limited (much lesser size and expensive than secondary) therefore Redis cannot store large files or binary data. **It can only store those small textual information which needs to be accessed, modified and inserted at a very fast rate.**

### Redis Single Instance Architecture

![single-redis](https://sherlockblaze.com/resources/img/daily/2019-12-02/single-redis.png)

Redis architecture contains two main processes: **Redis client** and **Redis Server**.

- Redis server is responsible for storing data in memory. It handles all kinds of management and forms the major part of architecture.
- Redis client can be Redis console client or any other programming language's Redis API.

### Redis Persistance

There are three different ways to make Redis persistance: **RDB**, **AOF** and **SAVE command**.

**RDB Mechanism**

RDB makes a **copy of all the data in memory and stores them in secondary storagei** (permanent storage).

**This happens in a specified interval. So there is chance that you will loose data that are set after RDB's last snapshot.**

**AOF**

AOF logs all the **write operations** received by the server. Therefore everything is persistance. **The problem with using AOF is that it writes to disk for every operation and it's a expensive task and also size of AOF file is large than RDB file.**

**Save command**

You can force redis server to create a RDB snapshot anytime using the redis console client SAVE command.

### Backup and Recovery Of Redis DataStore

Redis **does not** provide any mechanism for datastore backup and recovery. Therefore if there is any hard disk crash or any other kind of disaster then all data will be lost. You need to use some third party server backup and recovery softwares to workaround it.

If you are using Redis in a replicated environment then there is no need for backup.

### Redis Replication

Replication is a technique involving many computers to enable fault-tolerance and data accessibility. **In a replication environment many computers share the same data with each other so that even if few computers go down, all the data will be available.**

![redis-replication](https://sherlockblaze.com/resources/img/daily/2019-12-02/redis-replication.png)

All the slaves contain exactly same data as master. There can be as many as slaves per master server. When a new slave is inserted to the environment, the master automatically syncs all data to the slave.

- If a slave fails, then also the environment continues working. When a slave again starts working, the master sends updated data to the slave.
- If there is a crash in master server and it looses all data then you should convert a slave to master instead of bringing a new computer as a master.

### Persistance In Redis Replication

One way whole data can be lost is if the whole replicated environment goes down due to power failure.

We can configure master or any one slave to store data in secondary storage using any method(AOF and RDB). Now when the whole environment is again started, make the persistent server as the master server.

**Using persistance and replication together all our data is completely safe and protected from unexpected failures.**

### Clustering In Redis

![redis-cluster](https://sherlockblaze.com/resources/img/daily/2019-12-02/redis-cluster.png)

Clustering is a technique by which data can be shared(divided) into many computers. **The main advantage is that data can be stored in a cluster because its a combination of computers.**

In the above image we can see that data is sharded into four nodes. Each node is a redis server configured as a cluster node.

**If one node fails then the whole cluster stops working.**

### Persistance In Cluster

Data is stored in primary memory of nodes. We need to make the data of each node persistance. We can do that using the previously mentioneds (AOF and RDB). Just configure every node for persistance storage.

### Clustering And Replication Together

![redis-cluster-replication](https://sherlockblaze.com/resources/img/daily/2019-12-02/redis-cluster-replication.png)

Here we convert each node server to a master server. And we keep a slave for every master. So if any node(master) fails, the cluster will start using the slave to keep the cluster operating.

