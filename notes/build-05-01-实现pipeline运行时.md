---
title: build-05-01-实现pipeline运行时
created: '2020-03-15T04:05:46.590Z'
modified: '2020-03-15T04:28:34.963Z'
---

# build-05-01-实现pipeline运行时

此运行时的目标是能够独立完整完成一次流程请求的所有步骤的执行和调度，在一个进程里，并且运行的整个过程由实现者自行编排

## 需要具备的技术特征

- [x] 发起流程时流程级别的参数传递和校验
- [x] 步骤间的协同和参数传递直接通过channel来处理
- [x] 有自行编排goroutine，从而实现类似multi stage pipeline的能力
- [x] 返回流程整体成功或失败

