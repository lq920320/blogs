## DevOps系列-03：什么是容器编排


上一篇我们介绍了容器是什么，以及如何使用 Dockerfile 来构建一个镜像，运行一个容器，然而一个容器基本上是无法完成我们的业务需求的，这时候就要很多个容器配合使用，要协调容器的端口、网络、扩展等等，那就需要对多个容器进行管理和编排了，就像晚会编排节目一样，那个优先，哪个排后，哪个耗时长，哪个需要的资源多等等。

### 容器编排的定义

容器编排是指自动化容器的部署、管理、扩展和联网。

Docker和其他容器引擎可以极大地简化部署服务器端应用程序的许多方面，但许多应用程序由多个容器组成。随着部署了更多的应用程序和服务，管理一组容器只会变得更加困难；这导致了一类称为容器编排器的工具的开发。

容器提供的便利性伴随着一些权衡；如果有人严格遵守 Docker 的想法，即每个服务都应该有自己的容器，那么最终会运行大量的容器。即使是数据库的简单web界面也可能需要为数据库服务器和应用程序运行单独的容器；它还可能包括一个单独的容器，用于web服务器处理服务静态文件，一个代理服务器终止SSL/TLS连接，一个键值存储作为缓存，甚至一个第二应用程序容器来处理后台作业和调度任务。

负责几个此类应用程序的管理员会很快发现自己希望使用一种工具来简化工作；这是容器协调器介入的地方。容器编排器是一种工具，可以将一组多个容器作为一个单元进行管理。与在单个服务器上运行不同，协调器允许将多个服务器合并到一个集群中，并在集群节点之间自动分配容器工作负载。

容器编排可以为需要部署和管理成百上千个Linux容器和主机的企业提供便利，可以在使用容器的任何环境中使用。

使用容器编排可以自动化和管理任务，例如：

- 置备和部署

- 配置和调度

- 资源分配

- 容器可用性

- 根据平衡基础架构中的工作负载而扩展或删除容器

- 负载平衡和流量路由

- 监控容器的健康状况

- 根据运行应用的容器来配置应用

- 保持容器间交互的安全

### 容器编排工具

比较著名的容器编排有 `Docker Swarm`、`Kubernetes` 以及 `Nomad`，其中最著名以及发展最好的便是 `Kubernetes` 了，简称 `K8S`，这也是我们本系列下一篇着重介绍的。下面我们先来简单介绍一下这些容器编排器。

#### Docker Compose 以及 Swarm

`Docker Compose` 其实称不上一个编排器，但它是 Docker 第一次尝试创建的一个工具，以便更轻松地管理由多个容器组成的应用程序。通过一个几乎总是命名为 `docker-compose.yml` 的 YAML 格式的文件，`Compose` 读取此文件并使用 `Docker API` 创建它声明的资源；`Compose` 还会向所有资源添加标签，以便在创建后可以将它们作为一个分组进行管理。实际上，它是在容器组上运行的 Docker 命令行界面 （CLI） 的替代方法。可以在撰写文件中定义三种类型的资源：

- `services` 包含要启动的容器的声明。`services` 中的每一项都等效于 `docker run` 命令。

- `networks` 声明可附加到文件中定义的容器的网络。网络中的每个条目都相当于一个 `docker netword create` 命令。

- `volumes` 可附加到容器的命名卷。在 Docker 的说法中，卷是装载到容器中的持久存储。命名卷由 Docker 守护程序管理。`volumes` 中的每个条目都相当于一个 `docker volume create` 命令。

下面我们可以看一个 `docker-compose.yml` 的例子：

```yaml
# 这里一般也会声明你使用的compose的版本是多少
# 不使用也是可以的，这个配置我看文档已经标记过时了
version: '3'
services:
  # 可以假设我们的项目要启动一个前端服务容器和后端服务容器
  # 同时可以指定容器各自端口号，网络，以及持久存储的卷等
  frontend:
    image: awesome/webapp
    ports:
      - "443:8043"
    networks:
      - front-tier
      - back-tier
    configs:
      - httpd-config
    secrets:
      - server-certificate

  backend:
    image: awesome/database
    volumes:
      - db-data:/etc/data
    networks:
      - back-tier
# 指定挂载卷相关的配置，驱动、占用磁盘大小等
volumes:
  db-data:
    driver: flocker
    driver_opts:
      size: "10GiB"
# 定义配置项
configs:
  httpd-config:
    external: true
# 鉴权配置
secrets:
  server-certificate:
    external: true
# 网络定义
networks:
  # 使用对象足以定义它们
  front-tier: {}
  back-tier: {}
```

