---
title: build-03-02-准备定义时的step
created: '2019-10-07T12:44:29.808Z'
modified: '2019-10-25T07:37:07.709Z'
---

# build-03-02-准备定义时的step

### step的概念对应原python项目里的atom

step/atom即实现一个流程步骤的最小单元，借用spincycle的job的定义来说明。

```
A Job is the smallest, reusable building block in Spin Cycle that has meaning
by itself. A job should do one thing and be reusable. For example, job type
"net/down-ip" removes an IP address from a network interface. This job is
meaningful by itself and reusable in different requests.
```

用step替换atom这个命名的原因是希望跟github workflow的step的思想保持一致，步骤能够兼容多种可能性，比如可以有一个继承概念ShellCommand专门用作shell command执行，同时又避免atom这样一个一眼不容易明确的概念，atom what? 这一点可以参照github workflow的step定义。

```
Steps can run commands, run setup tasks, or run an action in your repository, 
a public repository, or an action published in a Docker registry.
```

### run函数的返回值有所区别

python里有异常捕捉，但是golang里没有，所以run函数的实现的返回值里，得包含此次执行的error的同时，也得包含state。这里参考spincycle引入一个Return struct。

```
type Return struct {
	State  byte   // proto/STATE_ const
	Exit   int64  // Unix exit code
	Error  error  // Go error
}
```

### 初步实现完成后，引入了三个基本概念

```
type Runnable interface {
	Run(ctx Context) (Return, error)

	Stop() error

	Status() string
}
```

```
type Atom interface {
	Create(ctx workflow.Context) error
}

type Step struct {
	Type string
	Description string
	Atom
}
```

自行实现一个step应该内嵌Step struct，并实现Runnable以及Atom接口，以我实现的ShellCommand以及EchoCommand的例子来讲，

```
type EchoCommand struct {
	ShellCommand
}
```

ShellCommand实现了Runnable，EchoCommand实现了Atom接口，保留了这样一种灵活性。
