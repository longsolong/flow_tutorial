---
title: inspire-02-03-square-spincycle源码request-manager
created: '2019-10-19T06:08:03.759Z'
modified: '2019-10-19T06:23:46.090Z'
---

# inspire-02-03-square-spincycle源码request-manager

request-manager/grapher/doc.go

```
Package Grapher provides capabilities to construct directed acyclic graphs
from configuration yaml files. Specifically, the graphs that Grapher can
create are structure with the intent to organize workflows, or 'job chains.'

The implementation will read in a yaml file (structure defined below) and
will return a graph struct. Traversing the graph struct can be done via
two methods: a linked list structure and an adjacency list (also described
below).

Basics

* Graph - The graph is defined as a set of Vertices and a set of
directed edges between vertices. The graph has some restrictions on it
in addition to those of a traditional DAG. The graph must:
	* be acyclic.
	* ave only single edges between vertices.
	* contain exacly one source node and one sink node.
	* each node in the graph must be reachable from the source node.
	* the sink node must be reachable from every other node in the graph.
+ The user need not explicitly define a single start/end node in the config
file. Grapher will insert "no-op" nodes at the start/end of sequences to
enforce the single source/sink rule.

* Node - A single vertex in the graph.

* Sequence - A set of Nodes that form a subset of the main Graph. Each
sequence in a graph will also follow the same properties that a graph
has (described above).
```
