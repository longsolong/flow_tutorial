---
attachments: [Clipboard_2020-01-17-20-23-18.png]
title: TODO
created: '2019-10-13T14:51:19.135Z'
modified: '2020-03-15T04:28:43.696Z'
---

# TODO

重新定位sequence 包括执行/重试
sequence_fast_forward作为默认sequence执行的标准
有点类似dep inject的劲来了。

sequence默认的调度交由自行实现pipeline。




https://tour.golang.org/concurrency/7
https://tour.golang.org/concurrency/10


检查mutex

https://github.com/grafana/grafana/blob/e4c33e0be8418ae5fc99f4c94ee9bf62f9c1e511/pkg/cmd/grafana-server/server.go#L212


日志加入全局trace request_id

runner的逻辑，以及重试的逻辑

执行参数

job-runner/chain/traverser.go:573job-runner/chain/traverser.go:573

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

conditional expansion

// NodeSpec defines the structure expected from the yaml file to define each nodes.
type NodeSpec struct {
	Name         string            `yaml:"name"`      // unique name assigned to this node
	Category     string            `yaml:"category"`  // "job", "sequence", or "conditional"
	NodeType     string            `yaml:"type"`      // the type of job or sequence to create
	Each         []string          `yaml:"each"`      // arguments to repeat over
	Args         []*NodeArg        `yaml:"args"`      // expected arguments
	Sets         []string          `yaml:"sets"`      // expected job args to be set
	Dependencies []string          `yaml:"deps"`      // nodes with out-edges leading to this node
	Retry        uint              `yaml:"retry"`     // the number of times to retry a "job" that fails
	RetryWait    string            `yaml:"retryWait"` // the time, in seconds, to sleep between "job" retries
	If           string            `yaml:"if"`        // the name of the jobArg to check for a conditional value
	Eq           map[string]string `yaml:"eq"`        // conditional values mapping to appropriate sequence names
}

https://github.com/minio/dsync

