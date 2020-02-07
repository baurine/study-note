# K8S Note

## Note for 《Kubernetes 从上手到实践》

- [Kubernetes 从上手到实践](https://juejin.im/book/5b9b2dc86fb9a05d0f16c8ac)

### 1. 开篇： Kubernetes 是什么以及为什么需要它

k8s，容器编排

### 2. 初步认识：Kubernetes 基础概念

(感觉描述得不是很清楚)

- Node: 用于工作的服务器 (或者虚拟机)
- Deployment: 对期望状态的描述
- Pod: 集群中可调度的最小单元，可以是一组容器
- Container Runtime: 容器运行时，一般使用 Docker，但也有其它的容器运行时

一个集群由多个 Node 组成，每个 Node 上可以布署多个 Pod，一个 Pod 可以包含一个或多个容器。

### 3. 宏观认识：整体架构

c/s 架构

cli (kubectl) ---> kubernets server (master) <----> node / node / node

```
                               +-------------+
                               |             |
                               |             |               +---------------+
                               |             |       +-----> |     Node 1    |
                               | Kubernetes  |       |       +---------------+
+-----------------+            |   Server    |       |
|       CLI       |            |             |       |       +---------------+
|    (Kubectl)    |----------->| ( Master )  |<------+-----> |     Node 2    |
|                 |            |             |       |       +---------------+
+-----------------+            |             |       |
                               |             |       |       +---------------+
                               |             |       +-----> |     Node 3    |
                               |             |               +---------------+
                               +-------------+
```

master 是大脑，几个功能：

- 接收：外部请求和集群内部的通知反馈
- 发布：对集群整体的调度和管理
- 存储：存储

包含几个重要组件：

- Cluster state store: 存储集群需要持久化的状态，并通知各组件状态变更，使用 etcd k/v 存储。
- API Server: 集群入口，接收外部信号和请求，并将一些信号写到 etcd
- Controller Manager: 在后台运行着许多不同的控制器进程，用来调节集群的状态
- Scheduler: 集群的调度器，它会持续关注集群中未被调度的 Pod，并根据各种条件，将 Pod 绑定到 Node 上

```
+----------------------------------------------------------+
| Master                                                   |
|              +-------------------------+                 |
|     +------->|        API Server       |<--------+       |
|     |        |                         |         |       |
|     v        +-------------------------+         v       |
|   +----------------+     ^      +--------------------+   |
|   |                |     |      |                    |   |
|   |   Scheduler    |     |      | Controller Manager |   |
|   |                |     |      |                    |   |
|   +----------------+     v      +--------------------+   |
| +------------------------------------------------------+ |
| |                                                      | |
| |                Cluster state store                   | |
| |                                                      | |
| +------------------------------------------------------+ |
+----------------------------------------------------------+
```

Node：简单理解为加入集群中的机器

Node 是如何加入集群调度并运行服务的？归功于 Node 上的几个核心组件：

- kubelet: 实现了集群中最重要的关于 Node 和 Pod 的控制功能。Pod 能否运行于 Node 上，由 kubelet 决定。Pod 可以是一组容器
- Container Runtime: 最主要功能是下载镜像和运行容器，最常见是 Docker，也有一些其它实现，比如 rkt, cri-o。容器运行时接口 CRI
- kube-proxy: 提供一种代理的服务，可以通过 Service 访问到 Pod

```
+--------------------------------------------------------+
| +---------------------+        +---------------------+ |
| |      kubelet        |        |     kube-proxy      | |
| |                     |        |                     | |
| +---------------------+        +---------------------+ |
| +----------------------------------------------------+ |
| | Container Runtime (Docker)                         | |
| | +---------------------+    +---------------------+ | |
| | |Pod                  |    |Pod                  | | |
| | | +-----+ +-----+     |    |+-----++-----++-----+| | |
| | | |C1   | |C2   |     |    ||C1   ||C2   ||C3   || | |
| | | |     | |     |     |    ||     ||     ||     || | |
| | | +-----+ +-----+     |    |+-----++-----++-----+| | |
| | +---------------------+    +---------------------+ | |
| +----------------------------------------------------+ |
+--------------------------------------------------------+
```

### 4. 搭建 Kubernetes 集群 - 本地快速搭建

minikube

配置：

- 首先安装 virtual box (备选：vm fusion, hyperkit)
- 再安装 kubectl: `brew install kubectl`
- 安装 minikube: `brew install minikube`

创建集群：`minikube start`。(结果还是自动选了 hyperkit，而不是 virtual box)

`minikube start --vm-driver=virtualbox` 指定使用 virtualbox

遇到错误：

```
> minikube start --vm-driver=virtualbox
> 😄 minikube v1.6.2 on Darwin 10.15.1
> ✨ Selecting 'virtualbox' driver from user configuration (alternates: [hyperkit])
> 🔥 Creating virtualbox VM (CPUs=2, Memory=2000MB, Disk=20000MB) ...
> panic: runtime error: invalid memory address or nil pointer dereference
> [signal SIGSEGV: segmentation violation code=0x1 addr=0x28 pc=0x4ecd18f]
```

解决：https://github.com/kubernetes/minikube/issues/6147

```
> minikube start --vm-driver=virtualbox --image-repository="registry.cn-hangzhou.aliyuncs.com/google_containers"
> 😄 minikube v1.6.2 on Darwin 10.15.1
> ✨ Selecting 'virtualbox' driver from user configuration (alternates: [hyperkit])
> ✅ Using image repository registry.cn-hangzhou.aliyuncs.com/google_containers
> 🔥 Creating virtualbox VM (CPUs=2, Memory=2000MB, Disk=20000MB) ...
> 🐳 Preparing Kubernetes v1.17.0 on Docker '19.03.5' ...
> 💾 Downloading kubeadm v1.17.0
```

验证集群是否配置正确：

```
> kubectl cluster-info
Kubernetes master is running at https://192.168.99.101:8443
KubeDNS is running at https://192.168.99.101:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

> kubectl get nodes
NAME       STATUS   ROLES    AGE    VERSION
minikube   Ready    master   5m4s   v1.17.0
```

打开 dashboard: `minikube dashboard`

总结：

> Minikube 的本质其实是将一套 "定制化" 的 K8S 集群打包成 ISO 镜像，当执行 `minikube start` 的时候，便通过此镜像启动一个虚拟机，在此虚拟机上通过 kubeadm 工具来搭建一套只有一个节点的 K8S 集群。关于 kubeadm 工具，我们在下节进行讲解。

> 同时，会通过虚拟机的相关配置接口拿到刚才所启动虚拟机的地址信息等，并完成本地的 kubectl 工具的配置，以便于让用户通过 kubectl 工具对集群进行操作。

### 5. 动手实践：搭建一个 Kubernetes 集群 - 生产可用

方案选择：kubeadm

> kubeadm 是 Kubernetes 官方提供的一个 CLI 工具，可以很方便的搭建一套符合官方最佳实践的最小化可用集群。当我们使用 kubeadm 搭建集群时，集群可以通过 K8S 的一致性测试，并且 kubeadm 还支持其他的集群生命周期功能，比如升级/降级等。

这个要在服务器上操作，然而我并没有，所以，先跳过吧...

主要操作：

准备：

- 关闭 swap
- 安装 docker
- 安装 kubectl
- 安装 kubeadm 和 kubelet (两者在一个安装包中)

配置

- 用 systemd 管理 kubelet

安装前置依赖：

- cri-tools
  - crictl : kubelet CRI 的 cli
  - critest: kubelet CRI 的测试工具集
- socat : 端口转发

初始化集群：`kubeadm init`

验证：`kubectl cluster-info`

配置 kubectl，略。

配置集群网络，暂略，CNI，flannel

新增 Node: 在新机器上做相同的配置后，执行 `kubeadm join [options]`，新的 node 加入集群，第一个 node 为 master，在 master 上执行 `kubectl get nodes` 可以看到有 2 个 nodes 了。

```
[root@master ~]# kubectl get nodes
NAME      STATUS    ROLES     AGE       VERSION
master    Ready     master    12m       v1.11.3
node1     Ready     <none>    7m        v1.11.3
```

### 6. 集群管理：初识 kubectl

基础配置：~/.kube/config

基本命令：

- `kubectl get nodes`

explain:

- `kubectl explain node`

### 7. 集群管理：以 Redis 为例-部署及访问

Pod 是 K8S 中最小的调度单元，所以我们无法直接在 K8S 中运行一个 container 但是我们可以运行一个 Pod 而这个 Pod 中只包含一个 container。

部署：

部署一个 redis 实例：`kubectl run redis --image='redis:alpine'`

查看所有组件：`kubectl get all`

```
> kubectl get all
NAME                         READY   STATUS    RESTARTS   AGE
pod/redis-5c74dbc6f7-lfvkz   1/1     Running   0          48s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   3h12m

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/redis   1/1     1            1           48s

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/redis-5c74dbc6f7   1         1         1       48s
```

Deployment 是一种高级抽象，允许我们扩容，滚动更新及降级等操作，我们使用 `kubectl run redis --image='redis:alpine` 命令便创建了一个名为 redis 的 Deployment，并指定了其使用的镜像为 redis:alpine。

同时 K8S 会默认为其增加一些标签（Label）。label 可以用来滚动发布时，选择特定的 label 进行操作。

ReplicaSet 是一种较低级别的结构，允许进行扩容。

Pod 托管给 ReplicaSet，而 ReplicaSet 则会去检查当前的 Pod 数量及状态是否符合预期，并尽量满足这一预期。

Service 简单点说就是为了能有个稳定的入口访问我们的应用服务或者是一组 Pod。通过 Service 可以很方便的实现服务发现和负载均衡。

后面的看着有点吃力了。

Deployment / Serveice / ReplicaSet / Node

这书写得不行啊

今天就看到这吧

### 8. 安全重点: 认证和授权

三重安全访问：用于识别用户身份的认证（Authentication），用于控制用户对资源访问的授权（Authorization）以及用于资源管理方面的准入控制（Admission Control）。

授权：RBAC 方式使用最多 (Role-based access control)

先跳过

### 9. 应用发布：部署实际项目

如何将实际项目部署至 K8S 中

先准备一个项目，分前端，后端，work，nginx 四个模块。

- 先分别编写 dockerfile，然后使用 docker build 生成 docker 镜像
- 使用 docker compose 进行简单编排验证，编写 docker-compose.yml，`docker compose up` / `docker compose ps`

然后尝试布署到 k8s 中

先定一个 Namespace，name 为 work

namespace.yaml

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: work
```

部署 namespace: `kubectl apply -f namespace.yaml`

因为 redis 是无依赖的，所以先布署 redis

编写 Deployment redis-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: redis
  name: saythx-redis
  namespace: work
spec:
  selector:
    matchLabels:
      app: redis
  replicas: 1
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
        - image: redis:5
          name: redis
          ports:
            - containerPort: 6379
```

Deployment 各字段含义：

- apiVersion ：指定了 API 的版本号，当前我们使用的 K8S 中， Deployment 的版本号为 apps/v1，而在 1.9 之前使用的版本则为 apps/v1beta2，在 1.8 之前的版本使用的版本为 extensions/v1beta1。在编写配置文件时需要格外注意。
- kind ：指定了资源的类型。这里指定为 Deployment 说明是一次部署。
- metadata ：指定了资源的元信息。例如其中的 name 和 namespace 分别表示资源名称和所归属的 Namespace。
- spec ：指定了对资源的配置信息。例如其中的 replicas 指定了副本数当前指定为 1。template.spec 则指定了 Pod 中容器的配置信息，这里的 Pod 中只部署了一个容器。

部署：

```sh
$ kubectl -n work get all
$ kubectl apply -f redis-deployment.yaml
$ kubectl -n work get all
...
```

Pod 正常运行，进入 Pod 内进行测试。`kubectl -n work exec -it saythx-redis-79d8f9864d-x8fp9 bash`。

Redis service 由于 Redis 是后端服务的依赖，我们将它作为 Service 暴露出来。相互间通过 Service 访问，而不是直接访问 Pod。

编写 redis-service.yaml

```
apiVersion: v1
kind: Service
metadata:
  labels:
    app: redis
  name: saythx-redis
  namespace: work
spec:
  ports:
  - protocol: TCP
    port: 6379
    targetPort: 6379
  selector:
    app: redis
  type: NodePort

```

布署：`kubectl apply -f redis-service.yaml`

接下来依次为后端服务/work/前端编写 Deployment 和 Service。

通过 Deployment 中的 replicas 的值为进行扩缩容，这个值即为 Pod 的数量。

按照步骤在本地操作了一遍，顺利跑起来了。

### 10. 应用管理：初识 Helm

从上一小节可以看出，手工一步一步操作很麻烦。我们需要更简单的机制，Helm 应运而生。

Helm 是构建于 K8S 之上的包管理器，可与我们平时接触到的 Yum，APT，Homebrew 或者 Pip 等包管理器相类比。

使用 Helm 可简化包分发，安装，版本管理等操作流程。同时它也是 CNCF 孵化项目。

helm 也是 c/s 架色，c 是 helm，s 是 tiller (... helm3 移除了 tiller)

Helm 概念：

Chart: chart 就是 Helm 所管理的包，类似于 Yum 所管理的 rpm 包或是 Homebrew 管理的 Formulae。它包含着一个应用要部署至 K8S 上所必须的所有资源。

Release 就是 chart 在 K8S 上部署后的实例。chart 的每次部署都将产生一次 Release。

Repository 就是字面意思，存储 chart 的仓库。

Config: 前面提到了 chart 是应用程序所必须的资源，当然我们实际部署的时候，可能就需要有些自定义的配置了。Config 便是用于完成此项功能的，在部署时候，会将 config 与 chart 进行合并，共同构成我们将部署的应用。

- [玩 K8S 不得不会的 HELM](https://juejin.im/post/5d5b8dba6fb9a06b1b19c21b)
- [前端领域的 Docker 与 Kubernetes](https://juejin.im/post/5dddd15b6fb9a071576dbd7a)

Service 是一组 Pod 的入口，一个 Pod 里包含一个或多个 container

Deployment 控制 Pod 的副本个数

### 11. 部署实践：以 Helm 部署项目

创建 chart: `helm create saythx`

然后生成 saythx 目录，目录中有 Chart.yaml，类似 package.json，是此 chart 的 manifest，第 9 小节的那些单独配置文件集中放到 templates 中，values.yaml 用来定义变量，供 templates 中的 yaml 文件使用。

布署：`helm install saythx`

打包分发，略。

使用 Helm 我们还可以管理 Release ，并进行更新回滚等操作。

操作了一番，还真跑起来了。

因为之前已经手工布了一个 namespace 为 work 的集群了，所以第一次 install 的失败，然后把 values.yaml 中的 namespace 改成 work2 后再次 install 成功。

```
> export NODE_PORT=$(kubectl get --namespace work2 -o jsonpath="{.spec.ports[0].nodePort}" services saythx-frontend)
> export NODE_IP=$(kubectl get nodes --namespace work2 -o jsonpath="{.items[0].status.addresses[0].address}")
> echo http://$NODE_IP:$NODE_PORT
> http://192.168.99.102:30158
```

awesome!

### 12. 庖丁解牛：kube-apiserver

kube-apiserver：对外提供 api 访问接口，提供认证和授权功能。

使用 kubectl 访问 api 时，会先加载 ~/.kube/config，这个文件里声明了认证所需的文件路径。

如果直接用 curl 访问的话，由于不会加载 ~/.kube/config，认证会失败。

可以先用 kubectl proxy 再用 curl，kubectl proxy 会加载 ~/.kube/config，之后 curl 的每个访问都会带上认证。

kube-apiserver 的基本工作逻辑，各类 API 请求先后通过认证，授权，准入控制等一系列环节后，进入到 kube-apiserver 的 Registry 进行相关逻辑处理。

### 13. 庖丁解牛：etcd

etcd 是由 CoreOS 团队发起的一个分布式，强一致的键值存储。它用 Go 语言编写，使用 Raft 协议作为一致性算法。多数情况下会用于分布式系统中的服务注册发现，或是用于存储系统的关键数据。

etcd 在 K8S 中，最主要的作用便是其高可用，强一致的键值存储以及监听机制。

在 kube-apiserver 收到对应请求经过一系列的处理后，最终如果是集群所需要存储的数据，便会存储至 etcd 中。部分主要是集群状态信息和元信息。

### 14. 庖丁解牛：controller-manager

k8s master 中最核心的一块。用来控制按 Deployment 预期进行工作，比如 Deployment 描述有 2 个 Pod，当我们 kill 掉一个 Pod 后，controller-manager 会自动重新创建一个 Pod。

### 15. 庖丁解牛：kube-scheduler

Master 是 K8S 是集群的大脑，Controller Manager 负责将集群调整至预期的状态，而 Scheduler 则是集群调度器，将预期的 Pod 资源调度到正确的 Node 节点上，进而令该 Pod 可完成启动。

从上层的角度来看，kube-scheduler 的作用就是将待调度的 Pod 调度至最佳的 Node 上，而这个过程中则需要根据不同的策略，考虑到 Node 的资源使用情况，比如端口，内存，存储等。

### 16. kubelet

kubelet 在 Node 上，它所承担的是 agenet 的角色。

可以看到 kubelet 不仅将自己注册给了 kube-apiserver，同时它所在机器的信息也都进行了上报，包括 CPU，内存，IP 信息等。

kube-scheduler 处理了 Pod 应该调度至哪个 Node，而 kubelet 则是保障该 Pod 能按照预期，在对应 Node 上启动并保持工作。

### 17. 庖丁解牛：kube-proxy

kube-proxy 是 K8S 运行于每个 Node 上的网络代理组件，提供了 TCP 和 UDP 的连接转发支持。

Pod 会动态地创建和销毁，但 Service 则较为稳定。使用 Service 将后端 Pod 暴露。

kubeproxy 如何工作：

默认情况下我们使用 iptables 的代理模式，当创建新的 Service ，或者 Pod 进行变化时，kube-proxy 便会去维护 iptables 规则，以确保请求可以正确的到达后端服务。

### 18. 庖丁解牛：Container Runtime（Docker）

自 K8S 1.5（2016 年 11 月）开始，新增了一个容器运行时的插件 API，并称之为 CRI（Container Runtime Interface），通过 CRI 可以支持 kubelet 使用不同的容器运行时，而不需要重新编译。

当前使用最为广泛的是 Docker，当前还支持的主要有 runc，Containerd，runV 以及 rkt 等。

### 19. troubleshoot

一些排除问题的命令和工具，详略。

### 20. 扩展增强：Dashboard

略。

### 21. 扩展增强：CoreDNS

DNS 服务器，现在成为 k8s 默认 DNS 服务器。

### 22. 服务增强：ingress

Ingress 是一组允许外部请求进入集群的路由规则的集合。它可以给 Service 提供集群外部访问的 URL，负载均衡，SSL 终止等。
直白点说，Ingress 就类似起到了智能路由的角色，外部流量到达 Ingress ，再由它按已经制定好的规则分发到不同的后端服务中去。

在本节中，我们介绍了 Ingress 的基本情况，了解了它是 K8S 中的一种资源对象，主要负责将集群外部流量与集群内服务的通信。但它的正常工作离不开 Ingress Controller ，当前官方团队维护的主要有两个 GLBC 和 NGINX Ingress Controller。

### 23.监控实践：对 K8S 集群进行监控

prometheus + grafana

详略。

### 24. 总结

略。大致有了个概念，对整体架构大致了解了。

- 客户端：kubectl
- 服务端
  - master
    - apiserver
    - etcd
    - controll manager
    - scheduler
  - node
    - kubelet
    - kubeproxy
    - CR
