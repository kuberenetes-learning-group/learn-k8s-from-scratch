# Arena <small>-- CLI of Kubeflow</small>

在 [再谈Kubeflow](kubeflow-intro.md) 中，我们说到过 `Kubeflow` 还存在很多可以优化的地方。其中就提到过 `Kubeflow` 需要更为方便的使用方式。目前的 `Kubeflow` 仅仅部署了相关的组件，但是对于想要直接开始进行机器学习的人来说并不容易，因为你需要知道自己需要使用哪些组件以及如何使用。而这显然提高了 `Kubeflow` 的门槛。在这种背景下，`Arena` 项目应运而生。

## Arena 是什么

根据 [官方](https://github.com/kubeflow/arena/blob/master/README_cn.md) 的描述：

> Arena 是一个命令行工具，可供数据科学家轻而易举地运行和监控机器学习训练作业，并便捷地检查结果。目前，它支持单机/分布式深度学习模型训练。

也就是说 `Arena` 为数据科学家提供了以下的功能：

* 基于 `Kubeflow` 的命令行工具
* 运行 单击/分布式 的训练任务
* 监控自己的训练任务和查看结果

同时还提供了额外的功能：

* 查看系统的资源和自己的任务占用资源的情况

## 安装 Arena

官方提供了详细的安装步骤，可以直接[查看](https://github.com/kubeflow/arena/blob/master/docs/installation_cn/README.md)。 这里就简单介绍一下：

* 因为使用的 `Kubeflow` 基于 `Kubernetes` 的，所以需要首先安装 `Kubernetes` 。 可以查看 [kubernetes安装（国内环境）- k8s启示录](https://zhuanlan.zhihu.com/p/46341911)。

* `Arena` 使用 `Helm` 来部署相应的组件，所以要安装 `Helm` 和对应的 `tiller`，可以查看 [Installing Helm](https://docs.helm.sh/using_helm/#installing-helm) 和 [INSTALLING TILLER](https://docs.helm.sh/using_helm/#installing-tiller)

* `Helm` 的部署采用 `chart` 的方式， 所以首先下载对应的 `chart`

```bash
mkdir /charts
git clone https://github.com/kubeflow/arena.git
cp -r arena/charts/* /charts
```

* 安装训练任务需要的控制器， 这里以 `tf-operator` 为例。

```bash
kubectl create -f arena/kubernetes-artifacts/jobmon/jobmon-role.yaml
kubectl create -f arena/kubernetes-artifacts/tf-operator/tf-operator.yaml
```

* 现在可以安装 `arena` 了， `arena` 目前采用源码编译的方式， 所以确保 `Go` 版本 >= 1.8,然后下载源码编译。

```bash
mkdir -p $GOPATH/src/github.com/kubeflow
cd $GOPATH/src/github.com/kubeflow
git clone https://github.com/kubeflow/arena.git
cd arena
make
```

* （使用arena命令） 将编译生成的可执行文件加入`$PATH`中。 可执行文件位于 `bin/` 下。

* （可选） 命令自动完成（ubuntu）

```bash
echo "source <(arena completion bash)" >> ~/.bashrc
```

* 如果是国内环境，为了确保arena不会每次跑任务都拉取最新镜像导致失败， 将 `imagePullPolicy` 策略由 `Always 修改为 `IfNotPresent`:

```bash
find /charts/ -name values.yaml| xargs sed -i "s/Always/IfNotPresent/g"
```

* （可选）如果需要查看 `job` 的 `GPU` 资源， 需要进行下面的安装步骤：

```bash
kubectl apply -f kubernetes-artifacts/prometheus/prometheus.yaml # 安装prometheus

kubectl label node <your GPU node> accelerator/nvidia_gpu=true # 非Alibaba Cloud kubernetes

kubectl apply -f kubernetes-artifacts/prometheus/gpu-exporter.yaml # 部署gpu exporter
```

## 使用 `Arena`

安装完 `Arena` 之后，就可以使用 `Arena` 来运行训练任务了。目前 `Arena` 支持很多功能，这里来简单介绍下其中比较重要的功能：

* 从 `git` 中拉取代码
* 支持tensorboard
* 支持分布式任务
* 支持导入外部训练数据
* 支持查看GPU资源
* 支持控制台查看log

命令举例：

```bash
arena submit tf --name=style-transfer              \
              --gpus=2              \
              --tensorboard         \
              --workers=2              \
              --workerImage=registry.cn-hangzhou.aliyuncs.com/tensorflow-samples/neural-style:gpu \
              --workingDir=/neural-style \
              --ps=1              \
              --psImage=registry.cn-hangzhou.aliyuncs.com/tensorflow-samples/style-transfer:ps   \
              "python neural_style.py --styles /neural-style/examples/1-style.jpg --iterations 1000000"
```

对于上面的参数可以通过 `arena submit tf --help` 来查看，这里简单介绍下：

* `gpus` : 每个 `worker` 节点使用的 `gpu` 数量，目前 `arena` 并不支持对 `worker` 节点进行地质。
* `tensorboard` : 是否开启 `tensorboard`
* `workers` : `worker` 节点数量
* `ps` : `ps` 节点数量

而需要需要查看系统的 `gpu` 分配情况，可以使用下面的命令：

```bash
arena top node
```

对于具体的 `job gpu` 使用情况， 则使用下面的命令：

```bash
aren top job style-transfer
```

然后可以开启控制台查看logs：

```bash
arena logview style-transfer
```

然后就可以在浏览器中查看了。

对于 `tensorboard` 的查看， 使用 `arena get style-transfer` 就能看到 `tensorboard` 的地址。

## 总结

`Arena` 的项目成立就是为了方便数据科学家来更好的使用 `Kubeflow` 来进行训练任务。 目前项目还处于早期，支持的功能还比较基本。但是对于目前的功能，已经可以支持基本的任务操作。而云原生的理念，使得我对于 `Arena` 这个项目还是很看好的。 数据科学家和希望理解 `Kubeflow` 的工程师都可以试用了解一下。