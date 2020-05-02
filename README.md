<!--
#
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
-->
# Fork改动
1. 修改一部分dockerfile，解决在国内构建openwhisk中的奇怪问题
  * https://hub.docker.com/repository/docker/heersin/openjdk-alpine-china
  * https://hub.docker.com/repository/docker/heersin/ow-utils-china
2. 预下载一部分文件，方便没有网络条件的情况下手动添加
3. 加入搭建时用的两个脚本。

## 背景
主要是搭建过程中时常遇到一些网络问题(<a href="">issue#4888</a>)，比如alpine/ubuntu的源问题
```sh
$ apk add --no-cache bash
fetch http://dl-cdn.alpinelinux.org/alpine/v3.7/main/x86_64/APKINDEX.tar.gz
fetch http://dl-cdn.alpinelinux.org/alpine/v3.7/community/x86_64/APKINDEX.tar.gz
(1/6) Installing pkgconf (1.3.10-r0)
(2/6) Installing ncurses-terminfo-base (6.0_p20171125-r0)
(3/6) Installing ncurses-terminfo (6.0_p20171125-r0)
ERROR: ncurses-terminfo-6.0_p20171125-r0: Protocol error
(4/6) Installing ncurses-libs (6.0_p20171125-r0)
(5/6) Installing readline (7.0.003-r0)
(6/6) Installing bash (4.4.19-r1)
ERROR: bash-4.4.19-r1: Protocol error
Executing busybox-1.27.2-r7.trigger
2 errors; 5 MiB in 15 packages
```
以及构建ow-utils时需要从google上下载一个东西，对于构建大型项目来说耗时太长了。所以对原始构建过程中的一些配置进行改动，以及构建了2个docker image解决网络问题。


## 项目文件
- Docker File改动
  * common/scala/Dockerfile, 原始dockerfile备份为.bak
  * tools/ow-utils/Dockerfile, 原始dockerfile备份为.bak

- pre_download_and_script
  * APKINDEX.tar.gz -- alpine 所需
  * OpenWhisk_CLI-latest-linux-amd64.tgz -- owutils 所需
  * kubectl -- owutils 所需
  * start_wsk.sh -- 启动ansible的命令
  * wsk_env.sh -- wsk_env 相关环境变量(手动在shell中输入而非执行脚本)

## Quick Start
构建过程参考原始项目即可([Deploy to Docker for Ubuntu](./tools/ubuntu-setup/README.md))，这里仅仅是个人搭建时的一些经验
1. ubuntu需要安装npm，在构建一个组件的时候需要，然而openwhisk的readme上似乎并没有提醒。
2. 构建过程中最好开启debug模式即"./gradlew distDocker --debug",方便出问题时排查
3. 需要更换如pip源、apt源等可以阅读对应Dockerfile，找到启动的镜像，若镜像内有sed，则可以用sed在dockerfile内写命令对docker的源配置进行修改。没有的话可能需要启动镜像，修改后commit，然后在dockerfile里引用自己修改后的镜像。
4. 可能用到的命令
```
docker run -it [IMAGE] "/bin/bash" 启动docker内的shell(也可能没有bash，换成sh)
docker cp [file_path] [docker_name]:/path/to/ 从外部拷贝文件到docker内
docker ps -a 列出运行中的docker
```

# ==========原始项目简介==========
# OpenWhisk

[![Build Status](https://travis-ci.org/apache/openwhisk.svg?branch=master)](https://travis-ci.org/apache/openwhisk)
[![License](https://img.shields.io/badge/license-Apache--2.0-blue.svg)](http://www.apache.org/licenses/LICENSE-2.0)
[![Join Slack](https://img.shields.io/badge/join-slack-9B69A0.svg)](http://slack.openwhisk.org/)
[![codecov](https://codecov.io/gh/apache/openwhisk/branch/master/graph/badge.svg)](https://codecov.io/gh/apache/openwhisk)
[![Twitter](https://img.shields.io/twitter/follow/openwhisk.svg?style=social&logo=twitter)](https://twitter.com/intent/follow?screen_name=openwhisk)

OpenWhisk is a serverless functions platform for building cloud applications.
OpenWhisk offers a rich programming model for creating serverless APIs from functions,
composing functions into serverless workflows, and connecting events to functions using rules and triggers.
Learn more at [http://openwhisk.apache.org](http://openwhisk.apache.org).

* [Quick Start](#quick-start) (Deploy and Use OpenWhisk on your machine)
* [Deploy to Kubernetes](#deploy-to-kubernetes) (For development and production)
* For project contributors and Docker deployments:
  * [Deploy to Docker for Mac](./tools/macos/README.md)
  * [Deploy to Docker for Ubuntu](./tools/ubuntu-setup/README.md)
* [Learn Concepts and Commands](#learn-concepts-and-commands)
* [OpenWhisk Community and Support](#openwhisk-community-and-support)
* [Project Repository Structure](#project-repository-structure)

### Quick Start

The easiest way to start using OpenWhisk is to install the "Standalone" OpenWhisk stack.
This is a full-featured OpenWhisk stack running as a Java process for convenience.
Serverless functions run within Docker containers. You will need [Docker](https://docs.docker.com/install),
[Java](https://java.com/en/download/help/download_options.xml) and [Node.js](https://nodejs.org) available on your machine.

To get started:
```
git clone https://github.com/apache/openwhisk.git
cd openwhisk
./gradlew core:standalone:bootRun
```

- When the OpenWhisk stack is up, it will open your browser to a functions [Playground](./docs/images/playground-ui.png),
typically served from http://localhost:3232. The Playground allows you create and run functions directly from your browser.

- To make use of all OpenWhisk features, you will need the OpenWhisk command line tool called
`wsk` which you can download from https://s.apache.org/openwhisk-cli-download.
Please refer to the [CLI configuration](./docs/cli.md) for additional details. Typically you
configure the CLI for Standalone OpenWhisk as follows:
```
wsk property set \
  --apihost 'http://localhost:3233' \
  --auth '23bc46b1-71f6-4ed5-8c54-816aa4f8c502:123zO3xZCLrMN6v2BKK1dXYFpXlPkccOFqm12CdAsMgRU4VrNZ9lyGVCGuMDGIwP'
```

- Standalone OpenWhisk can be configured to deploy additional capabilities when that is desirable.
Additional resources are available [here](./core/standalone/README.md).

### Deploy to Kubernetes

OpenWhisk can also be installed on a Kubernetes cluster. You can use
a managed Kubernetes cluster provisioned from a public cloud provider
(e.g., AKS, EKS, IKS, GKE), or a cluster you manage yourself.
Additionally for local development, OpenWhisk is compatible with Minikube,
and Kubernetes for Mac using the support built into Docker 18.06 (or higher).

To get started:

```
git clone https://github.com/apache/openwhisk-deploy-kube.git
```

Then follow the instructions in the [OpenWhisk on Kubernetes README.md](https://github.com/apache/openwhisk-deploy-kube/blob/master/README.md).

### Learn Concepts and Commands

Browse the [documentation](docs/) to learn more. Here are some topics you may be
interested in:

- [System overview](docs/about.md)
- [Getting Started](docs/README.md)
- [Create and invoke actions](docs/actions.md)
- [Create triggers and rules](docs/triggers_rules.md)
- [Use and create packages](docs/packages.md)
- [Browse and use the catalog](docs/catalog.md)
- [OpenWhisk system details](docs/reference.md)
- [Implementing feeds](docs/feeds.md)
- [Developing a runtime for a new language](docs/actions-actionloop.md)

### OpenWhisk Community and Support

Report bugs, ask questions and request features [here on GitHub](../../issues).

You can also join the OpenWhisk Team on Slack [https://openwhisk-team.slack.com](https://openwhisk-team.slack.com) and chat with developers. To get access to our public Slack team, request an invite [https://openwhisk.apache.org/slack.html](https://openwhisk.apache.org/slack.html).

### Project Repository Structure

The OpenWhisk system is built from a [number of components](docs/dev/modules.md).  The picture below groups the components by their GitHub repos. Please open issues for a component against the appropriate repo (if in doubt just open against the main openwhisk repo).

![component/repo mapping](docs/images/components_to_repos.png)
