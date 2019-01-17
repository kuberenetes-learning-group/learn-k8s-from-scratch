# Kubeflow

Kubeflow 是为了方便在kubernetes上部署机器学习工作流的项目。提供了很多有用的工具，方便运行机器学习项目。


* 1. [安装](#)
	* 1.1. [安装ksonnet](#ksonnet)
	* 1.2. [安装kubeflow](#kubeflow)
	* 1.3. [问题](#-1)


##  1. <a name=''></a>安装

###  1.1. <a name='ksonnet'></a>安装ksonnet

由于Kubeflow支持使用ksonnet编译生成，所以我们需要首先安装ksonnet。

1 下载ksonnet

```bash
wget -L https://github.com/ksonnet/ksonnet/releases/download/v0.13.1/ks_0.13.1_linux_amd64.tar.gz # 最新版本可以到 https://github.com/ksonnet/ksonnet/releases 查看
```

2 解压缩

```bash
tar -zxf ks_0.13.1_linux_amd64.tar.gz
```

3 将ksonnet添加到 $PATH 中，

```bash
vim ~/.bashrc
```

在最后一行添加 

```
export PATH=$PATH:/root/ks_0.13.1_linux_amd64 # 之前的解压目录
```

然后让配置生效

```bash
source ~/.bashrc
```

这时运行 ks 命令，可以发现ksonnet已经安装成功了。

###  1.2. <a name='kubeflow'></a>安装kubeflow

1. 下载 kfctl.sh

```bash
mkdir kubeflow
cd kubeflow
export KUBEFLOW_TAG=v0.3.5 # 这个是你想安装的版本

curl https://raw.githubusercontent.com/kubeflow/kubeflow/${KUBEFLOW_TAG}/scripts/download.sh | bash
```

2. 部署并启动kubeflow

```bash
scripts/kfctl.sh init config --platform none # kubeflow-config 是你想放置配置文件的目录，可以替换成任何你想要的
cd config
../scripts/kfctl.sh generate k8s
../scripts/kfctl.sh apply k8s
```
3. 更换被墙的镜像

运行下面的命令：

```bash
 kubectl -n kubeflow get pod
```

你会发现有些 pod 出现了 ImagePullBackOff 的错误，这是因为它们所用的镜像是gcr.io下的，所以无法下载，这时，我们需要更换镜像从国内镜像网站下载：

可以参照 [Docker镜像获取（gcr.io等）](https://zhuanlan.zhihu.com/p/54477544) 来进行安装，这里以一个镜像来举例：

使用上面的镜像我们可以看到

```
tf-hub-0    0/1     ImagePullBackOff    0          5m35s
```

所以我们首先看一下event：

```bash
kubectl -n kubeflow describe pod tf-hub-0
```

然后可以看到pod的event是：

```
Events:
  Type     Reason     Age                  From               Message
  ----     ------     ----                 ----               -------
  Normal   Scheduled  4m35s                default-scheduler  Successfully assigned kubeflow/tf-hub-0 to promote
  Warning  Failed     38s                  kubelet, promote   Failed to pull image "gcr.io/kubeflow/jupyterhub-k8s:v20180531-3bb991b1": rpc error: code = Unknown desc = Error response from daemon: Get https://gcr.io/v2/: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
  Warning  Failed     38s                  kubelet, promote   Error: ErrImagePull
  Normal   BackOff    37s                  kubelet, promote   Back-off pulling image "gcr.io/kubeflow/jupyterhub-k8s:v20180531-3bb991b1"
  Warning  Failed     37s                  kubelet, promote   Error: ImagePullBackOff
  Normal   Pulling    23s (x2 over 4m33s)  kubelet, promote   pulling image "gcr.io/kubeflow/jupyterhub-k8s:v20180531-3bb991b1"
```

所以我们只需要下载 gcr.io/kubeflow/jupyterhub-k8s:v20180531-3bb991b1 这个镜像就可以了，下面我们从阿里云进行下载：

```
docker pull registry.cn-hangzhou.aliyuncs.com/kubeflow/jupyterhub-k8s:v20180531-3bb991b1

docker tag registry.cn-hangzhou.aliyuncs.com/kubeflow/jupyterhub-k8s:v20180531-3bb991b1 gcr.io/kubeflow/jupyterhub-k8s:v20180531-3bb991b1

docker rmi registry.cn-hangzhou.aliyuncs.com/kubeflow/jupyterhub-k8s:v20180531-3bb991b1
```

有些pod按照这样更改后发现pod状态没有改变，这是因为deployment中设置的pod镜像拉取策略是Always，所以需要手动恢复成默认的IfNotPresent。这里就不列举了。

然后再检查状态可以发现变成 Running了：

```
tf-hub-0    1/1     Running             0          19m
```

###  1.3. <a name='-1'></a>问题

1. 安装时，apply k8s 有时会卡在 *ks apply default -c ambassador*，然后报 connection refused 的错误：

这个很有可能是因为你的内存不足等原因，导致 k8s api-server 被关闭，需要释放一些内存或者升级内存，然后重启k8s。