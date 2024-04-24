---
title: "smem 排查内存泄露工具"
date: 2023-10-23T10:57:28+08:00
tags: ["Linux"]
categories: ["Liunx"]
---
这次用下新工具smem，这是一个python写的小工具，可以统计系统中所有进程占用的物理内存RSS、以及去掉共享内存的PSS、以及程序本身的独占内存USS的情况。
```bash
# centos 下
yum install epel-release
yum install smem python-matplotlib python-tk

# ubuntu 下
apt-get install smem
```

常用命令：

* -k 带单位显示内存
```bash
[root@k8s haifeng18]# smem -k
  PID User     Command                         Swap      USS      PSS      RSS
270267 101      /usr/bin/dumb-init -- /ngin        0    64.0K    68.0K    72.0K
1372033 root     runsv bird                         0    72.0K   168.0K     1.2M
1372036 root     runsv allocate-tunnel-addrs        0    68.0K   172.0K     1.3M
1372039 root     runsv confd                        0    72.0K   173.0K     1.3M
1372040 root     runsv bird6                        0    72.0K   176.0K     1.3M
1372037 root     runsv felix                        0    72.0K   179.0K     1.3M
1372038 root     runsv cni                          0    72.0K   179.0K     1.3M
1371662 root     /usr/local/bin/runsvdir -P         0   100.0K   188.0K     1.2M
```


* -u -k 带单位显示每个用户的内存占用：
```bash
[root@k8s haifeng18]# smem -u -k
User     Count     Swap      USS      PSS      RSS
65535        3        0   120.0K   695.0K     1.8M
lldpd        1        0   540.0K     1.1M     2.9M
nobody       1        0     1.1M     1.3M     5.1M
rpc          1        0     1.3M     1.7M     5.8M
dbus         2        0     1.6M     2.0M     7.5M
haifeng18     2        0     2.0M     4.1M    11.1M
```

* -w -k 显示系统整体内存情况类似free
```bash
[root@k8s haifeng18]# smem -w -k
Area                           Used      Cache   Noncache
firmware/hardware                 0          0          0
kernel image                      0          0          0
kernel dynamic memory         17.7G      14.8G       2.9G
userspace memory               7.5G       1.5G       6.0G
free memory                    5.9G       5.9G          0
```

* -k -s uss -r 按照uss的占用从大到小排序的方式展示内存的占用情况 非常实用
```bash
[root@k8s haifeng18]# smem  -k -s uss -r
  PID User     Command                         Swap      USS      PSS      RSS
1216764 root     /usr/bin/kube-apiserver --v        0     4.7G     4.7G     4.7G
 1097 root     /data0/prometheus/prometheu        0   642.0M   642.0M   642.0M
  898 root     /usr/bin/etcd --config-file        0   203.1M   203.1M   203.1M
 1095 root     /usr/share/grafana/bin/graf        0   195.0M   195.1M   196.9M
 1037 root     /usr/bin/consul agent -conf        0   138.7M   138.7M   138.7M
```

* **USS(User State Size)**：用户状态大小或者用户状态占用大小，是指一个进程实际占用的物理内存的大小，仅包括用户态的部分
* **PSS(Page Status Size)**：页面状态大小或者页面状态占用大小，是指一个进程实际占用的物理内存的大小，包括数据和指令。PSS不仅包括用户态状态，还包括内核态状态
* **RSS(Real Memory Size)**：实际内存使用量或者实际内存占用大小，是指一个进程实际占用的物理内存的大小，包括数据、代码、堆栈等各种类型的内存占用。

好了基本命令介绍完毕，那我们来看看如何查看内存是否泄漏吧，因为内存泄漏的程序占用的内存是一直再增加的（这不是废话嘛），这样我们就可以用上面的排序命令只观察上面几个进程了。
```bash
[root@k8s haifeng18]# watch smem  -k -s uss -r

Every 2.0s: smem -k -s uss -r                                                                                                                                                                         k8s.wq.2.20.30: Tue Sep 12 16:35:49 2023

  PID User     Command                         Swap	 USS	  PSS	   RSS
1216764 root     /usr/bin/kube-apiserver --v        0     4.7G     4.7G     4.7G
 1097 root     /data0/prometheus/prometheu        0   642.0M   642.0M   642.0M
  898 root     /usr/bin/etcd --config-file        0   203.1M   203.1M   203.1M
 1095 root     /usr/share/grafana/bin/graf        0   195.0M   195.1M   196.9M
 1037 root     /usr/bin/consul agent -conf        0   138.7M   138.7M   138.7M
```
小技巧，watch加在命令前面，5s执行一次命令，会高亮显示改变的部分。

