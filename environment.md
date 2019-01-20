# 环境搭建

> （可选） VMWare Workstation 15 Pro

> （系统）Ubuntu 16.04.5 LTS

* 1. [ （可选）安装ssh](#ssh)
	* 1.1. [问题](#)
		* 1.1.1. [ubuntu安装好后 ssh进入出错 access denied](#ubuntusshaccessdenied)
* 2. [ （可选）安装vim](#vim)
* 3. [ 替换国内的源 （阿里的源）](#-1)
* 4. [安装k8s](#k8s)

##  1. <a name='ssh'></a> （可选）安装 ```ssh``` 

<small>*如果你需要使用远程ssh工具使用系统的话*</small>

```bash
sudo apt-get install openssh-server # 安装openssh-server
sudo ps -e |grep ssh # 查看ssh服务是否启动
```

###  1.1. <a name=''></a>问题

####  1.1.1. <a name='ubuntusshaccessdenied'></a> ```ubuntu``` 安装好后 ```ssh``` 进入出错 ```access denied```

修改 ```ssh``` 配置文件：

```bash
vi /etc/ssh/sshd_config
```

将其中的 ```PermitRootLogin without-password``` 注释掉， 然后新增 ```PermitRootLogin yes```。 最后重启一下ssh服务：

```bash
service ssh restart
```

##  2. <a name='vim'></a> （可选）安装 ```vim```

```bash
sudo apt-get install vim
```

##  3. <a name='-1'></a> 替换国内的源 （阿里的源）

```bash
vim /etc/apt/sources.list
```

将配置文件更改为下面的内容：

```
deb http://mirrors.aliyun.com/ubuntu/ xenial main multiverse restricted universe
deb http://mirrors.aliyun.com/ubuntu/ xenial-backports main multiverse restricted universe
deb http://mirrors.aliyun.com/ubuntu/ xenial-proposed main multiverse restricted universe
deb http://mirrors.aliyun.com/ubuntu/ xenial-security main multiverse restricted universe
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates main multiverse restricted universe
deb-src http://mirrors.aliyun.com/ubuntu/ xenial main multiverse restricted universe
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-backports main multiverse restricted universe
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-proposed main multiverse restricted universe
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-security main multiverse restricted universe
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-updates main multiverse restricted universe
```

##  4. <a name='k8s'></a>安装 ```k8s```

> 参考地址： [kubernetes安装（国内环境）- k8s启示录](https://zhuanlan.zhihu.com/p/46341911)

*<small>欢迎关注专栏 [k8s启示录](https://zhuanlan.zhihu.com/kubernetes-docker)</small> , 会不断更新k8s和docker方面的内容。*

至此，我们的基本环境算是搭建完成了。