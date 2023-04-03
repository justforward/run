# k8s 基础组件和作用

1）kubctl
2）api-server
3）kube-scheduler ：kube-scheduler 负责监视新创建、未指定运行Node的 Pods，决策出一个让pod运行的节点。
4）kubelet：
5）control-manager：k8s在后台运行许多不同的控制器进程，当服务配置发生更改时（例如，替换运行 pod 的镜像，或更改配置 yaml 文件中的参数），控制器会发现更改并开始朝着新的期望状态工作

**从kubectl开始，我们来看一下K8s的基本工作流程：**

0.  kubectl 客户端首先将CLI命令转化为RESTful的API调用，然后发送到kube-apiserver。
1.  kube-apiserver 在验证这些 API 调用后，将任务元信息并存储到etcd，接着调用 kube-scheduler 开始决策一个用于作业的Node节点。
2.  一旦 kube-scheduler 返回一个适合调度的目标节点后，kube-apiserver 就把任务的节点信息存入etcd，并创建任务。
3.  此时目标节点中的 kubelet正监听apiserver，当监听到有新任务需要调度到本节点后，kubelet通过本地runtime创建任务容器，执行作业。
4.  接着kubelet将任务状态等信息返回给apiserver存储到etcd。
5.  这样我们的任务已经在运行了，此时control-manager发挥作用保证任务一直是我们期望的状态。



# k8s控制器

无状态的服务
-  deployment
deployment 作用于一组pods的创建和运行
- replicaset

有状态的服务：
- statfulset
1）不仅能管理 Pod 的对象，还它能够保证这些 Pod 的顺序性和唯一性。
2）StatefulSet 会为每个 Pod 设置一个单独的持久标识符，这些用于标识序列的标识符在发生调度时也不会丢失。
3）StatefulSet 引入了 PV 和 PVC 对象来持久存储服务产生的状态。

service： 一个网络下的pod集合，service 使用labels标签来选择代理的pod。
1）clusterIP
 ClusterIP 是 Service 的默认类型，它会为 Service 分配一个 Cluster IP，用于在集群内部访问该 Service。集群的pod访问service
