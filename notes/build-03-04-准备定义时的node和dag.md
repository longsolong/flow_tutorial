---
attachments: [Clipboard_2020-01-12-15-55-45.png]
title: build-03-04-准备定义时的node和dag
created: '2019-11-16T08:57:33.507Z'
modified: '2020-01-12T07:58:34.693Z'
---

# build-03-04-准备定义时的node和dag

dag和node的部分属性都是用atom interface来表达的，是因为之后dag和node可能包含不同运行时的定义，希望是通用的。

## 定义

![](@attachment/Clipboard_2020-01-12-15-55-45.png)

### node

```
type Node struct {
	Datum   atom.Atom             // Data stored at this Node
	Next    map[atom.AtomID]*Node // out edges ( node id -> Node )
	Prev    map[atom.AtomID]*Node // in edges ( node id -> Node )
	EdgeMux *sync.RWMutex         // for access to vertices maps

	Name          string        // the name of the node
	Retry         uint          // the number of times to retry a node
	RetryWait     time.Duration // the time, in seconds, to sleep between retries
	SequenceID    atom.AtomID   // AtomID for first node in sequence
	SequenceRetry uint          // Number of times to retry a sequence. Only set for first node in sequence.
}
```

node有一个属性sequenceID，用来标识dag中一组连续的node同属一个sequence。
node的Prev Next存储了node之间的上下游关系。

node的Datum是atom，所以只要实现了atom interface的类型都可以存储/包装在node里面从而加入dag。
