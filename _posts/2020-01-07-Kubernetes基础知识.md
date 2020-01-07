---
layout:     post
title:      Kubernetes基础知识
subtitle:   Kubernetes/集群/运维
date:       2020-01-07
author:     Faymi
header-img: img/post-bg-rwd.jpg
catalog: true
tags:
    - Kubernetes
    - 运维
    - 容器
---



## Kubernetes Learning

### 创建一个k8s集群
- 查看 minikube 版本，检查其是否已安装

```
$ minikube version

minikube version: v1.6.2
commit: 54f28ac5d3a815d1196cd5d57d707439ee4bb392
```


- 启动 k8s 集群

```
$ minikube start

* minikube v1.6.2 on Ubuntu 18.04
* Selecting 'none' driver from user configuration (alternates: [])
* Running on localhost (CPUs=2, Memory=2461MB, Disk=47990MB) ...
* OS release is Ubuntu 18.04.3 LTS
* Preparing Kubernetes v1.17.0 on Docker '18.09.7' ...
  - kubelet.resolv-conf=/run/systemd/resolve/resolv.conf
* Pulling images ...
* Launching Kubernetes ...
* Configuring local host environment ...
* Waiting for cluster to come online ...
* Done! kubectl is now configured to use "minikube"
```

至此，Minikube 已为你启动了一个虚拟机，且k8s集群已在该虚拟机中运行。


- 集群版本，检查 kubectl 是否安装

```
$ kubectl version

Client Version: version.Info{Major:"1", Minor:"17", GitVersion:"v1.17.0", GitCommit:"70132b0f130acc0bed193d9ba59dd186f0e634cf", GitTreeState:"clean", BuildDate:"2019-12-07T21:20:10Z", GoVersion:"go1.13.4", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"17", GitVersion:"v1.17.0", GitCommit:"70132b0f130acc0bed193d9ba59dd186f0e634cf", GitTreeState:"clean", BuildDate:"2019-12-07T21:12:17Z", GoVersion:"go1.13.4", Compiler:"gc", Platform:"linux/amd64"}
```

Client Version 是kubectl版本，Server Version 是主安装Kubernetes版本


- 集群详细信息

```
$ kubectl cluster-info

Kubernetes master is running at https://172.17.0.56:8443
KubeDNS is running at https://172.17.0.56:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```


- 查看集群中的Nodes节点

```
$ kubectl get nodes

NAME       STATUS   ROLES    AGE    VERSION
minikube   Ready    master   119s   v1.17.0
```

显示只有一个节点，且可以看到其状态为 ready （准备接受要部署的应用程序），Kubernetes将根据Node可用资源选择将我们的应用程序部署到何处。

### k8s部署 - Kubernetes Deployments
拥有运行中的Kubernetes集群后，您可以在其之上部署容器化的应用程序。为此，您将创建一个Kubernetes部署配置。部署指示Kubernetes如何创建和更新应用程序实例。创建部署后，Kubernetes主服务器将提到的应用程序实例调度到集群中的各个节点上。

创建应用程序实例后，Kubernetes部署控制器将持续监视这些实例。如果承载实例的节点发生故障或被删除，则部署控制器将实例替换为群集中另一个节点上的实例。这提供了一种自我修复机制来解决机器故障或维护。

创建部署时，需要为应用程序指定容器映像以及要运行的副本数（replicas）。您可以稍后通过更新部署来更改该信息。


- 部署应用

通过 `kubectl creat deployment` 命令行部署app。需要提供部署名称和应用程序映像位置（包括Docker Hub外部托管的映像的完整存储库URL）。

```
$ kubectl create deployment kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1

deployment.apps/kubernetes-bootcamp created
```

这个操作执行了一下几件事：
> 搜索可以在其中运行应用程序实例的合适节点（示例中我们只有1个可用节点）

> 安排应用程序在该节点上运行

> 配置集群以在需要时在新节点上重新安排实例


- 列出部署的应用

```
$ kubectl get deployments

NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   1/1     1            1           13m
```


- 查看app应用

Kubernetes内部运行的Pod在私有的隔离网络上运行。默认情况下，它们在同一kubernetes集群中的其他Pod和服务中可见，但在该网络外部不可见。当使用`kubectl`时，我们通过API端点进行交互以与我们的应用程序进行通信。

kubectl命令可以创建一个代理，该代理会将通信转发到群集范围的专用网络中。可以通过按Ctrl-C终止代理，并且在运行时不显示任何输出。

开另一个terminal，运行：

```
$ kubectl proxy

Starting to serve on 127.0.0.1:8001
```