2）NodePort
通过每个节点上的 IP 和静态端口（NodePort）暴露服务。NodePort 服务会路由到 ClusterIP 服务
。用于外部流量访问service
3） LoadBalancer
LoadBalancer 会在集群外部创建一个负载均衡器，并将该负载均衡器的 IP 地址映射到 Service 的 Cluster IP 和端口上。这样，可以通过负载均衡器的 IP 地址和端口号来访问 Service。
4）`ExternalName
通过返回 CNAME 和它的值，可以将服务映射到 externalName 字段的内容（例如， foo.bar.example.com）。没有任何类型代理被创建，这只有 Kubernetes 1.7 或更高版本的 kube-dns 才支持。

# k8s的存储模型

面试官向我问及如果在一个具有很大磁盘的物理机上部署多个容器应用，其存储空间是否应保持共享。

1）emptydir--临时存储
	pod启动时，就会在pod所在节点的磁盘空间开辟出一块空卷，最开始里面是什么都没有的，pod启动后容器产生的数据会存放到那个空卷中。空卷变成了一个临时卷。
	但是作为一个临时存储会随着pod容器消失而销毁。
2） 本地储存实现的持久化存储——hostPath

k8s的hostPath存储方式，是一种映射本地的文件或目录进入pod中的方式，这种方式与docker的Data volume存储方式非常相似。
3）k8s持久化存储

PV和PVC

PV描述的是持久化存储数据卷，这个API对象主要定义的是一个持久化存储在宿主机上的目录，比如一个NFS的挂载目录。

PVC对象通常由平台的用户创建，或者以PVC模板的方式成为StatefulSet的一部分，然后由StatefulSet控制器负责创建带编号的PVC。

用户创建的PVC要真正被容器使用，就必须先和某个符合条件的PV进行绑定，此时必须要满足以下两个条件：

-   PV的大小必须满足PVC的要求，即PV和PVC的spec字段
-   PV和PVC的storageClassName必须一致
在对PV和PVC进行绑定之后，Pod就能像使用hostPath一样使用这个PVC了。

为什么要PVC？
1）解耦，通过中间层解耦PV和pod，PV的修改不会影响到pod的使用。
2）水平扩展

PVC 
1) rwo  针对的是节点
2) 
3) rwx 


# 网络

一个节点（Node）是指一个物理机器或虚拟机


1）同一个pod上的不同容器是如何通信的？共享命名空间的
2）在同一个节点上的不同pod如何通信？可以基于 网桥的方式，比如docker容器是可以通过
3） 不同节点的上pod通信？通过CNI（网络插件接口）方式，

CNI 的工作原理是，每个 Pod 都会有一个虚拟网卡，该虚拟网卡通过一个名为 CNI 插件的二进制文件来配置，该插件负责将容器和宿主机的网络连接起来。这个插件需要在 Kubernetes 的每个节点上运行，并且在每个 Pod 中启动的容器都需要通过该插件进行网络配置。


为每个pod分配一个虚拟网卡，

1.  **Flannel**：基于VXLAN协议（网络隧道技术）实现的覆盖网络方案，每个节点都会分配一个子网，并分配给Pod。Flannel具有高性能、可扩展、易于部署的特点。
	核心：它是一个虚拟的二层网络，在不同主机之间创建一个虚拟的二层网络，并分配一个唯一的子网给容器使用
    
2.  **Calico**：基于BGP协议实现的覆盖网络方案，每个节点都会分配一个子网，并分配给Pod。Calico具有高度的网络安全性、易于扩展的特点。



Flannel实现原理



Flannel使用了虚拟二层网络技术，通过在每个节点上创建一个虚拟的二层网络，并为每个Pod分配一个唯一的IP地址，从而实现Pod之间的跨节点通信。

在Flannel中，每个节点都有一个唯一的Subnet，Subnet是一个CIDR（CIDR（Classless Inter-Domain Routing，无分类域间路由）地址段是一种用于指定IP地址范围的标准化方法）地址段，它被用来为该节点上的Pod分配IP地址。例如，节点A的Subnet为10.244.0.0/16，节点B的Subnet为10.244.1.0/16，那么节点A上的Pod将会被分配10.244.0.0/24的IP地址段，节点B上的Pod将会被分配10.244.1.0/24的IP地址段。

当一个Pod在节点A上启动时，Flannel会为它分配一个本地IP地址，例如10.244.0.2，并在该节点上创建一个虚拟接口flannel.1，并将该接口的MAC地址设置为flannel.1接口的MTU和IP地址，从而模拟一个物理接口的行为。同时，Flannel会为该Pod创建一个路由规则，将目标地址为10.244.1.0/24的数据包发送到节点B上。

在节点B上，Flannel会为该Pod创建一个相同的虚拟接口flannel.1，并将它的MAC地址设置为与节点A上的flannel.1接口相同。当节点B收到从节点A发送来的数据包时，它会将数据包从flannel.1接口转发到目标Pod的网络命名空间中，并将目标MAC地址设置为该Pod的MAC地址。


Flannel使用了虚拟二层网络技术，通过agent在每个节点上创建一个虚拟的二层网络，并为每个pod分配一个唯一的IP地址，从而实现pod之间的跨节点通信。


Flannel的源码实现分为两部分：Flannel Agent和Flannel Backend。

Flannel Agent是运行在每个节点上的进程，它负责将虚拟网络接口flannel.1创建和维护，并且在Pod启动和停止时为它们分配和释放IP地址。

Flannel Backend是一个键值存储系统，用于保存节点和Subnet的映射关系，并为每个节点分配唯一的Subnet。常见的Flannel Backend有etcd、Consul等。


VXLAN 在 VTEP 间建立隧道，通过 Layer 3 网络传输封装后的 Layer 2 数据。


图中 Host-A 和 Host-B 位于 VNI 10 的 VXLAN，通过 VTEP-1 和 VTEP-2 之间建立的 VXLAN 隧道通信。  
数据传输过程如下：


