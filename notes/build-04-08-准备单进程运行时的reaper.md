---
attachments: [Clipboard_2020-01-12-18-42-33.png]
title: build-04-08-准备单进程运行时的reaper
created: '2019-12-18T00:33:24.670Z'
modified: '2020-02-23T14:12:51.087Z'
---

# build-04-08-准备单进程运行时的reaper

## 概览

reaper的主要工作就是来消费doneJobChan，并在满足执行依赖的情况下将下游的job放入runJobChan。

另外本单进程运行时不处理持久化相关的事情，所以相比起spincycle，本运行时只实现RunningChainReaper。
比如，spincycle里另外的StoppedChainReaper用来处理traverser执行Stop后的状态下，对doneJobChan的消费进行不执行但是做好记录的处理。
本运行时暂不作处理，traverser Stop的时候chain的job还没执行完就算了。

## 定义

![](@attachment/Clipboard_2020-01-12-18-42-33.png)

```
type reaper struct {
	chain  *Chain // 绑定的chain
	logger *infra.Logger

	stopMux  *sync.Mutex // 以下字段类似traverser，不过是reaper自身的这部分
	stopped  bool
	stopChan chan struct{}

	doneChan    chan struct{}
	doneJobChan chan job.Job
}
```

## 实现

```
// JobReaper handles jobs and chains that have finished running.
type JobReaper interface {
	// Run reaps done jobs from doneJobChan, saving their states and enqueueing
	// any jobs that should be run to runJobChan. When there are no more jobs to
	// reap, Run finalizes the chain and returns.
	Run(ctx context.Context)

	// Stop stops the JobReaper from reaping any more jobs. It blocks until
	// Run() returns.
	Stop(ctx context.Context)
}
```

### Run

- defer close(r.doneChan)
- for loop
  - 监听doneJobChan对每一个job执行Reap
  - 同时监听stopChan，如果stopChan close则直接返回

### Stop

- 置stopped为true
- close(r.stopChan)，即给Run中的for loop终止信号
- 等待doneChan被close，即等待Run执行完

### Reap

- r.chain.SetJobState(job.ID(), job.State)修改chain的记录中job的状态
- 如果job的状态是执行成功，则将当前job的直接下游job判断IsRunnable为true后发送给runJobChan，如果下游job除了当前job还有其他上游的话，则除非当前job是最后一个完成的上游，否则此时下游job的IsRunnable会为false
- 如果job的状态没有执行成功，则判断CanRetrySequence，即sequence retry。注意此处不处理job retry，job retry在runner中进行。如果能够进行sequence retry，此处调用r.prepareSequenceRetry(job)后，将结果sequence start job重新放入runJobChan

### prepareSequenceRetry

- 获取当前failedJob的sequenceStartJob

- sequenceJobsToRetry := r.sequenceJobsCompleted(sequenceStartJob)

```
	// sequenceJobsToRetry is a list containing the failed job and all previously
	// completed jobs in the sequence. For example, if job C of A -> B -> C -> D
	// fails, then A and B are the previously completed jobs and C is the failed
	// job. So, jobs A, B, and C will be added to sequenceJobsToRetry. D will not be
	// added because it was never run.
```

- 判断sequenceJobsToRetry中是否包含当前failedJob，如果不包含将failedJob加入sequenceJobsToRetry

- 重置所有sequenceJobsToRetry的状态为StateUpForRetry

- 将sequenceStartJob返回后，被Reap重新加入runJobChan
