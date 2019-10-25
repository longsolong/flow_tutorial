---
title: build-02-开始项目
created: '2019-09-21T12:05:56.524Z'
modified: '2019-10-24T08:29:09.065Z'
---

# build-02-开始项目

### 项目基本结构

初始化一个项目的时候，会首先考虑项目的目录结构，这个项目本身的定位是一个library，实现一个基本的workflow engine所需要具备的逻辑，可能出于演示/示范这个library用法的目的，会包装一个api或者cli。

花了一些时间看了下，最终考虑采用https://github.com/marvincaspar/go-web-app-boilerplate这个模版。

这个模板有几个依赖，

- github.com/gorilla/mux 为web framework
- https://github.com/urfave/cli 为cli library
- github.com/sirupsen/logrus 为日志库

当前的目录结构

```
.
├── LICENSE
├── Makefile
├── README.md
├── api
├── cmd
│   └── server
│       └── main.go
├── configs
├── githooks
│   └── pre-commit
├── go.mod
├── go.sum
├── pkg
│   ├── README.md
│   ├── http
│   │   └── rest
│   │       ├── handler.go
│   │       ├── handler_test.go
│   │       ├── healthcheck.go
│   │       ├── healthcheck_test.go
│   │       └── middleware
│   │           ├── jsonresponse.go
│   │           ├── jsonresponse_test.go
│   │           ├── logging.go
│   │           ├── logging_test.go
│   │           ├── middleware.go
│   │           └── middleware_test.go
│   ├── infra
│   │   ├── logger.go
│   │   └── server.go
│   ├── models
│   ├── registry
│   ├── services
│   ├── setting
│   └── storage
├── scripts
│   └── coverage.sh
├── test
│   ├── logger.go
│   └── rest.go
└── web

18 directories, 23 files
```

其中pkg目录下，比如registry、services、storage、models、infra等我一开始都有疑问。

其中大部分概念都可以参考

https://github.com/marvincaspar/go-web-app-boilerplate/blob/master/pkg/README.md

其中repository的概念参考
https://github.com/grafana/grafana/blob/b858a5f4969981323a9569050331a77d9a46fba7/pkg/services/annotations/annotations.go#L51

pkg这个目录的具体实现可以参考 https://github.com/grafana/grafana/blob/master/pkg/ARCHITECTURE.md 我疑惑了很久才翻到这样一个对该标准进行阐释的样例。
注意"When Grafana was born there didnt exist much guides or direction for how to write medium sized application. So there are parts of Grafana code base that didn't quite pan out as we wanted. More about that under current rewrites! :)" 这句话，即grafana这个实现里，跟我们用的模板命名上可能会有不一样的地方，放置的目录可能也有不一样的地方，但这都是仁者见仁的事情，重要的是该文档对于理解有很大的帮助。


另外对于本项目考虑增加两个目录
- workflows
- pkg/orchestration

目录结构这个问题可以多参考下
- https://github.com/golang-standards/project-layout
- https://github.com/katzien/go-structure-examples 这个作者还在readme里列了她的演讲内容，可以看下


### 项目需要达成的几个小目标

- 跑通example
- https://github.com/github/orchestrator/blob/master/.github/workflows/main.yml 模拟这个orchestrator的build过程
- 出于一种执念，至少第一版还是会保持只用golang代码来表达而不是引入某种DSL。

### 一些额外依赖项

- https://github.com/caarlos0/env
- https://github.com/doug-martin/goqu
- https://github.com/facebookincubator/ent
- https://github.com/jmoiron/sqlx
- https://github.com/uber-go/dig


### 准备期间的其他阅读

https://www.alexedwards.net/blog/organising-database-access
