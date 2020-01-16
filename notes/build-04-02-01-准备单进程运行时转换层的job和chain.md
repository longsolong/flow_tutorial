---
attachments: [Clipboard_2020-01-12-18-12-21.png, Clipboard_2020-01-12-18-14-58.png]
title: build-04-02-01-准备单进程运行时转换层的job和chain
created: '2019-11-16T09:50:37.679Z'
modified: '2020-01-12T10:17:30.927Z'
---

# build-04-02-01-准备单进程运行时转换层的job和chain

## job

定义

![](@attachment/Clipboard_2020-01-12-18-12-21.png)

## chain

定义

![](@attachment/Clipboard_2020-01-12-18-14-58.png)

主要回答当前job的上下游是哪些job，以及job是否running able相关问题，一般情况下，当前步骤只有上游节点都complete后，才能进行。
当然可能有些流程不是这样，可能失败了也能继续往下走。这就可能涉及到定义不同策略的chain，暂不考虑。
