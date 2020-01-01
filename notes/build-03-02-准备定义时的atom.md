---
attachments: [Clipboard_2020-01-12-14-23-08.png, Clipboard_2020-01-12-15-19-59.png]
title: build-03-02-准备定义时的atom
created: '2020-01-12T05:50:19.369Z'
modified: '2020-01-12T07:45:41.389Z'
---

# build-03-02-准备定义时的atom

## atom

atom是单个节点的interface，定义如下。

![](@attachment/Clipboard_2020-01-12-14-23-08.png)

## runable

runable也是interface，定义如下，

![](@attachment/Clipboard_2020-01-12-15-19-59.png)

可以看到，atom和runable加起来，就定义了StepID Create Run Stop几个具象step必须实现的方法。
分开成atom和runable两个interface，也是实现的一种取舍，
- 抽象step甚至只实现atom interface的一部分StepID
- 具象step嵌入抽象step后，再自行实现Create Run以及Stop，即补齐atom interface的剩余部分以及runable这个inferface

### run函数的返回值有所区别

python里有异常捕捉，但是golang里没有，所以run函数的实现的返回值里，得包含此次执行的error的同时，也得包含state。这里参考spincycle引入一个Return struct。
