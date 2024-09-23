**原文链接：https://www.ibm.com/topics**

# Checklist

- [x] What is high-performance computing (HPC)?
- [x] What is SaaS?
- [x] What is cloud computing?
- [x] What is Kubernetes?

* 所有云上提供的服务（云上 HPC 、SaaS 等）都有两个重要优势：**快速部署**和**按需付费**，前者可以减少开发周期，后者则可以减少开发所需的软硬件成本，包括无需支付大型计算机设备的成本和维护成本，以及根据流量的峰值和谷值来扩展或缩减容量。

# 软件即服务

* SaaS 应用程序的特征
	* 云上
	* 用户通过具有互联网连接的设备访问应用程序
	* 多租户架构
	* **由供应商负责应用程序的维护和管理，包括应用软件的软硬件的部署与维护，服务的可用性等等**
	* （可选）提供 API 供用户使用以集成其他 SaaS 服务
> SaaS 供应商负责运营、管理和维护软件及其运行的基础设施。客户只需创建一个帐户，支付费用并开始工作。

# `IaaS` VS `PaaS` VS `SaaS

* 供应商承担的部分： `IaaS` < `PaaS` < `SaaS`
* `IaaS` 供应商仅提供应用开发所需的硬件设施， `PaaS` 供应商提供了完整的应用开发所需的工具链，包括硬件设施、开发环境等，而 `SaaS` 供应商则直接提供完整的应用程序供用户使用。

# 云计算

* `IaaS` 、 `PaaS` 和 `SaaS` 都是常见的云计算服务模式。
* Serverless 并不是指服务不需要服务器，而是指应用程序的开发人员完全将后端基础设施的管理工作交付给云服务供应商，即应用开发人员不需要自己部署后端。 `FaaS` 是无服务计算的一个子集。

# Kubernetes

* Kubernetes 架构主要由控制层组件和管理单个节点的组件组成；
* Kubernetes 集群由节点组成，每个节点代表单个主机，可以是物理机或者虚拟机；节点由 pod 组成，pod 是共享相同计算资源和网络的容器组；pod 也是 Kubenetes 中可扩展性的单位，当 pod 中的容器负载过大时，Kubernetes 会把它 replicate 到集群中的其他节点；
* 集群的控制层的主要组件
	* API 服务器：暴露 Kubernetes API（用于管理集群的 API），是所有命令和查询的入口；
	* etcd：一个开源的分布式键值存储，用于管理配置数据、状态数据和元数据；
	* 调度器：跟踪新建的 pod 并为其选择运行的节点；
	* controller-manager：监控集群状态，与 API 服务器通信以管理资源、pod 或服务端点；
	* cloud-controller-manager：功能类似 controller-manager，区别在于它与云服务提供商通信；
* 工作节点主要组件：
	* Kubelet：负责接收和运行来自主节点的任务；
	* Kube-proxy：安装在集群中的每一个节点上，负责维护主机的网络规则，并监控服务和 pod 的变化；
* Kubernetes 中的重要概念
	* ReplicaSet：为特定工作负载维护的一组**稳定的 pod 实例**；
	* 部署：管理容器的创建和状态，指定集群中应该运行多少 pod 实例，如果一个 pod 运行出错，将会创建一个新的 pod 替代它；
	* kubectl：用于管理集群的命令行工具，直接使用 Kubernetes API 通信；
	* DaemonSets：负责帮助确保集群中的每个节点上创建 pod ；
	* Add-ons：包括集群 DNS 、Web UI （用于管理集群的仪表盘）等附加组件；
	* Service：一个定义了一组逻辑 pod 以及如何访问它们的抽象层，提供一种抽象的方法来平衡 pod 的负载；