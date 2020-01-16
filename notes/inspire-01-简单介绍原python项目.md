---
title: inspire-01-简单介绍原python项目
created: '2019-09-30T14:01:33.253Z'
modified: '2020-01-11T02:48:58.557Z'
---

# inspire-01-简单介绍原python项目

本文主要介绍下原python项目的基本概念，不做详细展开。

### 流程编排及定义

python项目没有引入DSL，即代码需要承担定义流程实现和编排的责任，而非通过yaml或者界面来配置，所以项目代码严格分成了两种类别，为了方便，通常我会称作两种时态，一个是定义时，一个是运行时。一般具体流程实现的开发者接触到的定义时。

定义时包含dag=>atom这样一个层级，dag即directed acyclic graph，atom即一个个独立的节点。
运行时对应的层级是flow=>task，一般一个atom对应一个task，但也可以expand成多个task。

### 流程执行

涉及到三组件，web_api/scheduler/worker

首先，web_api提供flows/create接口来接受发起流程的http请求，这个请求传入dag的标识，api根据标识找到dag(流程定义)并创建一个flow对象，并持久化到MySQL中。

接下来就是执行flow的两个组件，都是单进程单线程的实现，但都可以在MySQL性能/应用机器性能允许的情况下开尽量多的进程，然后在MySQL get_lock的协同下进行工作。
- 这里的并发控制并没有引入更多的第三方依赖，也没有用消息系统来协调，主要还是看场景和性能的需要。
- 这里保持单进程单线程的实现也主要是保持实现的简单。

其中scheduler主要分成两部分
- 负责读取MySQL中flow的记录，并根据对应的dag(流程定义)的atom信息，来生成atom对应的task。
- 并负责周期性的汇总检查flow中task的执行情况，并相应地修改flow的状态。

worker则负责读取MySQL中这些task，一次读取一个，并根据task对应的atom定义来实际执行操作，并相应地修改task的状态。

即这三个组件通过MySQL交换/传递数据，并不直接通信。

