---
title: Kafka-原理
tags:
  - Kafka
toc: true
date: 2021-04-22 11:26:23
categories:
---

## Customer Group 消费者组

Customer Group 是 Kafka 提供的可扩展且具有容错性的消费者机制

### 重要特征

1. 组内可以有多个消费者实例（Consumer Instance）
2. 消费者组的唯一标识被称为Group ID，组内的消费者共享这个公共的ID
3. 消费者组订阅主题，主题的每个分区只能被组内的一个消费者消费
4. 消费者组机制，同时实现了**消息队列**模型和**发布/订阅**模型   
    >若所有实例都属于一个 Group, 则实现**消息队列**模型  
    >若所有实例分别属于不同 Group, 则实现**发布/订阅**模型

### Customer Group 的 Customer 数量

理想情况下，Consumer 实例的数量应该等于该 Group 订阅主题的分区总数

>假设一个 Group 订阅了 3 个主题，分别是 A、B、C，它们的分区数依次是 1、2、3（总共是 6 个分区）  
>若设置 6 个 Consumer 实例，则平均每个示例消费 6/6=1 个分区，是**理想情形**，最大限度地实现高伸缩性  
>若设置 3 个 Consumer 实例，则平均每个示例消费 6/3=2 个分区  
>若设置 8 个 Consumer 实例，则会浪费 8-6=2 个 Customer 示例，他们不会被分配到任何分区，永远空闲

## Rebalance 重平衡

### 本质
本质上是一种协议，规定了一个 Consumer Group 下的所有 Consumer 如何达成一致，来分配订阅 Topic 的每个分区

### 触发条件

1. 组成员数发生变更  
    >比如有新的 Consumer 实例加入组或者离开组，抑或是有 Consumer 实例崩溃被“踢出”组
2. 订阅主题数发生变更
    >Consumer Group 可以使用正则表达式的方式订阅主题，比如 consumer.subscribe(Pattern.compile("t.*c")) 就表明该 Group 订阅所有以字母 t 开头、字母 c 结尾的主题
    >在 Consumer Group 的运行过程中，你新创建了一个满足这样条件的主题，那么该 Group 就会发生 Rebalance
3. 订阅主题的分区数发生变更  
    >Kafka 当前只能允许增加一个主题的分区数。当分区数增加时，就会触发订阅该主题的所有 Group 开启 Rebalance  

### 分配策略

Rebalance 发生时，Group 下所有的 Consumer 实例都会协调在一起共同参与，根据**分配策略**的协助， 给每个 Consumer 实例分配对应的主题分区  

### 缺点

1. Stop The World
    >在 Rebalance 过程中，所有 Consumer 实例都会停止消费，等待 Rebalance 完成，耗时长
2. 所有 Customer 全部重新分配分区
    >更高效的做法是尽量减少分配方案的变动。例如实例 A 之前负责消费分区 1、2、3，那么 Rebalance 之后，如果可能的话，最好还是让实例 A 继续消费分区 1、2、3，而不是被重新分配其他的分区。这样的话，实例 A 连接这些分区所在 Broker 的 TCP 连接就可以继续用，不用重新创建连接其他 Broker 的 Socket 资源。

## Offset 位移