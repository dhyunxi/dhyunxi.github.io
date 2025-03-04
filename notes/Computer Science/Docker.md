



# 介绍
---

## 什么是Docker
- Docker是一个应用容器引擎，开发者可以打包应用及其依赖到一个可抑制的容器中虚拟化运行，以便隔离进程和资源。
	- LXC，Linux Container：一种内核虚拟化技术，可以实现轻量级的虚拟化。从 `0.7` 版本以后开始去除 `LXC`，转而使用自行开发的 [libcontainer](https://github.com/docker/libcontainer)，从 `1.11` 版本开始，则进一步演进为使用 [runC](https://github.com/opencontainers/runc) 和 [containerd](https://github.com/containerd/containerd)。
	- 沙盒，Sand Box：一种虚拟技术
- Docker的结构
	![[Pasted image 20250304194541.png]]
- Docker和虚拟机
	- 虚拟机采用不同的操作系统
	![[Pasted image 20250304185237.png]]
	- 容器则采用与主机的操作系统
	![[Pasted image 20250304185316.png]]
	- Docker不需要Hypervisor实现硬件资源虚拟化，因此运行更快

- Docker局限性：Docker用于应用程序时是最有用的，但并不包含数据。日志、数据库等通常放在Docker容器外。一个容器的镜像通常都很小，不用和存储大量数据，存储可以通过外部挂载等方式使用，比如：NFS、ipsan、MFS等 ，或者docker命令 ，-v映射磁盘分区。总之，docker只用于计算，存储交给别人。

- Docker流程图
![[Pasted image 20250304190103.png]]

- Docker核心：
	- Namespace：实现Container的进程、网络、消息、文件系统和主机名的隔离
	- Cgroup：实现对资源的配额和调度

- Docker特性：
	- 文件系统隔离：每个进程容器运行在一个完全独立的根文件系统里。
	- 资源隔离：系统资源，像CPU和内存等可以分配到不同的容器中，使用cgroup。
	- 网络隔离：每个进程容器运行在自己的网路空间，虚拟接口和IP地址。
	- 日志记录：Docker将收集到和记录的每个进程容器的标准流（stdout/stderr/stdin），用于实时检索或者批量检索
	- 变更管理：容器文件系统的变更可以提交到新的镜像中，并可重复使用以创建更多的容器。无需使用模板或者手动配置。
	- 交互式shell：Docker可以分配一个虚拟终端并且关联到任何容器的标准输出上，例如运行一个一次性交互shell。

## Docker对象
- 镜像image：一个虚拟机的快照，包含了应用程序及其依赖的库、环境等
- DockerFile：自动化脚本，用来运行镜像
- 数据卷，volume：容器和主机共享的文件夹，用于数据挂载
# 