此示例说明了`volumes`、`configs` 和 `secrets` 之间的区别。虽然所有这些文件或目录都作为挂载的文件或目录公开给服务容器，但只能配置一个卷进行读+写访问。`configs` 和 `secrets` 是只读的。卷配置允许选择卷驱动程序并传递驱动程序选项，以根据实际基础结构调整卷管理。配置和鉴权依赖于平台服务，并且声明为外部服务，因为它们不作为应用程序生命周期的一部分进行管理：`Compose` 实现将使用特定于平台的查找机制来检索运行时值。

`Compose` 虽说提供了一种更方便的方式来管理由多个容器组成的应用程序，然而，至少在其原始版本中，它只能与单个主机一起使用，它创建的所有容器都在同一台计算机上运行。为了将其覆盖范围扩展到多个主机，Docker 在 2016 年引入了 `Swarm` 模式。这实际上是 Docker 的第二个名称为 “Swarm” 的产品 - 2014 年的产品实现了完全不同的方法来跨多个主机运行容器，但它不再维护。它被 `SwarmKit` 取代，`SwarmKit` 提供了当前版本的 `Docker Swarm` 的基础。

`Swarm` 模式就包含在 Docker 中，无需其他软件。创建集群很简单，只需在初始节点上运行 `docker swarm init`，然后在要添加的每个附加节点上运行 `docker swarm join`。群集群包含两种类型的节点。管理器节点提供了一个 API 来启动集群上的容器，并使用基于 Raft 共识算法（这个我们前面的拜占庭的文章提到过，忘了的去复习）的协议相互通信，以便在所有管理器之间同步集群的状态。工作器节点执行运行容器的实际工作。目前尚不清楚这些集群有多大，Docker 的文档说，集群不应超过 7 个管理器节点，但没有指定工作节点数量的限制。跨节点桥接容器网络是内置的，但在节点之间共享存储不是，需要使用第三方卷插件来提供跨节点的共享持久存储。

