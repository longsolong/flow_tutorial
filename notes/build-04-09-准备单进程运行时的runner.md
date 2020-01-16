---
attachments: [Clipboard_2020-01-12-18-43-54.png]
title: build-04-09-准备单进程运行时的runner
created: '2020-01-01T09:32:12.453Z'
modified: '2020-01-12T10:43:55.791Z'
---

# build-04-09-准备单进程运行时的runner

## 概览

runner是执行单个job的单元/组件

## 定义

![](@attachment/Clipboard_2020-01-12-18-43-54.png)

## 实现

```
// A Runner runs and manages one job in a job chain. The job must implement the
// job.Job interface.
type Runner interface {
	// Run runs the job, blocking until it has completed or when Stop is called.
	// If the job fails, Run will retry it as many times as the job is configured
	// to be retried. When the job successfully completes, or reaches the maximum number
	// of retry attempts, Run returns the final state of the job.
	Run(ctx context.Context) Return

	// Stop stops the job if it's running. The job is responsible for stopping
	// quickly because Stop blocks while waiting for the job to stop.
	Stop(ctx context.Context) error
}
```

### Run

- 按重试次数for循环
  - 如果runner stopped了，则break
  - 调用step定义的Run
  - 如果执行成功或者stopped，则break
  - 如果重试达到最大次数也break
  - sleep一个retryWait的时间

### Stop

- close(r.stopChan)
- 调用step定义的Stop

