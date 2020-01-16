---
attachments: [Clipboard_2020-01-12-15-34-49.png, Clipboard_2020-01-12-15-41-26.png, Clipboard_2020-01-12-15-42-43.png]
title: build-03-03-准备定义时的抽象step和具象step
created: '2019-10-07T12:44:29.808Z'
modified: '2020-01-12T07:42:43.573Z'
---

# build-03-03-准备定义时的抽象step和具象step

## step的概念对应原python项目里的atom

step/atom即实现一个流程步骤的最小单元，借用spincycle的job的定义来说明。

```
A Job is the smallest, reusable building block in Spin Cycle that has meaning
by itself. A job should do one thing and be reusable. For example, job type
"net/down-ip" removes an IP address from a network interface. This job is
meaningful by itself and reusable in different requests.
```

step是一个包容性的概念，比如理解为步骤，可以参照github workflow的step定义。

```
Steps can run commands, run setup tasks, or run an action in your repository, 
a public repository, or an action published in a Docker registry.
```

## step目录

```
tree pkg/workflow/step/
pkg/workflow/step/
├── builtin
│   ├── command
│   │   ├── command.go
│   │   ├── echo.go
│   │   └── echo_test.go
│   ├── noop.go
│   ├── sleep.go
│   └── sleep_test.go
└── step.go
```

## 抽象step 

![](@attachment/Clipboard_2020-01-12-15-34-49.png)

抽象的Step struct，目前实际上只是帮忙处理了StepID。

## builtin具象step举例

### shell command

![](@attachment/Clipboard_2020-01-12-15-41-26.png)

### echo command

![](@attachment/Clipboard_2020-01-12-15-42-43.png)




