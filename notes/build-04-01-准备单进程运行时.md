---
title: build-04-01-准备单进程运行时
created: '2019-11-10T06:19:16.294Z'
modified: '2020-02-22T08:49:19.108Z'
---

# build-04-01-准备单进程运行时

此运行时的目标是能够独立完整完成一次流程请求的所有步骤的执行和调度，在一个进程里。

## 需要具备的技术特征

- [x] 发起流程时流程级别的参数传递和校验
- [x] 步骤级别的参数传递和校验
- [ ] 有步骤间参数的上下文参数传递
- [x] 有步骤重试以及sequence重试
- [ ] 有sequence expansion并且有并发执行expansion的能力
- [x] dag有分支并且有并发执行分支的能力
- [ ] 有conditional导致ignored的情况
- [ ] 有动态生成步骤
- [ ] 有动态填充步骤
- [x] 发起执行流程的接口
- [ ] 中断执行的能力
- [ ] 有自行通过channel编排goroutine pipeline的能力

### 项目需要达成的小目标

- [x] ping是静态的dag，也是静态填充chain，演示重试次数。
- [ ] wc演示sequence expansion，即静态的dag，动态填充chain。
- [ ] fibonacci是可以动态dag，但是静态填充chain。
- [ ] guess_game是动态dag，也是动态填充chain。
- [ ] wc演示并发以及分支汇总。

### 其他

一点奇思妙想，其实完成了此运行时后，此项目甚至可以作为一个api framework:)
- 专门用来处理较为笨重/复杂的调用链路
- 并且像组装流程一样组装

