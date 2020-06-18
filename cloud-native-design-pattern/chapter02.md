# 云原生如何来进行HTTPS升级

本篇是《云原生架构》系列的第二篇。<br />
<br />在[云原生架构的基石](https://mp.weixin.qq.com/s?__biz=Mzg4MTIwMTkxNQ==&mid=2247484056&idx=1&sn=c9651f70c882194b9d0450572efec2df&chksm=cf68c50cf81f4c1a5df47765e29ff11baf1dcd71d10e7ae024e9c0670472dd607966402a810b&scene=126&sessionid=1592015054&key=15fd7ea6eecdea22c3a6460ccc93bc34d8ef7f40f5bfc32150095344026b4fe3b51e8902095b077cadb592930aa054b757ef674642eb930b2c9c8533b582fb76b932d6b59c25e2d6c711863122d24401&ascene=1&uin=NTkyMjg4NQ%3D%3D&devicetype=Windows+10+x64&version=62090070&lang=zh_CN&exportkey=AfwI%2BFQAGmknUybysIePafg%3D&pass_ticket=9LGjoHwPVAOJ5dVM5LZFNpMBM8b2FdQnTXzsmtxpG%2FM%3D)中我们知道云原生的核心是容器。容器的出现打破了原有的软件部署方式，除了容器带来的资源隔离和环境一致性。使得软件的部署变得更加方便和易于操作。除此之外就是容器带来的一些额外的特性，而这些就是云原生架构的重要依据。今天我们就讲一下基于容器资源共享产生的一种云原生架构组件--sidecar。
<a name="C9Oqg"></a>
# 容器的资源共享
容器对于资源隔离的核心技术来源于namespace。namespace用于不同进程组的资源隔离，其目的是将某个特定的全局系统资源通过抽象方法使得namespace 中的进程看起来拥有它们自己的隔离的全局系统资源。<br />


| **namespace** | **被隔离的全局系统资源** | **在容器语境下的隔离效果** |
| :--- | :--- | :--- |
| **Mount ** | 文档系统挂接点 | 每个容器能看到不同的文档系统层次结构 |
| **UTS** | nodename 和 domainname | 每个容器可以有自己的 hostname 和 domainame |
| **PID ** | 进程 ID 数字空间 （process ID number space） | 每个 PID namespace 中的进程可以有其独立的 PID； 每个容器可以有其 PID 为 1 的root 进程；也使得容器可以在不同的 host 之间迁移，因为 namespace 中的进程 ID 和 host 无关了。这也使得容器中的每个进程有两个PID：容器中的 PID 和 host 上的 PID。 |
| **IPC ** | 特定的进程间通信资源，包括[System V IPC](http://www.kernel.org/doc/man-pages/online/pages/man7/svipc.7.html) 和 [POSIX message queues](http://www.kernel.org/doc/man-pages/online/pages/man7/mq_overview.7.html) | 每个容器拥其自己的 System V IPC 和 POSIX 消息队列文档系统，因此，只有在同一个 IPC namespace 的进程之间才能互相通信 |
| **Network ** | 网络相关的系统资源 | 每个容器用有其独立的网络设备，IP 地址，IP 路由表，/proc/net 目录，端口号等等。这也使得一个 host 上多个容器内的同一个应用都绑定到各自容器的 80 端口上。 |
| **User ** | 用户和组 ID 空间 | 在 user namespace 中的进程的用户和组 ID 可以和在 host 上不同； 每个 container 可以有不同的 user 和 group id；一个 host 上的非特权用户可以成为 user namespace 中的特权用户； |


<br />而namespace是可以共享的，当进程namespace共享被激活，一个容器的进程可以在其他容器中看到。这样就为云原生设计模式提供了一种新的方向--sidecar模式。
<a name="BcId5"></a>
# Sidecar
Sidecar模式是指通过对两个容器间的资源共享，来为原有的应用容器提供额外的功能。比如题目中的场景，我们之前的服务都是HTTP访问的，现在由于安全等原因，需要切换到HTTPS接口。那么传统的方式除了安装证书外，可能还涉及到原有服务代码的更改，比如源路径的重定向和链接修改等。这时如果我们的代码这种修改很多或者代码已经运行很长时间了，有些代码难以修改的话，那么升级到HTTPS就可能面临着很大的难题。<br />
<br />那么Sidecar模式如怎么做的呢？那就是在原有的应用容器旁，启动一个Sidecar容器来负责HTTPS请求的转发。为了能够访问到原有的应用容器，应用容器和Sidecar容器需要共享Network Namespace。具体设计如下：<br />![image.png](https://cdn.nlark.com/yuque/0/2020/png/783466/1592191121543-1ca2f7f0-96e6-448c-81aa-f4651728cd21.png#align=left&display=inline&height=405&margin=%5Bobject%20Object%5D&name=image.png&originHeight=405&originWidth=949&size=18269&status=done&style=none&width=949)<br />而对于这个Sidecar容器来说，里面就是一个HTTPS代理服务，通过将外部的HTTPS请求转发到应用服务器的HTTP接口上，达到整个系统的HTTPS升级。<br />
<br />这样做的好处有几个：

- **我们可以选用任何语言和技术来实现**：特别是在原应用代码修改较为麻烦的情况下，因为Sidecar和原有应有服务器完全解耦，所以我们可以选用任何语言和技术来实现我们的HTTPS代理服务。
- **复用**：有些人可能觉得直接实现一个HTTPS代理服务并不难，可以完全不用容器。而使用容器这种Sidecar模式的好处就在于复用性。容器支持环境变量或者配置文件的方式来修改服务的功能，我们可以以此在将这个服务应用于别的应用服务器时，只需要少量的修改便可以达成目的。
<a name="TnG5C"></a>
# 应用场景
除了像上述的HTTPS场景，Sidecar模式主要适用于在不修改原有应用服务器业务逻辑的情况下，为原有应用服务器增加其他功能或者修改行为等，比如：

- **动态加载配置**：Sidecar和原有应用服务器共享Mount Namespace（即存储），Sidecar容器部署一个配置管理程序，接受外部的配置输入，这样当配置发生改动后，自动更改原有应有服务器的配置。如果同时共享了PID Namespace，可以在配置发生改动后，向应用服务器发送终止命令（比如SIGKILL），促使应用服务器重启然后加载新的配置。
- **健康检查**：Sidecar模式可以部署一个健康检查服务，来监控原有的应用服务的健康性。
- **日志管理**：Sidecar和原有的应用共享日志存储，然后Sidecar容器部署一套日志管理服务来提供高级的日志管理功能，比如统计，筛选等。


<br />当然Sidecar模式还有很多其他的应用，我们可以在更改系统功能或者拆分系统的时候将一些通用组件设置为Sidecar模式，来帮助我们的应用更关注我们内部的业务逻辑。
<a name="jciaa"></a>
# Sevice Mesh
Sidecar模式的集大成者就是Service Mesh。即服务网格。服务网格是近年来随着云原生发展提出了一种非常热门的模式，主要是为了解决微服务之间的通信问题。这里以Istio（Sevice Mesh的一种实现）为例：<br />![](https://cdn.nlark.com/yuque/0/2020/svg/783466/1592200908683-3506a539-e5d8-4e47-96f3-1e2ea710b0c7.svg#align=left&display=inline&height=458&margin=%5Bobject%20Object%5D&originHeight=150&originWidth=215&size=0&status=done&style=none&width=656)<br />从上面的设计图可以看出，每一个服务容器旁都设置了一个Proxy Sidecar容器，来负责对流入和流出流量的过滤，并提供了微服务之间的注册和发现。通过对输入和输出流量的代理，可以改变整个系统中流量的流向。而原本的服务容器不需要关心这些事情。<br />
<br />我们可以看出Sidecar模式的设计减少了对原有系统的侵入。让原有系统可以更关注于自身逻辑，而其他功能由Sidecar容器来负责。如果了解Kubernetes的朋友，可以很明显的看出Kubernetes的基本单位Pod的设计和Sidecar如出一辙。目的都是让每个容器的职责明确，更关注于自身的业务。
