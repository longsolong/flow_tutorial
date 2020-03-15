---
title: build-03-01-定义时介绍
created: '2019-10-19T02:50:58.051Z'
modified: '2020-02-23T03:00:34.973Z'
---

# build-03-01-定义时介绍

### 总体目录

```
pkg/workflow/
├── atom
│   ├── atom.go
│   └── runnable.go
├── context
│   └── context.go
├── dag
│   ├── dag.go
│   └── dag_test.go
├── errors.go
├── state
│   └── state.go
└── step
    ├── builtin
    │   ├── command
    │   │   └── command.go
    │   ├── noop.go
    │   ├── noop_atom.go
    │   └── noop_test.go
    └── step.go

7 directories, 12 files
```

### 包含关系

| New                                | Origin                |
| ---------------------------------- |:---------------------:|
| DAG struct(map of Node as fields)  | class Dag(dict of subclass of AtomBase as property)      |
| Node struct(embeds Atom interface) |                       |
| user's struct(embeds Step struct)  | subclass of AtomBase  |
| Step struct                        | class AtomBase |
| Atom interface                     |                       |


可以看出来多了两个层级，node以及atom，这主要是由于golang是静态类型语言而目前又没有完整的范型支持，需要node及atom这两个抽象，才能分别在运行时和定义时里作为明确的参数类型传递。上面的"Node struct(embeds Atom interface)"就是一个明显的例子。

