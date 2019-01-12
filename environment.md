# 环境准备

> （可选） VMWare Workstation 15 Pro

> （系统）Ubuntu 16.04.5 LTS

* 1. [1. （可选）安装ssh](#ssh)
	* 1.1. [问题](#)
* 2. [2. （可选）安装vim](#vim)
* 3. [3. 替换国内的源 （阿里的源）](#-1)

##  1. <a name='ssh'></a>1. （可选）安装ssh 

<small>*如果你需要使用远程ssh工具使用系统的话*</small>

```bash
sudo apt-get install openssh-server # 安装openssh-server
sudo ps -e |grep ssh # 查看ssh服务是否启动
```

###  1.1. <a name=''></a>问题

**ubuntu安装好后 ssh进入出错 access denied**

修改ssh配置文件：

```bash
vi /etc/ssh/sshd_config
```

将其中的 *PermitRootLogin without-password* 注释掉， 然后新增 *PermitRootLogin yes*。 最后重启一下ssh服务：

```bash
service ssh restart
```

##  2. <a name='vim'></a>2. （可选）安装vim

```bash
sudo apt-get install vim
```

##  3. <a name='-1'></a>3. 替换国内的源 （阿里的源）

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