---
title: Kafka-基本概念
tags:
  - Kafka
toc: true
date: 2021-04-21 11:08:07
categories:
---

![kafka](https://raw.githubusercontent.com/altman-xu/altman-xu.github.io/source/ImageForBlog/apache_kafka_logo_icon.svg)

## Producer
生产者

## Consumer
消费者

## Consumer Group
说明: 一个消费者组，由多个consumer组成，消费者组内每个消费者负责消费不同分区的数据，一个分区只能由一个组内的消费者消费，消费者组之间互不影响。所有的消费者都属于某个消费者组，及消费者组是逻辑上的一个订阅者。

一个partition分区中的消息只能被某一个消费者组中的一个消费者消费。这样设计的目的是为了提高消费者组的**并发度**。

当一个消费者组中的消费者个数与主题的分区数一致时才最合理，如果消费者个数过多就造成了性能浪费。

注: kafka 允许一个 topic 被多个消费者组消费，这种情况下。 假如topic有10条消息，则消费组A里面的所有消费者消费10条数据；消费组B里面所有消费者消费10条数据。即一条消息可以被多给消费组重复消费(在同一个消费组内，一条消息只会被组内的一个消费组消费)

## Broker

说明: 一台kafka服务器就是一个broker。一个集群由多个broker组成，一个broker容纳多个topic

## Topic

说明: 可以理解为队列，生产者和消费者都是面向topic

## Replica:副本机制

目的: 保证数据持久化或消息不丢失

同类: MySQL的主从同步(注:mysql从库可以支持读操作，不支持写操作)

说明: 备份即将相同数据拷贝到多台机器上，这些相同数据拷贝在kafka中被称为副本(Replica), kafka定义了以下两类副本: 

### 领导者副本(Leader Replica)
  前者对外提供服务(处理读写消息)

### 追随者副本(Follower Replica)
  不提供任何服务,仅向leader发送请求，同步最新数据，当leader发生故障，某个Follower会成为新的Leader

限制: 同一主题的同一分区的leader和follower一定不在同一台机器上

限制: 创建topic时指定的副本数量，不能超过集群中的broker数量。 (分区数量则没限制，单机也可以创建多分区的topic)

```bash
➜  bin kafka-topics --create --bootstrap-server localhost:9092 --replication-factor 3 --partitions 3 --topic test2
Error while executing topic command : Replication factor: 3 larger than available brokers: 1.
[2021-04-09 11:59:26,516] ERROR org.apache.kafka.common.errors.InvalidReplicationFactorException: Replication factor: 3 larger than available brokers: 1.
 (kafka.admin.TopicCommand$)
➜  bin
```

## Partition:分区机制 

目的: 解决服务伸缩性问题(Scalability)。 提高主题Topic的负载能力。消息会轮询发送到同一Topic主题在不同Broker中的不同Partition分区中

同类: 类似 MongoDb 和 Elasticsearch 中的 Sharding , Hbase 中的 Region

说明: kafka将topic划分成多个partition，每个partition是一组有序的消息日志。生产者往名称为A的topic生产的一条消息，改消息只能被发送到Atopic一个分区

副本是在分区这个层级定义的，每个分区可配置若干个副本，其中只能有一个leader副本，N-1个follower副本。

## 重复消费

整个故事是这样的：假设C1消费P0,P1, C2消费P2,P3。如果C1从未提交，C1挂掉，C2开始消费P0,P1，发现没有对应提交位移，那么按照C2的auto.offset.reset值决定从那里消费，如果是earliest，从P0，P1的最小位移值（可能不是0）开始消费，如果是latest，从P0, P1的最新位移值（分区高水位值）开始消费。但如果C1之前提交了位移，那么C1挂掉之后C2从C1最新一次提交的位移值开始消费。

所谓的重复消费是指，C1消费了一部分数据，还没来得及提交这部分数据的位移就挂了。C2承接过来之后会重新消费这部分数据。

## kafka的3层消息架构

第一层: 主题层(Topic)，每个主题可以配置M个分区，而每个分区可以配置N个副本

第二层: 分区层(Partition)，每个分区的N个副本中只有一个充当Leader角色，对外提供服务；其他N-1个副本是Follower角色，只提供数据冗余

第三层: 消息层(Message)，分区中包含若干条消息，每条消息的位移从0开始，依次递增

## 持久化数据

Kafka 使用消息日志（Log）来保存数据，一个日志就是磁盘上一个只能追加写（Append-only）消息的物理文件。因为只能追加写入，故避免了缓慢的随机 I/O 操作，改为性能较好的顺序 I/O 写操作，这也是实现 Kafka 高吞吐量特性的一个重要手段。

## 数据删除(日志删除)

通过日志段（Log Segment）机制。在 Kafka 底层，一个日志又进一步细分成多个日志段，消息被追加写到当前最新的日志段中，当写满了一个日志段后，Kafka 会自动切分出一个新的日志段，并将老的日志段封存起来。Kafka 在后台还有定时任务会定期地检查老的日志段是否能够被删除，从而实现回收磁盘空间的目的。

## 重平衡：Rebalance

消费者组内某个消费者实例挂掉后，其他消费者实例自动重新分配订阅主题分区的过程。Rebalance 是 Kafka 消费者端实现高可用的重要手段。

## 消息模型

传统的消息队列最少提供两种消息模型，一种P2P，一种PUB/SUB，而Kafka并没有这么做，巧妙的，它提供了一个消费者组的概念，一个消息可以被多个消费者组消费，但是只能被一个消费者组里的一个消费者消费，这样当只有一个消费者组时就等同与P2P模型，当存在多个消费者组时就是PUB/SUB模型

消费者组里面的所有消费者实例不仅“瓜分”订阅主题的数据，而且更酷的是它们还能彼此协助。假设组内某个实例挂掉了，Kafka 能够自动检测到，然后把这个 Failed 实例之前负责的分区转移给其他活着的消费者。这个过程就是 Kafka 中大名鼎鼎的“重平衡”（Rebalance）。嗯，其实既是大名鼎鼎，也是臭名昭著，因为由重平衡引发的消费者问题比比皆是。事实上，目前很多重平衡的 Bug 社区都无力解决。

每个消费者在消费消息的过程中必然需要有个字段记录它当前消费到了分区的哪个位置上，这个字段就是消费者位移（Consumer Offset）。注意，这和上面所说的位移完全不是一个概念。上面的“位移”表征的是分区内的消息位置，它是不变的，即一旦消息被成功写入到一个分区上，它的位移值就是固定的了。而消费者位移则不同，它可能是随时变化的，毕竟它是消费者消费进度的指示器嘛。另外每个消费者有着自己的消费者位移，因此一定要区分这两类位移的区别。我个人把消息在分区中的位移称为分区位移，而把消费者端的位移称为消费者位移。