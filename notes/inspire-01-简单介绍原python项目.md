---
title: inspire-01-简单介绍原python项目
created: '2019-09-30T14:01:33.253Z'
modified: '2019-10-19T03:18:08.773Z'
---

# inspire-01-简单介绍原python项目

本文主要介绍下原python项目的基本概念，不做详细展开。

### 流程编排及定义

python项目没有引入DSL，即代码需要承担定义的责任，所以代码分成了两种类型，为了顺口，通常我会称作两种时态，一个是定义时，一个是运行时。

定义时包含dag=>atom这样一个层级，dag即directed acyclic graph，atom即一个个独立的节点。
运行时对应的层级是flow=>task，一个atom一般对应一个task，但也可以expand成多个task。

### 流程执行

涉及到三组件，web api/scheduler/worker

web api提供flows/create接口来接受流程发起的请求，这个请求传入dag的标示，api根据标示找到dag(流程定义)并创建一个flow对象，并持久化到MySQL中。

接下来就是执行flow的两个组件，都是单进程单线程的实现，但可以在数据库性能/机器性能允许的情况下开尽量多的进程协同工作。

scheduler主要分成两部分
- 负责读取MySQL中flow的记录，并根据对应的dag(流程定义)的atom信息，来生成atom对应的task。
- 并负责周期性的汇总检查flow中task的执行情况，并相应地修改flow的状态。

worker则负责读取MySQL中这些task，一次读取一个，并根据task对应的atom定义来实际执行操作。

即这三个组件通过MySQL交换/传递数据，并不直接通信。

### 新项目默认会继承原项目概念

新项目基本会继承原python项目的概念，若有不同会在具体涉及到的地方进行说明。
