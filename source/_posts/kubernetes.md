---
title: Kubernetes 的基本概念
date: 2018-04-11 16:25:53
tags:
 - 工具
---

分布式微服务架构的后台开发调试中免不了和Kubernetes打交道，把基本概念和用法通一通省得被动。  
### 基本概念
**Kubernetes**，完备的分布式系统支撑管理平台。特点有：多租户应用支撑、服务注册和发现机制、内建智能负载均衡器、自我修复调整、服务滚动升级、动态扩容缩容机制、多粒度资源配额管理。  

**Service**，Kubernetes集群架构的核心。一个
Service特征有：唯一的name、唯一的虚拟IP（Cluster IP、Service IP或VIP）和Port、提供某种的服务（Redis、MySQL、WebServer）、映射到提供该服务的一组容器上。Service的服务进程都是通过Socket对外提供服务，服务一般都是TCP Server进程。  
Kubernetes通过Cluster IP + Service Port连接到特定的Service上，透明的负载均衡和故障恢复使得服务具有高可用性。  

**Pod**，服务的管理对象。Pod中包含若干个Container，多个Container运行着服务进程且相互隔离。建立Pod时设置Label，Service通过Label Selector选择具有指定Label的Pod，从而建立Service与Pod的联系。每个Pod中运行着一个特殊的名为Pause的Container，其他运行服务进程的Container称为业务容器。  
业务容器共享Pause的网络栈和Volume挂载卷。通常是将一组密切相关的服务进程放入同一个Pod。  
Label就是一个键值对，比如app: mysql  

**Node**，集群管理中的机器分为一个Master节点和多个Node。Master节点运行着集群管理相关的的进程kube-apiserver、kube-controller-manager和kube-scheduler，用于实现集群上完全自动化的资源管理、Pod调度、弹性伸缩、安全控制、监控纠错。Node节点运行着kubelet、kube-proxy服务进程。  
Master和Node可以是物理主机也可以是虚拟主机，Node的最小运行单元是Pod。kubelet、kube-proxy负责Pod的创建、启动、监控、重启、销毁以及负载均衡。  

**Replication Controller**，RC的定义文件里包含目标Pod的定义、目标Pod的副本数量、目标Pod的Label。Kubernetes根据定义文件创建RC后会自动创建目标Pod，Kubernetes通过Label筛选出Pod并实时监控其状态和数量。完全自动化地依据RC定义文件的Pod模版创建、销毁Pod使其达到预期数量。  

**横向扩容**，Kubernetes的最大优点就是横向扩容，可以平滑Node集群规模。  

### 基本使用
不赘述安装配置过程，要实现集群最少要两个主机，一个Master一个Node。假设已经配置好了，在Kubernetes上玩玩基本的RC、Pod、Service、Container。创建提供MySQL服务的Service为例。  

####创建RC、Pod   
Kubernetes的各种配置文件基本都是yaml，创建RC首先写一个xx-rc.yaml文件。  
**首先创建mysql-rc.yaml文件**  
```yaml
apiVersion: v1
kind: ReplicationController		#表明该文件用于创建RC
metadata:
 name: mysql 		#RC的名称
spec:
 replicas: 1		#期待Pod副本数量为1
 selector:
  app: mysql		#Label Selector的目标Label
 template:		#创建Pod的模版
  metadata:
   labels:
    app: mysql		#Pod的Label
  spec:
   containers:		#Pod中的Container数组，表明要创建哪些工作容器
   - name: mysql		#Pod中的容器名字
     image: mysql		#容器的docker image
     ports:
     - containerPort: 3306		#容器暴露的port
     env:		#容器内部的环境变量
     - name: MYSQL_ROOT_PASSWORD
       value: "123456"
```

**发布RC到Kubernetes集群**  
在Master结点执行命令，Kubernetes会会依据文件创建RC，然后自动创建对应数量的Pod，Pod中创建容器时默认从docker源中拉取mysql官方Image。  
```bash
$ kubectl create -f mysql-rc.yaml
replicationcontroller "mysql" created

$ kubectl get rc
NAME  DESERED  CURRENT  AGE
mysql   1        1       3m

$ kubectl get pods
NAME        READY  STATUS   RESTERTS  AGE
mysql-c95jd  1/1     running   0       9m

$ docker ps | grep mysql
//查看docker容器可以看到除了一个刚刚创建的工作容器，还有一个pause Container，这是Pod的根容器。
```

#### 创建Service
**首先创建mysql-svc.yaml文件** 
```bash
apiVersion: v1
kind: Service 	#表明该文件用于创建Service
metadata:
 name: mysql 	#Service的名字
spec:
 ports:
  - port: 3306 	#Service提供服务的端口号
 selector:
  app: mysql 	#Service对应提供服务的Pod的Label
```

**创建Service**  
创建mysql服务后，服务分配到一个Cluster IP虚地址。Kubernetes集群中的其他Pod可以通过Servie的Cluster IP + Port来访问服务。  
Cluster IP是Kubernetes自动分配的，所以事先不知道部署的Service地址，所以需要一个服务发现机制。这个机制可以让容器根据Service的唯一名字从环境变量中获取对应的Cluster IP和Port，从而进行TCP/IP通信。  
```bash
$ kubectl create -f mysql-svc.yaml
serivce "mysql" created

$ kubectl get svc
NAME  CLUSTER-IP      EXTERNAL-IP   PORT(S)  AGE
mysql 169.169.253.143 <none>        3306/TCP  1m
```
