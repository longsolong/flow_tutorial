---
title: inspire-02-01-square-spincycle源码job-runner
created: '2019-10-19T03:30:58.570Z'
modified: '2019-11-09T09:42:28.553Z'
---

# inspire-02-01-square-spincycle源码job-runner


## job-runner/chain/chain.go

chain这个概念基本对应原python项目里dag的部分概念，但又揉进了原python项目里task dep的概念。

所以一个考虑是，将原dag进行拆解，比如其中一部分叫做chain，一部分叫做traverser等稍后会介绍的概念。


### spincycle应该也是支持树形结构的，不止支持一条线

```
// NextJobs finds all of the jobs adjacent to the given job.
func (c *Chain) NextJobs(jobId string) proto.Jobs {
	c.jobsMux.RLock()
	defer c.jobsMux.RUnlock()
	var nextJobs proto.Jobs
	if nextJobIds, ok := c.jobChain.AdjacencyList[jobId]; ok {
		for _, id := range nextJobIds {
			if val, ok := c.jobChain.Jobs[id]; ok {
				nextJobs = append(nextJobs, val)
			}
		}
	}

	return nextJobs
}
```

```
// previousJobs finds all of the immediately previous jobs to a given job.
func (c *Chain) previousJobs(jobId string) proto.Jobs {
	var prevJobs proto.Jobs
	for curJob, nextJobs := range c.jobChain.AdjacencyList {
		if contains(nextJobs, jobId) {
			if val, ok := c.jobChain.Jobs[curJob]; ok {
				prevJobs = append(prevJobs, val)
			}
		}
	}
	return prevJobs
}
```

### RunnableJobs

```
// RunnableJobs returns a list of all jobs that are runnable. A job is runnable
// iff its state is PENDING and all immediately previous jobs are state COMPLETE.
func (c *Chain) RunnableJobs() proto.Jobs {
	var runnableJobs proto.Jobs
	for jobId, job := range c.jobChain.Jobs {
		if !c.IsRunnable(jobId) {
			continue
		}
		runnableJobs = append(runnableJobs, job)
	}
	return runnableJobs
}
```

这个run task dep策略基本和原python项目是一致的，但原python项目没有pending这个状态。暂不清楚为什么有这个必要。

### IsDoneRunning

如果有job失败对整个流程的影响

```
// For chain A -> B -> C, if B is stopped, C is not runnable; the chain is done.
// But add job D off A (A -> D) and although B is stopped, if D is pending then
// the chain is not done. This is a side-effect of not stopping/failing
// the whole chain when a job stops/fails. Instead, the chain continues to run
// independent sequences.
```

这里跟原python项目的处理不太一样，原python项目是一个分支有task失败，就会将整个流程置为失败，这会导致像上面描述中可能还能继续的分支也不继续了。本项目考虑支持同时支持这两种策略，通过可指定的方式。


### SequenceStartJob

```
func (c *Chain) SequenceStartJob(jobId string) proto.Job {
	c.jobsMux.RLock()
	defer c.jobsMux.RUnlock()
	return c.jobChain.Jobs[c.jobChain.Jobs[jobId].SequenceId]
}

func (c *Chain) IsSequenceStartJob(jobId string) bool {
	c.jobsMux.RLock()
	defer c.jobsMux.RUnlock()
	return jobId == c.jobChain.Jobs[jobId].SequenceId
}

func (c *Chain) CanRetrySequence(jobId string) bool {
	sequenceStartJob := c.SequenceStartJob(jobId)
	c.triesMux.RLock()
	defer c.triesMux.RUnlock()
	return c.sequenceTries[sequenceStartJob.Id] <= sequenceStartJob.SequenceRetry
}
```

看起来Sequence有一个convention，即用sequence中最开始的那个job的id作为sequence的id，sequence中所有的job的sequence id都是那个start job的id。SequenceStartJob即那个job。


## job-runner/chain/reaper.go

### JobReaper

```
// A JobReaper handles jobs and chains that have finished running.
//
// The chain's current state (running as normal, stopped, or suspended) influences
// how jobs are handled once they finish running, and how the chain is handled
// once there are no more jobs to run. There are different implementations of
// the JobReaper for each of these cases - a running, stopped, or suspended chain.
type JobReaper interface {
	// Run reaps done jobs from doneJobChan, saving their states and enqueing
	// any jobs that should be run to runJobChan. When there are no more jobs to
	// reap, Run finalizes the chain and returns.
	Run()

	// Stop stops the JobReaper from reaping any more jobs. It blocks until
	// Run() returns and the reaper can be safely switched out for another
	// implementation.
	Stop()
}
```

"The chain's current state (running as normal, stopped, or suspended)"暂时没能理解分成这样几个state并分别实现不同reaper的必要性。

下面看下分别实现的细节，

#### JobReaper

```
// JobReapers:
//  Each JobReaper implementation embeds a "reaper" struct with fields and methods
//  that all implementations use. In general, each reaper loops over doneJobChan,
//  receving jobs as they finish running, until there are no more jobs running.
//  Then the reaper finalizes the chain by sending some information about its state
//  to the Request Manager.
```

#### RunningChainReaper

