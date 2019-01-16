# Kubeflow

Kubeflow 是为了方便在kubernetes上部署机器学习工作流的项目。提供了很多有用的工具，方便运行机器学习项目。


## 安装

### 安装ksonnet

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

### 安装kubeflow

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

这样基本就完成了。