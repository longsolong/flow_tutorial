---
title: build-03-05-准备定义时的sequence
created: '2019-10-07T12:45:30.317Z'
modified: '2020-01-16T08:08:18.702Z'
---

# build-03-05-准备定义时的sequence

sequence这个概念没有自己的struct，sequence只是node的一个字段SequenceID。

sequence即是将多个step连成一组，并且约定指定第一个的StepID作为SequenceID。

sequence的概念希望达成的是两个目的，
1, 重试的时候，重试的单位可以为step，可以为sequence。
2, expand的时候，只能以sequence单位去expand。

步骤StepA->StepB->StepC这样一个sequence定义，
1，如果在StepB失败了，会让StepA/StepB都重回retry状态，但此时StepC还没执行，所以不用处理。
2，如果需要在比如叫做inputs=[1,2,3,4]这样的参数上进行展开。就会出现

```
               expansion1 ->
             /              \
pre step... -> ...           +-> post step...
             \              /
               expansion4 ->
```


这样一个四列展开的执行序列，每列的执行/重试是独立的，四列的执行是并发执行的。
后续其他step的执行等待条件是这四个序列同时处于完成状态。

