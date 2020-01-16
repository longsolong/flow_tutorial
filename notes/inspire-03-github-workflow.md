---
title: inspire-03-github-workflow
created: '2019-09-21T13:33:25.075Z'
modified: '2019-10-19T03:19:02.313Z'
---

# inspire-03-github-workflow


### DSL

https://help.github.com/en/articles/configuring-a-workflow

采用YAML来定义workflow

https://raw.githubusercontent.com/github/orchestrator/master/.github/workflows/main.yml

```
name: CI

on: [pull_request]

jobs:
  build:

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@master

    - name: Set up Go 1.12
      uses: actions/setup-go@v1
      with:
        version: 1.12
      id: go
    - name: Set up SQLite
      run:  sudo apt-get install sqlite3

    - name: Build
      run:  script/cibuild
```

### 基本概念

> Workflows must have at least one job, and jobs contain a set of steps that perform individual tasks. Steps can run commands or use an action. You can create your own actions or use actions shared by the GitHub community and customize them as needed.

层级应该是workflow => job => steps(run command|use action)

- 一个workflow包含多个job
- 一个job包含多个step

这个层级虽然是三层，但是实际上job这一层才对应的通常所说的流程，workflow只是简单的包含多个job(流程)而已，所以我认为只有两层。

> "you will see the build logs, tests results, artifacts, and statuses for each step of your workflow.you will see the build logs, tests results, artifacts, and statuses for each step of your workflow."
