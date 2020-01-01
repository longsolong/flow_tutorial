---
attachments: [Clipboard_2020-01-12-18-37-52.png]
title: build-04-03-准备单进程运行时的traverser
created: '2019-12-18T00:33:55.109Z'
modified: '2020-01-12T10:39:10.076Z'
---

# build-04-03-准备单进程运行时的traverser

## 概览

traverser是chain的traverser，一对一。traverser负责启动job runner，reaper则作为辅助，traverser通过协调这几者之间的工作，完成按照dag的step之间的依赖对chain中的job进行执行。这几个角色的功能如下。

- 一个request会创建一个chain
- 一个chain的具体执行会由一个traverser来负责协调
- traverser会启动running reaper来将的doneJobChan中的job的下游放入待执行队列runJobChan中。
- traverser会启动job runner来消费runJobChan
- job runner会把执行完的job放入doneJobChan
- 可以看到上面几条就是通过channel来构造了一个良性循环，traverser会负责将流程最开始的几个job放入runJobChan，整个循环就跑起来了。

## 定义

![](@attachment/Clipboard_2020-01-12-18-37-52.png)


```
// Traverser ...
type Traverser struct {
	req   *request.Request // chain是每次请求一个，traverser也是每次请求一个
	chain *Chain // traverser是每个chain启动一个

	reaper chain.JobReaper // 每个traverser启动一个reaper

	runJobChan  chan job.Job  // 用来传递即将执行的job的chan
	doneJobChan chan job.Job  // 用来传递完成执行的job的chan
	doneChan    chan struct{} // 当traverser Stop完成的时候close的chan

	stopMux     *sync.RWMutex // 访问stopped的互斥锁
	stopped     bool          
	stopChan    chan struct{} // stopChan close后，runJobs不能再发起新的goroutine
	pendingChan chan struct{} // runJobs return的时候close的chan
	pending     int64         // runJobs中启动goroutine前会增加1，放入runnerRepo中后会减少1

	runnerRepo runner.Repo // 存储running状态的runner

	logger *infra.Logger

	stopTimeout time.Duration // stopRunningJobs时等待的超时时间
	sendTimeout time.Duration // send job on doneJobChan的超时时间
}
```

## 实现

```
// A Traverser provides the ability to run a job chain while respecting the
// dependencies between the jobs.
type Traverser interface {
	// Run traverses a job chain and runs all of the jobs in it. It starts by
	// running the first job in the chain, and then, if the job completed,
	// successfully, running its adjacent jobs. This process continues until there
	// are no more jobs to run, or until the Stop method is called on the traverser.
	Run(ctx context.Context)

	// Stop makes a traverser stop traversing its job chain. It also sends a stop
	// signal to all of the jobs that a traverser is running.
	//
	// It returns an error if it fails to stop all running jobs.
	Stop(ctx context.Context) error
}
```

### Run

- 先启动runJobs的goroutine。消费runJobChan，job完成的时候发送到doneJobChan, reaper消费doneJobChan。
- 将chain的第一批可执行job放入runJobChan，即dag中没有上游的节点。
- 启动RunningChainReaper的goroutine，消费doneJobChan，并将下游发送给runJobChan。

```
	go func() {
		defer close(runningReaperChan) // indicate reaper is done (see select below)
		defer close(t.runJobChan)      // stop runJobs goroutine
		t.reaper.Run(ctx)
	}()
```

在reaper.Run返回后，会close掉runJobChan，这会接着导致runJobs的goroutine也停下来。
同时还会close掉runningReaperChan，这是reaper done的标志，在紧接着的部分就会监听这个标志。

- <-runningReaperChan select这个chan，如果监听到信号，说明上面的goroutine里的t.reaper.Run(ctx)已经返回。

```
	// Wait for running reaper to be done.
	select {
	case <-runningReaperChan:
		// If running reaper is done because traverser was stopped, we will
		// wait for Stop() to finish. Otherwise, the chain finished normally
		// (completed or failed) and we can return right away.
		t.stopMux.Lock()
		if !t.stopped {
			t.stopMux.Unlock()
			return
		}
		t.stopMux.Unlock()
	}
```

此处还会需要判断stopped标志，如果reaper done是由于traverser本身在进行Stop，则此处Run应该往下走，判断doneChan是否已经close，Stop在最后阶段会对doneChan执行close。如果reaper done是由于chain已经执行完成，完成了使命，这种情况，本Traverser的Run方法可以直接返回。

- 最后就是上面提到的判断doneChan是否已经close的逻辑。

```
	select {
	case <-t.doneChan:
		// Stopped successfully - nothing left to do.
		return
	case <-time.After(20 * time.Second):
		// Failed to stop in a reasonable amount of time.
		// Log the failure and return.
		logger.Warn("stopping the job chain took too long. Exiting...")
		return
	}
```

### runJobs

简单的说就是消费runJobChan并实际发起job runner，等待runner执行完后，将job发送给doneJobChan

- defer close(t.pendingChan)，即runJobs return的时候会close掉pendingChan
- for loop读取runJobChan，注意runJobChan会在上述Run里reaper.Run的时候被close掉。
  - 读取stopChan判断traverser是否已经进入Stop的程序，如果stopChan已经close，则对从runJobChan中读取的job不做任何实际处理，只是将runJobChan消费掉。

  ```
    // Must check before running goroutine because Run() closes runJobChan
		// when the runningReaper is done. Then this loop will end and close
		// pendingChan which stopRunningJobs blocks on. Since this check happens
		// in loop not goroutine, a closed pendingChan means it's been checked
		// for all jobs and either the job did not run or it did with pending+1
		// because the loop won't finish until running all code before the goroutine
		// is launched.
		select {
		case <-t.stopChan:
			fields := []zapcore.Field{
				zap.String("job_id", j.ID().String()),
			}
			logger.Info("not running job %s: traverser stopped", fields...)
			continue
		default:
		}
  ```

  代码本身不难，但是这段注释值得说明下，简单的说就是这个检查放在接下来的"atomic.AddInt64(&t.pending, 1)"以及启动goroutine之前是有原因的，如果在add pending之后，启动goroutine之前的话，就会出现pending增加了，但是goroutine没有启动，导致pending不会相应的减1的情况。导致stopRunningJobs无法结束。另外如果放到启动goroutine之后的话，则没有起到判断的初衷。

  - atomic.AddInt64(&t.pending, 1)
  - 启动执行job runner的goroutine
    - defer一个func在goroutine执行结束的时候无论如何将job发送给doneJobChan，除非sendTimeout，并且从runnerRepo中remove。
    - 读取job当前的totalTries作为参数新建jobRunner
    - runnerRepo.Set该jobRunner并atomic.AddInt64(&t.pending, -1)
    - t.chain.SetJobState设置job state为running并执行jobRunner.Run
    - IncrementJobTries记录增加的job retry到chain的记录上
    - 设置job.State为jobRunner.Run的返回状态

### Stop

- close stopChan并且将stopped置为true
- 执行t.reaper.Stop，其中reaper为上述RunningChainReaper。
- 带着stopTimeout执行stopRunningJobs
- close doneChan

### stopRunningJobs

- 等待pendingChan被close
- 等待pending为0
- 通过WaitGroup等待并发执行所有runnerRepo中的jobRunner的Stop
