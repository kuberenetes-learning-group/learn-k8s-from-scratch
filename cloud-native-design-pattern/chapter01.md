# 云原生架构的基石

时至今日，如果要问后端最为热门的词语是什么？那么云原生至少会出现在许多人的答案中。从Docker到Kubernetes，再到目前火热的CNCF基金会，云原生技术已经越发成熟和强大。<br />
<br />![landscape.png](https://cdn.nlark.com/yuque/0/2020/png/783466/1591956506953-87aaf122-0693-4b77-8b87-0677c38ccdfe.png#align=left&display=inline&height=4380&margin=%5Bobject%20Object%5D&name=landscape.png&originHeight=4380&originWidth=7660&size=6358733&status=done&style=none&width=7660)<br />
<br />随着云原生的不断发展，后端服务正在转向 "**面向云原生架构**" ,而云原生的架构思路就来自于云原生的基石：**容器技术**。所以在我们继续之后的云原生架构设计之前，我们来首先了解云原生和容器技术以及为什么容器的设计对云原生架构有这么大的影响。
<a name="0OtmH"></a>
# 云原生
在CNCF的官网中，对云原生的定义是：
> Cloud native technologies empower organizations to build and run scalable applications in modern, dynamic environments such as public, private, and hybrid clouds. Containers, service meshes, microservices, immutable infrastructure, and declarative APIs exemplify this approach.
> 

> These techniques enable loosely coupled systems that are resilient, manageable, and observable. Combined with robust automation, they allow engineers to make high-impact changes frequently and predictably with minimal toil.


<br />翻译过来就是云原生技术能够使组织在现代化的、动态的环境下（比如公有云、私有云、混合云）构建和运行可扩展应用程序。 容器，服务网格，微服务，坚固的基础服务和声明性API就是这种方法的例证。<br />
<br />云原生技术能够将耦合的系统进行解耦，使得其具有弹性，易于管理和可以观测。通过结合强大的自动化功能，让工程师能够以最小的工作量来按照预期的持续更新一些重大的改动。

从云原生的定义可以看出云原生的目标就是给原有的开发进行赋能，帮助以前的开发来适应云端环境。并且给予原有的应用更为强大的功能。
<a name="Mssfj"></a>
# 容器
那么云原生技术是如何做到上面说的这些呢？这就要从容器化技术说起。容器化技术来源于虚拟化技术，相比于虚拟化技术的系统资源占用严重的问题，容器化技术在一开始就设计成在共用内核上进行隔离。<br />
<br />![](https://cdn.nlark.com/yuque/0/2020/webp/783466/1591957542803-071d1ed8-a121-4332-abbe-b3e7c3a83e2f.webp#align=left&display=inline&height=261&margin=%5Bobject%20Object%5D&originHeight=261&originWidth=692&size=0&status=done&style=none&width=692)<br />
<br />![](https://cdn.nlark.com/yuque/0/2020/webp/783466/1591957471723-d82124bd-d5d9-4b77-a4ee-d59ea22afaaa.webp#align=left&display=inline&height=195&margin=%5Bobject%20Object%5D&originHeight=195&originWidth=689&size=0&status=done&style=none&width=689)<br />
<br />从上面的对比可以看出，Docker（目前最受欢迎的容器化技术）和 Virtual Machines的区别在于减少了庞大的Guest OS占用，通过统一的 Docker Engine来达到虚拟化的目的。<br />
<br />那么重点就是这个Docker Engine，这个是一个轻量级的进程运行时和打包工具，能够打包成可以在Engine上运行的镜像。那么最终当我们需要使用容器来运行我们的程序时，只需要打包成镜像后进行运行后就可以了。<br />
<br />![](https://cdn.nlark.com/yuque/0/2020/webp/783466/1591958125646-756819f8-e702-4dca-a76d-b24c58863bc7.webp#align=left&display=inline&height=658&margin=%5Bobject%20Object%5D&originHeight=658&originWidth=1200&size=0&status=done&style=none&width=1200)<br />

- Docker Client：接收命令和Docker Host进行交互的客户端
- Docker Host：运行Docker服务的主机
   - Docker Daemon：守护进程，用于管理所有镜像和容器
   - Docker Images/Containers：镜像和容器实例
- Registry(Hub)：镜像仓库


<br />那么容器是如何来进行虚拟化的呢？主要是三个技术：namespace, control groups, union FS.<br />
<br />![image.png](https://cdn.nlark.com/yuque/0/2020/png/783466/1591958403400-c4abc9ba-d2c6-4b5a-850d-d431ebb1c0ff.png#align=left&display=inline&height=177&margin=%5Bobject%20Object%5D&name=image.png&originHeight=177&originWidth=686&size=60147&status=done&style=none&width=686)<br />

- **namespace **: Linux提供了七种不同的命名空间，包括 CLONE_NEWCGROUP、CLONE_NEWIPC、CLONE_NEWNET、CLONE_NEWNS、CLONE_NEWPID、CLONE_NEWUSER、CLONE_NEWUTS，分别提供不用资源上的隔离，要记住这一点，因为这些命名空间是可以共享的。通过命名空间的隔离，容器内的进程只能感受到内部的资源，而对宿主机和其他进程一无所知。通过这种方式达到了基本的隔离。
- **control groups（CGroups）**：Linux的CGroup能够为一组进程分配资源，包括CPU，内存，带宽等。
- **Union FS **: Docker中的每一个镜像都是由一系列的只读层组成的，Dockerfile 中的每一个命令都会在已有的只读层上创建一个新的层。通过 docker run 命令可以在镜像的最上层添加一个可写的层 - 容器层，所有对于运行时容器的修改其实都是对这个容器读写层的修改。容器和镜像的区别就在于，所有的镜像都是只读的，而每一个容器其实等于镜像加上一个可读写的层，也就是同一个镜像可以对应多个容器。同时已构建的每一层镜像也可以作为其他镜像的基础层进行共用。


<br />通过这种方式，容器为我们的应用提供了基本的隔离，并且从上面的核心技术比如namespace和CGroups都是基于进程的，所以容器推荐的是单进程部署。由于我们的应用都是单进程的，所以我们的应用可以很方便的以容器化的方式部署。
<a name="ECXDv"></a>
# 云原生架构
那么容器化对云原生的意义在哪里呢？那就是屏蔽了具体的应用，而以一种更为通用的方式来进行编排和设计，那就是容器。现在流行的Kubernetes就是很好的容器编排系统。而容器其实就是一种抽象，对于应用的抽象。通过这种抽象，我们可以来设计如何组合和调度我们的容器，也就产生了云原生的架构思想。<br />
<br />云原生架构其实是一种分布式的架构思想，我们会在之后的文章中进行介绍。而云原生架构的核心理念就是源自容器的核心技术，也是设计思想中的抽象和职责分离等思想的外延。如何来使用容器来更好的实现我们的业务或者拓展我们的业务功能就是云原生架构的核心理念。这些我们将在之后的文章一一讲解。<br />
<br />事实上，每一个开发者都应该了解云原生技术，因为这是一种全新的开发和部署理念。在之后的文章中，我们会讲解目前主流的云原生架构思想。
<a name="uKNgk"></a>
# 参考链接

- CNCF Home Page: [https://github.com/cncf/foundation/blob/master/charter.md](https://github.com/cncf/foundation/blob/master/charter.md)
