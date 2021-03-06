---
title: BAETYL
tags:
  - Architecture
date: 2019-04-12
---

## What

这次主要跟大家分享一下 baetyl 架构和核心功能实现，随着时代的发展，边缘计算受到的关注也越来越多，最近通信行业的快速发展，也推动了边缘计算的发育。baetyl 作为百度第一款开源边缘计算框架，有很多值得分享的地方。个人功力还比较浅，覆盖面可能不够全面，欢迎大家提问或纠正。

## Why

### 为什么要做边缘计算？

1. 先从网络的角度考虑，随着时代的发展，我们需要移动网络有能力发现离用户比较近的边缘节点实现流量就近出，并且可以满足计费、安全监管等网络运维要求。
2. 再从云来考虑。有部分企业级用户不太能接受数据完全上云的方案。这就需要边缘计算资源具备弹性、可调度性，能够配合业务逻辑实现资源的合理分配，实现计算能力到边缘的延伸。
3. 发展通用的平台计算的能力。

### 边缘和云端之间的关系

边缘计算可以理解成云计算的一个扩展，云端可以提供一些 Web Services，提供存储，模型训练等功能，但是有一些工作，在云端是很难触及到的，比如视频流的采集，以及一些IoT业务范围的事情。所以边缘计算对于云端来说起到了一个能力上的补充。另外，由于一些数据敏感性、处理时延的问题，部分工作放在端上处理显的更加合适。

从上面的角度来考虑，边缘计算对云市场来说是很大的一个机会。

1. 流量问题，我们不可能，或者说不被允许，将大量的数据传输到云端，一个是从安全性角度考虑，一个是实际流量上的问题，如果全部上传，所有的计算都发生在云端，显然需要巨大的流量。
2. 时延，就像刚刚说的，如果所有的计算发生在云端，我们需要经历数据上传、云端服务进行计算、响应、响应处理这几个大过程，时效性上显然存在问题，特别是对于一些需要低时延的功能或操作。

同时，我们也涉及到了一个问题，也就是动态调度，因为在边缘设备大多性能偏差、远远不如云端服务器强大，所以我们需要对资源进行动态的管理。这里主要涉及到两个方面：

1. 云计算中心和边缘设备之间的调度
2. 边缘设备之间的调度

## How

目前我们的边缘计算分为了两部分一个是我们的通用框架，也就是今天要讲的主题 baetyl，另一个就是 BIE。

我们主要分享一下 baetyl Framework。

### 我们想要做成什么样？

1. 独立于硬件，尽量放大终端设备的运行能力，为此我们采用了 Pure Golang 来编写程序，借用到了 Golang 的跨平台能力，并且在程序中全部使用静态链接。当然，现在我们也在探索 Rust 的可能性。
2. 支持各种类型的应用程序，从较小的层面上来讲，我们提供了一个 Function Manager，用于启动不同的函数计算，来进行函数实例管理和消息出发的函数调用。从一个更大的层面来说，我们所有的除主程序之外的内容，都可以称为是一个应用程序，包括一些我们官方提供的一些模块，比如 `baetyl-agent`，`baetyl-hub` 等，都可以算作一个应用程序，一个模块，都可以进行替换或者和其他模块进行组合完成功能上的扩展。
3. 支持在线管理和离线运行，我们提供了一个 agent 模块，用于和云端进行交互，目前我们支持硬件状态的上报，和服务OTA，现在的情况是，进行服务OTA时，会关闭原先老的服务，再按照配置重新启动服务。目标情况是保持未发生变更服务的运行。

### baetyl 框架期待解决的问题

那我们需要解决的问题有哪些呢？

