---
title: inspire-02-02-square-spincycle源码proto
created: '2019-10-19T06:04:15.201Z'
modified: '2019-10-19T06:05:04.289Z'
---

# inspire-02-02-square-spincycle源码proto

## proto/proto.go

### state vs status

关于task/job的状态用state还是status这个问题，有段源码有点意思。

```
// JobStatus represents the status of one job in a job chain.
type JobStatus struct {
	RequestId string `json:"requestId"`
	JobId     string `json:"jobId"`
	Type      string `json:"type"`
	Name      string `json:"name"`
	StartedAt int64  `json:"startedAt"`        // when job started (UnixNano)
	State     byte   `json:"state"`            // usually proto.STATE_RUNNING
	Status    string `json:"status,omitempty"` // real-time status, if running
	Try       uint   `json:"try"`              // try number, can be >1+retry on sequence retry
}
}
```

这段源码两个都用了，我觉得可能用中文翻译"态"作为state的翻译，比如固态液态，再用"状态"作为status的翻译，可能对两者的区分有一个直观的概念。