服务使用撰写文件部署在群中。`Swarm` 通过向每个服务添加一个[ `deploy` ](https://docs.docker.com/compose/compose-file/deploy/)关键字来扩展 `Compose` 格式，该项指定应运行多少个服务实例以及它们应在哪些节点上运行。

```yaml
services:
  frontend:
    image: awesome/webapp
    ports:
      - "8080:80"
    deploy:
      mode: replicated
      replicas: 2
      endpoint_mode: vip
```

不幸的是，这导致了 `Compose` 和 `Swarm` 之间的分歧，这引起了一些混乱，因为 CPU 和内存配额等选项需要根据所使用的工具以不同的方式指定。在这段分歧期间，用于 `Swarm` 的文件被称为“堆栈文件”而不是 `Compose` 文件，这样试图消除两者的歧义。值得庆幸的是，这些差异似乎在当前版本的 `Swarm` 和`Compose` 中得到了平滑度过，并且任何对堆栈文件与Compose文件不同的引用似乎在很大程度上是从网上搜索的。`Compose` 格式现在有一个 [开放的规范](https://compose-spec.io/) 和它自己的 [GitHub 组织](https://github.com/compose-spec/)，提供参考实现。

`Swarm` 的未来存在一定程度的不确定性。它曾经是一个名为 `Docker Cloud` 服务的骨干，但该服务在 2018 年突然关闭。它也被吹捧为 Docker 企业版的关键功能，但该产品已被出售给另一家公司，现在作为 Mirantis Kubernetes Engine 销售。同时，最新版本的 Compose 已经获得了将容器部署到亚马逊和微软托管的服务的能力。没有弃用公告，但在最近的记忆中也没有任何类型的公告，在 Docker 的网站上搜索 “Swarm” 这个词只会得到顺便提及。

#### Kubernetes

`Kubernetes`（亦被称为 `k8s`）是一个受 Google 内部工具 `Borg` 启发的项目。`Kubernetes` 管理资源并协调在多达数千个节点的集群上运行的工作负载;它主导了容器编排，就像谷歌主导了搜索一样。谷歌希望在 2014 年与 Docker 合作开发 `Kubernetes`，但 Docker 决定通过 `Swarm` 走自己的路。相反，`Kubernetes` 在云原生计算基金会（CNCF）的主持下成长起来。到 2017 年，`Kubernetes` 变得非常受欢迎，以至于 Docker 宣布它将集成到 Docker 自己的产品中。

除了它的受欢迎程度之外，`Kubernetes` 主要以其复杂性而闻名。手动设置新集群是一项复杂的任务，除了 `Kubernetes` 本身之外，还需要管理员选择和配置多个第三方组件。就像 Linux 内核需要与其他软件相结合才能形成完整的操作系统一样，`Kubernetes` 只是一个编排器，需要与其他软件结合使用才能形成一个完整的集群。它需要一个容器引擎来运行其容器，它还需要用于网络和持久卷的插件。

好，作为我们下一篇的主角，我们先知道它是个容器编排器，然后有着超高的入门门槛和复杂度就行了，之后我们再着重介绍和了解。

#### Nomad

`Nomad` 是 HashiCorp（一家公司） 的编排者，HashiCorp 把它作为 `Kubernetes` 的更简单替代品进行推广销售（是不是可以理解成 “no mad”，即以后不要再为 `k8s` 发疯了？哈哈哈哈哈）。`Nomad` 是一个开源项目，就像Docker 和 `Kubernetes` 一样。它由一个名为 `nomad` 的单个二进制文件组成，可用于启动称为代理的守护程序，还可以用作与代理通信的 CLI。根据配置方式，代理进程可以在两种模式之一中运行。在服务器模式下运行的代理接受作业并为其分配群集资源。在客户端模式下运行的代理会联系服务器以接收、运行任务，并将其状态报告回服务器。代理还可以在开发模式下运行，在该模式下，它同时承担客户端和服务器的角色，以形成可用于测试目的的单节点群集。

创建 `Nomad` 集群非常简单。在 `Nomad` 最基本的操作模式下，必须启动初始服务器代理，然后就可以使用 `nomad server join` 命令将其他节点添加到群集中。`HashiCorp` 还提供 `Consul`，这是一个通用的服务网格和发现工具。虽然它可以单独使用，但与 `Consul` 结合使用时，`Nomad` 会处于最佳状态。`Nomad` 代理可以使用 `Consul` 自动发现和加入集群，还可以执行运行状况检查、提供 DNS 记录以及为集群上运行的服务提供 HTTPS 代理。

`Nomad` 支持复杂的集群拓扑。每个集群分为一个或多个“数据中心”。与 `Swarm ` 一样，单个数据中心内的服务器代理使用基于 Raft 的协议相互通信，该协议具有严格的延迟要求，但可以使用gossip协议将多个数据中心链接在一起，该协议允许信息通过集群传播，而无需每个服务器保持与其他服务器的直接连接。从用户的角度来看，以这种方式链接在一起的数据中心可以作为一个集群。这种架构使 Nomad 在扩展到巨大的集群时具有优势。Kubernetes 官方支持多达 5,000 个节点和 300,000 个容器，而 `Nomad` 的文档引用了包含超过 10,000 个节点和 2,000,000 个容器的集群示例。

与 `Kubernetes` 一样，`Nomad` 不包含容器引擎或运行时。它使用任务驱动程序来运行作业。包括使用 Docker 和 Podman 运行容器的任务驱动程序，社区支持的驱动程序可用于其他容器引擎。与 `Kubernetes` 一样，`Nomad` 的野心不仅限于容器，还有适用于其他类型的工作负载的任务驱动程序，包括仅在主机上运行命令的 `fork/exec` 驱动程序、用于运行虚拟机的 QEMU 驱动程序以及用于启动 Java 应用程序的 Java 驱动程序。社区支持的任务驱动因素将 `Nomad` 连接到其他类型的工作负载。

与 Docker 或 `Kubernetes` 不同，`Nomad` 避开了 YAML 而支持 HashiCorp 配置语言（HCL），它最初是为另一个 HashiCorp 项目创建的，用于配置名为`Terraform` 的云资源。HCL 在整个 HashiCorp 产品线中使用，尽管它在其他地方的采用有限。用 HCL 编写的文档可以很容易地转换为 JSON，但它旨在提供一种比 JSON 更友好的语法，并且比 YAML 更不容易出错。

```hcl
io_mode = "async"

service "http" "web_proxy" {
  listen_addr = "127.0.0.1:8080"
  
  process "main" {
    command = ["/usr/local/bin/awesome-app", "server"]
  }

  process "mgmt" {
    command = ["/usr/local/bin/awesome-app", "mgmt"]
  }
}
```

上面的 hcl 配置等同于下面的 json 配置：

```json
{
  "io_mode": "async",
  "service": {
    "http": {
      "web_proxy": {
        "listen_addr": "127.0.0.1:8080",
        "process": {
          "main": {
            "command": ["/usr/local/bin/awesome-app", "server"]
          },
          "mgmt": {
            "command": ["/usr/local/bin/awesome-app", "mgmt"]
          },
        }
      }
    }
  }
}
```

HashiCorp 还有个相当于 `Helm`（ `Kubernetes` 的包管理器） 的产品被称为 `Nomad Pack`。与 `Helm` 一样，`Nomad Pack` 处理一个充满模板和变量声明的目录来生成作业配置。`Nomad` 还有一个预打包应用程序的社区注册表，但选择范围比 Artifact Hub 上 Helm 可用的注册表要小得多。

`Nomad` 的流行程度远不如 `Kubernetes`。与 `Swarm` 一样，它的发展似乎主要由其创建者推动，尽管它已被许多大公司部署，但 HashiCorp 仍然是 `Nomad` 周围社区的中心。在这一点上，该项目似乎不太可能获得足够的动力来独立于其母公司的生活。相比于 Docker 对 `Swarm`，HashiCorp 更明确地致力于 `Nomad` 的开发和推广，用户也许可以从这一事实中找到保证。

### 总结

本篇我们简单介绍了什么是容器编排以及容器编排的一些工具（容器编排器）。

`Swarm`、`Kubernetes` 和 `Nomad` 不是唯一的容器编排器，但它们是最可行的三个。`Apache Mesos` 也可以用于运行容器，但它在 2021 几乎被封存；DC/OS基于 Mesos，但与 Docker Enterprise Edition 非常相似，支持其开发的公司现在专注于 `Kubernetes` 。大多数“其他”容器编排项目，如 `OpenShift` 和 `Rancher`，实际上只是增强（和认证）的 `Kubernetes` 发行版，即使它们的名称中没有`Kubernete` 。

尽管 `Kubernetes` 很复杂（或者可能是因为），但它目前是最受欢迎的，但HashiCorp 与 `Nomad` 的成功表明，仍有其他选择的空间。一些用户仍然忠于`Docker Swarm` 的简单性，但它的未来是不确定的。其他替代方案在这一点上似乎基本上被放弃了。看起来，这三个角色的周围环境已经基本稳定下来，但容器编排仍然是一个相对不成熟的领域。十年前，这项技术几乎不存在，而且事情仍在快速发展。在容器编排方面，可能还有许多令人兴奋的新想法和发展。期待我们的创新和加入。

### 链接：

- The container orchestrator landscape： https://lwn.net/Articles/905164/

- 什么是容器编排？： https://www.redhat.com/zh/topics/containers/what-is-container-orchestration

- Compose specification | Docker Documentation： https://docs.docker.com/compose/compose-file/

- GitHub nomad： https://github.com/hashicorp/nomad

- GitHub HCL： https://github.com/hashicorp/hcl

- Consul： https://www.consul.io/

- Helm | Docs： https://helm.sh/zh/docs/