```
$ curl http://localhost:8001/version

{
  "major": "1",
  "minor": "15",
  "gitVersion": "v1.15.0",
  "gitCommit": "e8462b5b5dc2584fdcd18e6bcfe9f1e4d970a529",
  "gitTreeState": "clean",
  "buildDate": "2019-06-19T16:32:14Z",
  "goVersion": "go1.12.5",
  "compiler": "gc",
  "platform": "linux/amd64"
}
```

为了在不使用 Proxy（代理）的情况下可以访问新部署，可以通过 Service（服务）实现。

### Kubernetes Pods

Pod是Kubernetes的抽象，代表一组一个或多个应用程序容器（例如Docker或rkt）以及这些容器的一些共享资源。这些资源包括：

> 共享存储，如Volumes（卷）

> 网络，如唯一的集群IP地址

> 有关如何运行每个容器的信息，例如容器映像版本或要使用的特定端口

Pod为特定于应用程序的“逻辑主机”建模，并且可以包含相对紧密耦合的不同应用程序容器。
例如，一个Pod可能同时包含带有Node.js应用程序的容器以及一个不同的容器，该容器将提供要由Node.js Web服务器发布的数据。
Pod中的容器共享一个IP地址和端口空间，它们始终位于同一位置并共同调度，并在同一节点上的共享上下文中运行。

Pods是Kubernetes平台上的最小单位。当我们在Kubernetes上创建Deployment时，该Deployment将在Pod中创建内部包含容器的Pod（与直接创建容器相反）。每个Pod都绑定到计划的节点上，并保持在那里，直到终止（根据重新启动策略）或删除为止。如果节点发生故障，则会在群集中的其他可用节点上调度相同的Pod。


![Pods Overview](https://raw.githubusercontent.com/faymi/static-repository/master/images/pods_20200107150328.jpg)

### Nodes

Pod始终在节点上运行。Node是Kubernetes中依赖于集群上的工作机，并且可以是虚拟机或物理机。每个节点由Master管理。一个节点可以有多个Pod，Kubernetes主节点会自动处理跨集群中所有Node调度Pod的过程。Master的自动调度会计算每个节点上的可用资源。

每个Kubernetes节点至少运行：

> Kubelet，一个负责Kubernetes Master与Node之间通信的过程；它管理在机器上运行的Pod和容器

> 容器运行时（例如Docker，rkt）负责从注册表中提取容器映像，解压缩容器并运行应用程序。


![Node Overview](https://raw.githubusercontent.com/faymi/static-repository/master/images/Node_Overview_20200107151237.jpg)

### kubectl 相关命令

- `kubectl get` - 列出资源
- `kubectl describe` - 显示有关资源的详细信息，如containers、images信息
- `kubectl logs` - 打印Pod中的容器的日志
- `kubectl exec` - 在Pod的中容器执行命令

例子：

**列出资源**

```
$ kubectl get pods
NAME                                   READY   STATUS    RESTARTS   AGE
kubernetes-bootcamp-5b48cfdcbd-rzz2b   1/1     Running   0          4m35s
```


**显示有关资源的详细信息**

```
$ kubectl describe pods

Name:           kubernetes-bootcamp-5b48cfdcbd-rzz2b
Namespace:      default
Priority:       0
Node:           minikube/172.17.0.24
Start Time:     Tue, 07 Jan 2020 07:31:10 +0000
Labels:         pod-template-hash=5b48cfdcbd
                run=kubernetes-bootcamp
Annotations:    <none>
Status:         Running
IP:             172.18.0.4
Controlled By:  ReplicaSet/kubernetes-bootcamp-5b48cfdcbd
Containers:
  kubernetes-bootcamp:
    Container ID:   docker://38c46180f69053aa22e85d3c0fd769eb0272f634b6ec732b5d206d269628f0fb
    Image:          gcr.io/google-samples/kubernetes-bootcamp:v1
    Image ID:       docker-pullable://jocatalin/kubernetes-bootcamp@sha256:0d6b8ee63bb57c5f5b6156f446b3bc3b3c143d233037f3a2f00e279c8fcc64af
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Tue, 07 Jan 2020 07:31:30 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-8gxjf (ro)
.....
```


**设置Pod名称的环境变量，示例用到**

```
export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}')
```


**打印Pod的容器日志**

```
$ kubectl logs $POD_NAME
```


**列出环境变量**

```
$ kubectl exec $POD_NAME env
```


**进入Pod中的容器**

```
$ kubectl exec -ti $POD_NAME bash
```
*<font color="orange">注：若在Pod中只有一个容器，容器的名称可以省略，如上面的例子。</font>*