---
title: inspire-02-00-square-spincycle
created: '2019-09-21T13:32:18.909Z'
modified: '2019-10-19T03:31:31.827Z'
---

# inspire-02-00-square-spincycle

### DSL

https://square.github.io/spincycle/v1.0/learn-more/basic-concepts#request-specs

Request Specs是用YAML来定义Request。

### 基本概念

https://square.github.io/spincycle/v1.0/learn-more/basic-concepts

层级是Request => sequence => job

- request类似于一个流程
- 一个request包含多个sequence
- 一个sequence包含多个job，能够被多个request服用
- 每个job是一个atomic的unit，能够比sequence被更大程度的复用

这里的层级相比python项目，多了一个sequence的中间层级。

#### sequence-expansion

https://square.github.io/spincycle/v1.0/learn-more/basic-concepts#sequence-expansion

sequence-expansion是一个很好的概念，这里就引出了一个和python项目的不同，expansion在python项目里是在atom上的，而此项目选择是在中间这一层上，是一个有意思的点，sequence这样一来，也就是一个有意思的值得考虑的点了。

> Sequence expansion is not required but it should be used. For example, instead of expanding a sequence, a job could iterate on “hosts”. Spin Cycle doesn’t prevent this, but it should be avoided unless needed. Sequence expansion has many benefits. One benefit is parallelism: expanded sequence run in parallel. Another benefit is isolation: expanded sequences execute independently, which is important for error handling, sequence retry, etc.
Sequence expansion is to Spin Cycle what xargs is to the command line.

这基本也就是python项目里关于expand的思考，这个概念乍看在atom的基础上在dag中引入了一个似乎不一致的的概念，展开的这些节点既要当作多个节点，但又不是完全独立的多个节点，但在批量并行操作的时候用处真的很大。

#### jobs-repo

https://square.github.io/spincycle/v1.0/learn-more/jobs-repo.html

jobs-repo有点类似python项目里集中存放定义时的definitions目录。

- jobs-repo 在外部实现，通过ln symlink进该项目目录然后编译。 job.Factory即对应将atom展开成task的方法，这个概念/命名可以借鉴下。

- definitions目录设计上是可以独立打包，并分发，然后python运行时动态加载。


#### jobs

https://square.github.io/spincycle/v1.0/develop/jobs.html

jobs的代码实现，从interface来讲，分成了Create/Run两个阶段，并且参数分为jobArgs/jobData两种，jobArgs用于创建时即能固定下来的时候采用，jobData更倾向于动态获取的时候采用。

#### sequence-node

https://square.github.io/spincycle/v1.0/develop/requests.html#sequence-node

这里面提到sequence的另外一个重要特性，就是重试的时候是整体重试，这种场景使得sequence是一个更值得考虑新引入的概念。


#### conditional-node

https://square.github.io/spincycle/v1.0/develop/requests.html#conditional-node

conditional-node不是一个强需求的概念，但是如果有这个概念，表达起来会更准确些，比如步骤的状态可以由流程系统置为skipped，而不真实run一个if condition(false)，然后被流程系统置为success。

原python项目里关于此中情况的处理，success/skipped两种都有，success更多一些，可以考虑明确下。


### build & deploy

https://square.github.io/spincycle/v1.0/operate/deploy#building
https://square.github.io/spincycle/v1.0/develop/extensions.html
这两个连接提到了自行实现的job和extension怎么和源码一起编译


