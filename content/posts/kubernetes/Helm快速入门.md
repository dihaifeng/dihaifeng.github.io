---
title: "Helm快速入门"
date: 2024-04-24T17:57:56+08:00
comment: true
tags: ["Helm"]
categories: ["Kubernetes"]
---

Helm 是 Kubernetes 界的瑞士军刀，一个强大的包管理工具，它让部署和管理 Kubernetes 应用程序变得前所未有的简单。它通过 Helm Chart（一组预先配置的 Kubernetes 资源）来实现这一目标。
## Helm工作流程

{{< figure src="https://github.com/dihaifeng/dihaifeng.github.io/raw/main/static/images/Helm-chart-architecture.png" title="工作流程" >}}

从这个架构图中，我们可以看到 Helm 如何与 Kubernetes 的各个组件协同工作，以便于部署和管理 Helm Charts。Helm 通过 Kubernetes API 与集群交互，使用 Helm CLI 来管理仓库中的 Charts，并通过 Releases 来跟踪和管理部署在 Kubernetes 集群上的应用程序。
## 安装 Helm

Helm 的安装非常简便，提供了多种方法以适应不同的操作系统和环境。

### 脚本安装
```bash
[root@master ~]#  curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
[root@master ~]#  chmod 700 get_helm.sh
[root@master ~]#  ./get_helm.sh

# 验证安装结果，查看helm版本
[root@master ~]# helm version
version.BuildInfo{Version:"v3.14.2", GitCommit:"c309b6f0ff63856811846ce18f3bdc93d2b4d54b", GitTreeState:"clean", GoVersion:"go1.21.7"}
```
### 二进制安装
下载地址：[https://github.com/helm/helm/releases](https://github.com/helm/helm/releases)

![在这里插入图片描述](https://github.com/dihaifeng/dihaifeng.github.io/raw/main/static/images/be945617670c4ad1995e21565782b8c3.png)

```bash
# 下载二进制文件
[root@master ~]#  wget https://get.helm.sh/helm-v3.14.2-linux-amd64.tar.gz

# 使用命令提取二进制文件
[root@master ~]# tar -zxvf helm-v3.14.2-linux-amd64.tar.gz
[root@master ~]#mv linux-amd64/helm /usr/local/bin/helm
```
### macOS 安装
```bash
brew install helm
 ```

## 创建 Helm Chart：快速入门

创建一个基本的 Helm Chart，只需几个命令即可开始。

```bash
[root@master ~]#  helm create helloworld
[root@master helloworld]# tree
.
├── charts
├── Chart.yaml
├── templates
│   ├── deployment.yaml
│   ├── _helpers.tpl
│   ├── hpa.yaml
│   ├── ingress.yaml
│   ├── NOTES.txt
│   ├── serviceaccount.yaml
│   ├── service.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml

3 directories, 10 files

```

这将在 `helloworld` 目录下生成一系列的文件和目录，包括 `deployment.yaml` 和 `values.yaml` 等，这些文件共同定义了部署的形态。

### 定制 Helm Chart

定制 Helm Chart 常常涉及到修改 `values.yaml` 文件，如下所示：

```yaml
[root@master helloworld]# cat values.yaml
...
service:
  type: NodePort # 默认为ClusterIP类型
  port: 80
...
```
通过这种方式，无需更改 Chart 的模板文件，就可以轻松调整部署的具体参数。
### 部署 Helm Chart

部署一个 Helm Chart，可以使用以下命令：

```bash
[root@master ~]# helm install myhelloworld helloworld
NAME: myhelloworld
LAST DEPLOYED: Tue Apr 23 10:07:31 2024
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
1. Get the application URL by running these commands:
  export NODE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].nodePort}" services myhelloworld)
  export NODE_IP=$(kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[0].address}")
  echo http://$NODE_IP:$NODE_PORT
```
这个命令会将 `helloworld` Chart 部署到 Kubernetes，并命名为 `myhelloworld`。
### 跟踪和升级 Helm 部署

Helm 提供了全面的命令来跟踪和管理部署。

#### **查看部署**
```bash
[root@master ~]# helm list -a
NAME        	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART           	APP VERSION
myhelloworld	default  	1       	2024-04-23 10:07:31.873730017 +0800 CST	deployed	helloworld-0.1.0	1.16.0
[root@master ~]#
  ```
#### 获取服务信息
```bash
[root@master helloworld]# kubectl get svc myhelloworld
NAME           TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
myhelloworld   NodePort   10.99.133.63   <none>        80:31814/TCP   14m
  ```
{{< admonition type=tip title="注意" >}}
NodePort 端口号范围在30000-32767内变化，因此可能会得到不同的NodePort。
{{< /admonition >}}

集群IP + NodePort的端口31814，我们就可以访问 myhelloworld Helm Chart 的 Nginx 页面。

![在这里插入图片描述](https://github.com/dihaifeng/dihaifeng.github.io/raw/main/static/images/9bae0aaae2564c058c9087126d8fa15b.png)

## 管理 Helm Chart 仓库
在 Linux 发行版中，我们有 apt、yum、dnf 等包管理器；类似地，Helm 依赖于如 bitnami 这样的 Chart 仓库。Chart 开发者可以创建 YAML 配置文件，并将它们打包成 Charts，然后发布为 Chart 仓库。

例如 - 如果你想通过 Helm 仓库在 Kubernetes 集群中部署 Redis 内存缓存，你只需简单地运行以下命令：

```bash
[root@master ~]#  helm install redis bitnami/redis
```

>**注意：** 上述命令将在 bitnami Chart 仓库中搜索 Redis Chart，然后将其安装到你的 Kubernetes 集群中。

Helm 提供了五个与仓库相关的命令，它们可以用于添加、列出、移除、更新和索引 Chart 仓库。
### 添加仓库
```bash
[root@master linux-amd64]# helm repo add bitnami https://charts.bitnami.com/bitnami

# 使用helm search返回bitnami存储库中可用的所有Charts
[root@master linux-amd64]# helm search repo bitnami

NAME                                        	CHART VERSION	APP VERSION  	DESCRIPTION
bitnami/airflow                             	13.0.0       	2.3.3        	Apache Airflow is a tool to express and execute...
bitnami/apache                              	9.1.14       	2.4.54       	Apache HTTP Server is an open-source HTTP serve...
bitnami/argo-cd                             	4.0.4        	2.4.8        	Argo CD is a continuous delivery tool for Kuber...
bitnami/argo-workflows                      	2.3.7        	3.3.8        	Argo Workflows is meant to orchestrate Kubernet...
bitnami/aspnet-core                         	3.4.16       	6.0.7        	ASP.NET Core is an open-source framework for we...
bitnami/cassandra                           	9.2.9        	4.0.5        	Apache Cassandra is an open source distributed ...
bitnami/cert-manager                        	0.7.5        	1.9.1        	Cert Manager is a Kubernetes add-on to automate...
bitnami/common                              	1.16.1       	1.16.0       	A Library Helm Chart for grouping common logic ...
...
```
### 列出仓库
 ```bash
[root@master linux-amd64]# helm repo list
NAME                           	URL
brigade                        	https://brigadecore.github.io/charts
bitnami                        	https://charts.bitnami.com/bitnami
hashicorp                      	https://helm.releases.hashicorp.com
nfs-subdir-external-provisioner	https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
harbor                         	https://helm.goharbor.io
stable                         	http://mirror.azure.cn/kubernetes/charts/
  ```

### 更新仓库
```bash
[root@master linux-amd64]# helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "harbor" chart repository
...Successfully got an update from the "hashicorp" chart repository
...Successfully got an update from the "stable" chart repository
...Successfully got an update from the "nfs-subdir-external-provisioner" chart repository
...Successfully got an update from the "bitnami" chart repository
...Successfully got an update from the "brigade" chart repository
Update Complete. ⎈Happy Helming!⎈
  ```
### 生成索引文件
用于生成的索引文件，为Helm Chart仓库提供了一个包含所有可用 Charts 及其元数据的中央索引，从而使得用户能够搜索、解析依赖、管理版本，并安全高效地与仓库交互。
```bash
[root@master ~]# helm repo index helloworld
[root@master ~]# cat helloworld/index.yaml
apiVersion: v1
entries: {}
generated: "2024-04-23T11:17:42.043648173+08:00"
```
### 删除Chart仓库
```bash
[root@master ~]#  helm repo remove bitnami
"bitnami" has been removed from your repositories
```

这些命令构建了 Helm 的仓库管理基础，允许用户轻松地在 Kubernetes 之上部署、扩展和管理复杂的应用程序集合。

### 结语

通过本篇文章，我们深入探讨了 Helm 的工作机制、安装方法、Chart 创建和管理，以及 Helm Chart 仓库的使用。Helm 以其简洁、直接的方式，成为了管理 Kubernetes 应用程序不可或缺的工具。如果你是 Kubernetes 用户，Helm 绝对值得一试。