1. 对硬件做抽象，上面提到，我们通过使用纯粹的 Golang 来编写框架，利用 Golang 本身强大的跨平台能力，尽可能的去屏蔽底层硬件的差别。
2. 对 App 提供 Runtime，对用户自定义的函数，我们需要尽可能的提供所需的运行环境，比如目前已有的 Python Runtime，到后面会支持的 Java、NodeJs、Go等等。
3. 提供 SDK 和 Service，我们需要为开发者提供更便捷的方式，去使用我们的 baetyl 框架，在 baetyl 框架的基础上进行开发。目前在 SDK 上我们做了这样一些操作，提供了 Dataset（静态数据集），用来存储一些配置文件，可执行文件之类，另外还提供了 Volume（动态存储集），对于一些数据，我们需要做持久化，或者记录日志，我们通过动态数据集，用挂载的方式来实现。

### 目前怎么做的？

接下来我们切入正题，从我目前了解的角度来分析一下已有的架构。架构图如下：

![baetyl-architectures](https://sherlockblaze.com/resources/img/profession/architectures/baetyl-architecture.png)

我们知道，一个程序或者说一个软件，主要做的事情有这三件：

1. 输入
2. 计算
3. 输出

#### 输入

那么我们如何处理输入问题呢？

我们完成了一个模块，叫 `baetyl-hub`，它主要提供基于 MQTT 的消息路由服务。完成对我们的输入进行接受、分发的操作。

目前我们支持 4 种接入方式：

1. TCP
2. SSL（TCP + SSL）
3. WS（Websocket）
4. WSS（Websocket + SSL）

其中，如果使用证书双向认证，Client 必须在连接时发送 非空 的 username 和 空 的 password ，username 会用于主题鉴权。如果 password 不为空，则还会进一步检查密码是否正确

而对于 MQTT 协议支持度大概是：

1. 支持 Connect、Disconnect、Subscribe、Publish、Unsubscribe、Ping 等功能
2. 支持 QoS 等级 0 和 1 的消息发布和订阅
3. 支持 Retain、Will、Clean Session
4. 支持订阅含有 +、# 等通配符的主题
5. 支持符合约定的 ClientID 和 Payload 的校验
6. 暂时 不支持 发布和订阅以 $ 为前缀的主题
7. 暂时 不支持 Client 的 Keep Alive 特性以及 QoS 等级 2 的发布和订阅

如我们上面所说，虽然是官方模块，但如果该模块无法满足要求，是可以使用第三方的 MQTT Broker/Server 来进行替换的。

#### 计算

对于计算，我们提供了 `baetyl-function-manager` 模块，基于 MQTT 消息机制，支持弹性、高可用、可扩展、响应快的计算能力。函数计算模块通过从 `hub` 模块接受消息，触发功能，调用主程序提供的 API 来实现函数实例的启停，通过配置文件中 `instance` 的字段，来界定所能启动的函数实例个数。

我们目前提供到的函数 runtime 有 `baetyl-function-python27`、`baetyl-function-python36` 以及 `opendge-function-sql`。

设计的思路基本是相同的，我们以 `baetyl-function-python27` 作为例子做一次比较浅显的讲解。

`baetyl-function-python27` 提供的 Python 函数和 `CFC（Cloud Function Compute）` 类似，用户可以通过编写的自己的函数来处理消息，可进行消息的过滤、转换和转发等，使用上非常灵活。特别的是，该模块还可以作为 GRPC 服务单独启动，也可以为函数管理模块提供函数运行实例。

#### 输出

我们处理输出的方式比较多样，比如，我们可以定制 `BOS` 模块，将我们需要的数据上传到 BOS，或者写到持久化文件里，或者利用 Kafka等等。在我们目前已有的官方模块里，我们提供了 `baetyl-remote-mqtt` 模块，用来做远程信息同步。在设计上，这个模块的思路主要是作为一个 `MQTT Client` 从 `baetyl-hub` 模块中接受数据，然后转发到另一个远程的 `MQTT Server` 上。

### 在线管理

从目前的描述来看，我们主要了解了在端上如何进行消息的收发，利用函数运行时实例以及CFC来进行消息的处理。那我们如何管理这些模块呢？

因为我们的设计目标其实是，在用户启动这些模块之后，不再需要在端上进行操作，而把控制的内容转移到云上，也就是我们先前说的支持在线管理，我们提供了一个模块 `baetyl-agent`，作为一个代理服务，和云端进行交互，提供云端控制的能力。

连接方式上，`agent` 拥有 MQTT 和 HTTPS 通道，MQTT 强制 SSL/TLS 证书双向认证，HTTPS 强制 SSL/TLS 证书单向认证。

功能上，`agent` 模块目前就做两件事：

1. 启动后定时向主程序获取状态信息并上报给云端
2. 监听云端下发的事件，触发相应的操作，目前只处理应用 OTA 事件

所以说，`agent` 模块赋予了我们从云端管理端上 baetyl 的能力。但如果想实现离线运行，只需要关闭端上运行的 `agent` 模块，就可以实现 `baetyl` 的离线运行，当然也可以在网络不好的情况下被迫离线运行。

### 主程序

上面大概简述了各个模块的功能和大致的设计思想，那我们如何启动这些模块呢？答案是 baetyl 主程序。通过 baetyl 主程序提供的命令行，我们可以很方便的进行 baetyl 系统的启停。

主程序主要由两个部分组成：

1. RESTFUL API
2. Engine 系统

#### Restful API

Restful API 主要用来提供一些通用的 API 供其他模块使用，目前提供的接口有一下五个：

1. GET  `/system/inspect` 用于获取系统信息和状态
2. PUT  `/system/update` 更新系统和服务
3. GET `/ports/available` 获取宿主机的空闲端口
4. PUT `/services/{serviceName}/instances/{instanceName}/start` 动态启动某个服务的一个实例
5. PUT `/services/{serviceName}/instances/{instanceName}/stop` 动态停止某个服务的某个实例

接口暴露的方式有两种，一种是在 Linux 平台下，使用 `Unix Domain Socket`，默认地址为 `/var/run/baetyl.sock`，其他平台则使用 TCP 的方式，默认地址是 `tcp://127.0.0.1:50050`。

既然涉及到接口的调用，我们就需要考虑到鉴权问题，目前主程序采用的鉴权方式为动态 Token，主程序在启动服务时，会为每一个服务动态生成一个 Token，将服务名和Token以环境变量的方式传入服务实例，实例读取后放入请求的 Header 中发给主程序即可。由于动态实例并非通过主程序来启动，所以动态实例没有启动其他实例的权利。

#### 引擎系统

目前我们的 baetyl 提供了两种运行模式：

1. Docker 容器模式
2. Native 进程模式

目前我们已经在很大程度上，实现了两种模式下配置项的统一，在支持 Docker 的运行环境下，我们更建议使用 Docker 容器模式方式的运行，而在不支持容器模式的硬件平台下，我们通过 Native 模式，以存储卷的方式尽可能的模仿容器模式的运行形式。

## Future

至此，我们对 baetyl 架构的大致介绍基本就结束了。接下来，从我个人的视角来展望一下 baetyl 可能的发展。

- 从纵向的角度来看，也就是边缘计算的出发点，我们希望用户在端上启动 baetyl 系统后，无需再直接操作端上设备进行控制，所有的管理操作，都可以在云端完成，这样的情况下，目前的操作系统提供的很多功能，我们都是用不上的。不如对其做一些裁剪，通过 `Linux Kernl + init` 的组合，来实现一个 `baetyl` 的启动，完成一个 `baetyl OS`。

同时我们也可以通过直接的 `containerd` 来实现容器的运行，取代 `Docker`，因为目前的情况来说，Docker 项目正在不断膨胀，变得越来越大，而且在硬件能力较差的端设备上，需要占用更多的资源，造成比较大的性能占用。

- 从横向的角度来看，边缘计算节点未来一定是分布式，可扩展的。我们可以通过启动多个 `baetyl OS`，并且选中一台服务器，启动 `Kubernetes` 的管理模块，一方面和其他 baetyl 节点建立联系，一方面，向云端开放远程管理能力。

## Recap

到这里，我们这次的分享就结束了，主要分享了一下 baetyl 框架在实现我们三个目标上所做的事情：

1. 独立于硬件
2. 支持各种类型的应用程序
3. 支持在线管理和离线运行

谢谢大家。