```
//  The RunningChainReaper is used for normally running chains - in the typical
//  case, this is the only reaper that will be used in the traverser. When a job
//  finished running and is sent to doneJobChan, the RunningChainReaper checks its
//  state and handles retrying the sequence if it failed or starting the next jobs
//  in the chain running if it completed. Once the chain is done (no more jobs are
//  or can be run), the reaper sends the chain's final state to the Request Manager.
```

这段描述，不仅描述了RunningChainReaper的逻辑，还描述了retrying the sequence的逻辑。我似乎觉得retry必须都是sequence的retry。没必要有step自己的retry。只保留一个retry的入口即可。

#### SuspendedChainReaper以及StoppedChainReaper

```
//  The SuspendedChainReaper is switched out for the RunningChainReaper when the
//  traverser receives a signal that this Job Runner instance is shutting down.
//  It waits for all currently running jobs to finish and then determines whether
//  the chain is done (can any more jobs be run?). If the chain is done, the reaper
//  sends its final state to the Request Manager. In the more likely case that the
//  chain is NOT done, the reaper sends a SuspendedJobChain, containing all the info
//  required to later resume the chain, to the Request Manager.
//
//  The StoppedChainReaper is switched out for the RunningChainReaper when a user
//  requests that a currently-running chain be stopped. It waits for all currently
//  running jobs to finish and then sends the chain's final state (most often,
//  failed) to the Request Manager.
// 
```

新项目里可以暂时只考虑StoppedChainReaper。

#### 从RunningChainReaper的Run和Reap实现的comment里，可以窥见Reaper的定位。

```
// Run reaps jobs when they finish running. For each job reaped, if...
// - chain is done: save final state + send to RM.
// - job failed:    retry sequence if possible.
// - job completed: prepared subsequent jobs and enqueue if runnable.
func (r *RunningChainReaper) Run() {

```

```
// reap takes a job that just finished running, saves its final state, and prepares
// to continue running the chain (or recognizes that the chain is done running).
//
// If chain is done: save final state + stop running more jobs.
// If job failed:    retry sequence if possible.
// If job completed: prepared subsequent jobs and enqueue if runnable.
func (r *RunningChainReaper) Reap(job proto.Job) {

```

其中func (r *RunningChainReaper) Reap(job proto.Job)的下面这段实现给我有启发。

```
	default:
		// Job was NOT successful. The job.Runner already did job retries.
		// Retry sequence if possible.
		if !r.chain.CanRetrySequence(job.Id) {
			jLogger.Warn("job failed, no sequence tries left")
			return
		}
		jLogger.Warn("job failed, retrying sequence")
		sequenceStartJob := r.prepareSequenceRetry(job)
		r.runJobChan <- sequenceStartJob // re-enqueue first job in sequence

```

原python项目里worker拿task执行的时候，每次都是独立从数据库轮训检查/争抢。但是其实另外一个思路是，worker完成了流程里的某个步骤后，worker可以继续执行下一个步骤，而不用去争抢。
主要区别来自于task并发执行的实现的问题，
- 原python项目采取的策略是开很多的单线程进程去执行。
- 本项目采用golang实现，有了goroutine的加持，可能会选择不一样的策略，会考虑两种都支持。

## job-runner/chain/validate.go

没特别需要指明的，主要是和job-runner/chain/chain.go有强关联。

## job-runner/chain/traverser.go

```
//  The RunningChainReaper is used for normally running chains - in the typical
//  case, this is the only reaper that will be used in the traverser.
```

```
// runJobs loops on the runJobChan, and runs each job that comes through the
// channel. When the job is done, it sends the job out through the doneJobChan
// which is being consumed by a reaper.
func (t *traverser) runJobs() {
```

这就是reaper、traverser、runner的关系，

- traverser的Run traverses a job chain and runs all of the jobs in it
- traverser的runJobs消费runJobChan，调用runner的Run真正执行job。
- traverser用reaper，来调整job的状态等。

```
// A Traverser provides the ability to run a job chain while respecting the
// dependencies between the jobs.
type Traverser interface {
	// Run traverses a job chain and runs all of the jobs in it. It starts by
	// running the first job in the chain, and then, if the job completed,
	// successfully, running its adjacent jobs. This process continues until there
	// are no more jobs to run, or until the Stop method is called on the traverser.
	Run()

	// Stop makes a traverser stop traversing its job chain. It also sends a stop
	// signal to all of the jobs that a traverser is running.
	//
	// It returns an error if it fails to stop all running jobs.
	Stop() error

	// Running returns all currently running jobs. The status.Manager uses this
	// to report running status.
	Running() []proto.JobStatus
}
```

## job-runner/server/server.go

```
// Runner Factory makes a job.Runner to run one job. It's used by chain.Traversers
// to run jobs.
rf := runner.NewFactory(jobs.Factory, rmc)

// Traverser Factory is used by API to make a new chain.Traverser to run a
// job chain. These are stored in a Traverser Repo (just a map) so API can
// keep track of what's running.
trFactory := chain.NewTraverserFactory(s.chainRepo, rf, rmc, s.shutdownChan)
s.traverserRepo = cmap.New()
```

## 总结

总的来说，spincycle在job-runner/chain/里的功底上还是胜过原python项目，很值得借鉴。
