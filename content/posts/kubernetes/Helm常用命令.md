---
title: "Helm常用命令"
date: 2024-04-24T18:08:31+08:00
tags: ["Helm"]
comment: true
categories: ["Kubernetes"]
---
## 引言
Helm 是 Kubernetes 的包管理器，它通过提供一种称为 Helm Chart 的方式来简化 Kubernetes 应用的配置、部署和管理。本文将详细介绍 Helm 的常用命令及其使用场景，帮助用户更高效地利用 Helm 进行 Kubernetes 应用的生命周期管理。

## 常用的 Helm 命令
### helm create
该命令用于初始化一个新的 Helm Chart。Helm Chart 是 Kubernetes 应用的包，包含了运行应用所需的所有资源定义。

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
```

### helm install
用于将 Helm Chart 安装到 Kubernetes 集群中。它会创建所有定义在 Chart 中的资源。

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

### helm upgrade
用于升级已安装的 Helm Chart 到新版本。可以修改 Chart 的配置和资源。

```bash
# 将helloworld的副本调整为2个副本
[root@master ~]# cat helloworld/values.yaml |head
# Default values for helloworld.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

replicaCount: 2

image:
  repository: nginx
  pullPolicy: IfNotPresent
  # Overrides the image tag whose default is the chart appVersion.
...

# 将已有的 release 升级到一个新版本的 chart
[root@master ~]# helm upgrade myhelloworld helloworld
Release "myhelloworld" has been upgraded. Happy Helming!
NAME: myhelloworld
LAST DEPLOYED: Tue Apr 23 13:56:52 2024
NAMESPACE: default
STATUS: deployed
REVISION: 2
NOTES:
1. Get the application URL by running these commands:
  export NODE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].nodePort}" services myhelloworld)
  export NODE_IP=$(kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[0].address}")
  echo http://$NODE_IP:$NODE_PORT
```

### helm rollback
当 Helm Chart 升级后出现问题时，可以使用该命令回滚到之前的版本。

```bash
[root@master ~]# helm rollback myhelloworld 1
Rollback was a success! Happy Helming!
[root@master ~]# kubectl get pod -l app.kubernetes.io/name=helloworld
NAME                            READY   STATUS    RESTARTS   AGE
myhelloworld-768888cf49-xd64t   1/1     Running   0          3h54m

```
### helm install --debug --dry-run
在实际安装之前，可以使用该命令查看 Helm 将如何安装 Chart。

```bash
[root@master ~]# helm install myhelloworld --debug --dry-run helloworld
install.go:214: [debug] Original chart version: ""
install.go:231: [debug] CHART PATH: /root/helloworld

NAME: myhelloworld
LAST DEPLOYED: Tue Apr 23 14:02:46 2024
NAMESPACE: default
STATUS: pending-install
REVISION: 1
USER-SUPPLIED VALUES:
{}
...
```
### helm template
该命令用于渲染 Helm Chart 的模板，但不执行安装操作。

```bash
[root@master ~]# helm template helloworld

# 执行完毕后，会在helloworld目录下生成templates目录
[root@master helloworld]# tree templates/
templates/
├── deployment.yaml
├── _helpers.tpl
├── hpa.yaml
├── ingress.yaml
├── NOTES.txt
├── serviceaccount.yaml
├── service.yaml
└── tests
    └── test-connection.yaml
```

### helm lint
对 Helm Chart 进行静态分析，检查可能存在的问题。

```bash
[root@master ~]# helm lint helloworld
==> Linting helloworld
[INFO] Chart.yaml: icon is recommended

1 chart(s) linted, 0 chart(s) failed
# 对1个chart进行了linting（代码审查），并且在审查过程中没有发现任何失败的问题
```
### helm show
用于查看 Helm Chart 的内容，包括模板、配置文件和其他资源的定义。
```bash
# 显示Chart的信息和元数据
[root@master ~]# helm show chart helloworld
apiVersion: v2
appVersion: 1.16.0
dependencies:
- name: mysql
  repository: https://charts.helm.sh/stable
  version: 1.6.1
description: A Helm chart for Kubernetes
name: helloworld
type: application
version: 0.1.0

# 查看 Helm Chart的所有内容，包括模板、配置文件和其他资源的定义
[root@master ~]# helm show all helloworld

# 显示 Helm Chart的 values.yaml 文件内容，这个文件包含了图表的默认配置值
[root@master ~]# helm show values helloworld
```
### helm pull
从远程仓库下载或拉取一个 Helm Chart 到本地。

```bash
[root@master ~]# helm repo add stable https://charts.helm.sh/stable
[root@master ~]# helm pull stable/mysql --version 1.6.9
[root@master ~]# helm env
HELM_BIN="helm"
HELM_BURST_LIMIT="100"
HELM_CACHE_HOME="/root/.cache/helm"
HELM_CONFIG_HOME="/root/.config/helm"
HELM_DATA_HOME="/root/.local/share/helm"
...
[root@master ~]# ls /root/.cache/helm/repository/|grep mysql
mysql-1.6.9.tgz
```
**常用参数**：
* `--version`：允许指定一个版本约束，用于下载特定版本的 Helm 图表。默认使用latest版本；
* `--untar`：下载Chart后是否自动解压缩。如果设置为true，则Helm会在下载后自动解压缩
* `--untardir`：指定图表提取到的目录；
* `--repo`：指定从中下载图表的存储库的 URL；
* `--username`和`--password`：提供访问私人存储库的凭证；
* `--verify`：可以在使用图表之前对其进行验证，确保图表的完整性和安全性。
```bash
# 拉取Package到本地
[root@master ~]# helm pull stable/nginx-ingress --version 1.2.3 --untar
[root@master ~]# tree nginx-ingress/
nginx-ingress/
├── Chart.yaml
├── ci
│   └── psp-values.yaml
├── OWNERS
├── README.md
├── templates
│   ├── clusterrolebinding.yaml
│   ├── clusterrole.yaml
...
```
### helm dependency
管理 chart 的依赖关系。

修改helloworld目录中Chart.yaml文件，文件末尾追加依赖关系，例如：
```yaml
[root@master ~]# cat helloworld/Chart.yaml |tail
...
appVersion: "1.16.0"
dependencies:
  - name: mysql
    version: 1.6.1
    repository: https://charts.helm.sh/stable

# 列出当前helloworld chart的依赖项
[root@master ~]# helm dependency list helloworld
load.go:120: Warning: Dependencies are handled in Chart.yaml since apiVersion "v2". We recommend migrating dependencies to Chart.yaml.
NAME 	VERSION	REPOSITORY                   	STATUS
mysql	1.6.1  	https://charts.helm.sh/stable	missing

[root@master ~]# helm dependency update helloworld
```

### helm uninstall
移除已安装的 Helm Chart。

```bash
[root@master ~]# helm uninstall myhelloworld
release "myhelloworld" uninstalled
```

## 命令速查表
为了便于快速查阅，Helm 提供了一个[命令速查表](https://helm.sh/zh/docs/intro/cheatsheet/)。

## 结语
本文详细介绍了 Helm 的常用命令，旨在帮助用户更深入地理解 Helm 的使用和 Kubernetes 应用的管理。通过掌握这些命令，用户可以更高效地进行 Kubernetes 应用的部署、升级和维护。同时，本文也提供了一些最佳实践和技巧，以提高用户的工作效率。

