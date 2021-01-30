---
title: Github Actions 初体验
date: 2021-01-29 20:59:30
tags:
  - hexo
  - Github
---

Github Actions 是 Github 提供给广大开发者的持续集成服务，于 2018 年 10 月推出。
本文将介绍我是如何使用 Github Actions 来为本博客提供自动部署服务的。
这里只讲部署，不会涉及 Hexo 的搭建相关工作

### Hexo 的部署步骤

Hexo 提供了快速方便的一键部署功能，让您只需一条命令就能将网站部署到服务器上。

1. 首先在 _config.yml 文件中设置相关信息

   ```Yaml
   # Deployment
   ## Docs: https://hexo.io/docs/one-command-deployment
   # 可以填写多个仓库
   deploy:
     type: git
     repository: git@github.com:username/username.github.io.git # 仓库地址
     branch: master # 分支
   ```

2. 随后在我们写完文章之后执行以下语句

   ```bash
   $ hexo clean
   $ hexo deploy
   ```

至此我们的博客部署完成，但是每次都要人工操作是否有些麻烦呢？我们只想简单的写个文章，这么多繁琐的部署流程肯定是不想做的，所以接下来有请我们的 Github Actions 登场。

### Github Actions 介绍

首先 Github Actions 有一些规则：

1. **workflow** : 工作流程，持续集成一次运行的过程就是一个 workflow
2. **job** : 任务，一个 workflow 由一个或多个 jobs 构成，意思是一次持续集成的运行，可以完成多个任务
3. **step** : 步骤，每个 job 由多个 step 构成，一步一步的完成
4. **action** : 动作，每个 step 可以依次执行一个或多个命令（action）

#### workflow 文件

GitHub Actions 的配置文件叫做 workflow 文件，存放在代码仓库的 .github/workflows 目录中。

workflow 文件采用 Yaml 格式，文件名可以任意取。一个库可以有多个 workflow 文件，Github只要发现 .github/workflows 目录中存在 .yaml 或者 .yml 的文件，就会自动运行该文件。

workflow 文件的配置字段非常多，详见[官方文档](https://help.github.com/en/articles/workflow-syntax-for-github-actions)。下面是一些基本字段。

- `name`

  `name` 字段是 workflow 的名称。如果省略该字段，默认为当前 workflow 的文件名。

  ```yaml
  name: Hexo deploy on Github Actions
  ```
- `on`

  `on` 字段指定触发 workflow 的条件，通常是某些事件。

  ```yaml
  on: push
  # 当 push 触发时执行 workflow
  ```

- 多个事件触发

  ```yaml
on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main
  # 以上代码表示在 push 事件在分支 main 上触发 或者 pull_request 事件在分支 main 上触发时，执行 workflow
  ```
- `jobs`

  workflow 文件的主体是 `jobs` 字段，表示要执行的一项或多项任务。

  - `jobs.<job_id>`

    每项任务必须关联一个 ID。 键值 `job_id` 是一个字符串，其值是作业配置数据的映像。 `<job_id>` 必须以字母或 `_` 开头，并且只能包含字母数字字符、`-` 或 `_`。

    ```yaml
    jobs:
      my_first_job:
        name: My first job
      my_second_job:
        name: My second job
    ```

  - `jobs.<job_id>.name`

    任务显示在 GitHub 上的名称

  - `jobs.<job_id>.needs`

    进行此项任务之前，需要完成的前置任务。若某个任务失败，则之后需要它的任务都会被跳过，除非这些任务使用可以让任务进行的条件表达式

    ```yaml
    jobs:
      job1:
      job2:
        needs: job1
      job3:
        needs: [job1, job2]
    ```

    在此示例中，`job1` 必须在 `job2` 开始之前成功完成，而 `job3` 要等待 `job1` 和 `job2` 完成。

    **不要求相关任务完成的实例：**

    ```yaml
    jobs:
      job1:
      job2:
        needs: job1
      job3:
        if: always()
        needs: [job1, job2]
    ```

    `job3` 使用 `always()` 条件表达式，因此它始终在 `job1` 和 `job2` 完成后运行，不管它们是否成功。

  - `jobs.<job_id>.run-on`

    运行任务所需容器（**必需**）

    | 虚拟环境             | YAML 工作流程标签                  |
    | :------------------- | :--------------------------------- |
    | Windows Server 2019  | `windows-latest` 或 `windows-2019` |
    | Ubuntu 20.04         | `ubuntu-20.04`                     |
    | Ubuntu 18.04         | `ubuntu-latest` 或 `ubuntu-18.04`  |
    | Ubuntu 16.04         | `ubuntu-16.04`                     |
    | macOS Big Sur 11.0   | `macos-11.0`                       |
    | macOS Catalina 10.15 | `macos-latest` 或 `macos-10.15`    |

    示例

    ```yaml
    runs-on: ubuntu-latest
    ```

  - `jobs.<job_id>.steps`

    `steps`字段指定每个 Job 的运行步骤，可以包含一个或多个步骤。每个步骤都可以指定以下三个字段。

    - `jobs.<job_id>.steps.name`：步骤名称。
    - `jobs.<job_id>.steps.run`：该步骤运行的命令或者 action。
    - `jobs.<job_id>.steps.env`：该步骤所需的环境变量。

下面是一个完整的 workflow 文件范例：

```yaml
name: Workflow Demo
on: 
  push:
    branches:
      - main

jobs:
  my-job:
    name: My Job
    runs-on: ubuntu-latest
    steps:
    - name: Print text
      env:
        MY_NAME: Mona
        MY_AGE: 18
      run: |
        echo "My name is" $MY_NAME "and age is" $MY_AGE.
```

### 利用 Github Actions 持续部署我们的博客

我这里使用两个仓库来演示

#### 创建对应的仓库

首先，我们需要两个仓库。一个用于存放我们博客的源码、另一个用于存放我们的博客编译后的数据

![2021-01-29-01.png](/images/2021-01-29-01.png)

![2021-01-29-02.png](/images/2021-01-29-02.png)

git 相关的操作我们就略过哈！！！

#### 编写 workflow 操作源码库

具体代码如下：

```yaml
name: Hexo Deploy

on:
  push:
    branches:
      - master

jobs:
  build:
    runs-on: ubuntu-latest
    if: github.event.repository.owner.id == github.event.sender.id

    steps:
      - name: Checkout source
        uses: actions/checkout@v2
        with:
          ref: master

      - name: Setup Node.js
        uses: actions/setup-node@v1
        with:
          node-version: '14.x'

      - name: Setup Hexo
        env:
          ACTION_DEPLOY_KEY: ${{ secrets.ACCESS_TOKEN }}
        run: |
          mkdir -p ~/.ssh/
          echo "$ACTION_DEPLOY_KEY" > ~/.ssh/id_rsa
          chmod 700 ~/.ssh
          chmod 600 ~/.ssh/id_rsa
          ssh-keyscan github.com >> ~/.ssh/known_hosts
          git config --global user.email "zhangjieepic@gmail.com"
          git config --global user.name "zhangjie-me"
          npm install hexo-cli -g
          npm install
      - name: Deploy
        run: |
          hexo clean
          hexo deploy
```

至此，每当我们将写好的文章提交到 GIthub 的源码库时，Github Actions 会自动执行相关的部署的操作，将部署好的网页源码同步到我们的 Github Pages 仓库。

@o@