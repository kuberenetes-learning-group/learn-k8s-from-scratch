# [Rook项目](https://rook.github.io/docs/rook/master/) <small>-- 使用默认storageclass</samll>

一般来说，我们需要为k8s配置一个默认的 ```storageclass``` <sup> [[1]](#ref-1)</sup> ; 这样在安装一些项目的时候，省去了配置 ```pv``` 和 ```storageclass``` 的麻烦。

因此介绍一下存储编排项目 ```rook``` <sup>[[2](#ref-2)]</sup>;

* 1. [安装](#)
* 2. [使用Ceph存储](#Ceph)
* 3. [参考](#-1)

##  1. <a name=''></a>安装

虽然 ```Rook``` 项目支持多种存储系统，但是从目前的开发进度来看，对于 ```Ceph``` 的支持最为完善。所以本文主要介绍基于 ```Ceph``` 的 ```Rook``` 安装。

1. 下载 ```Rook``` 源码

```bash
git clone https://github.com/rook/rook.git
```

2. 部署 ```Rook Operator```

```bash
cd rook/cluster/examples/kubernetes/ceph/

kubectl create -f operator.yaml
```

检查是否安装成功：

```bash
kubectl -n rook-ceph-system get pod
```

如果出现类似下面的结果，则说明安装成功：

```bash
NAME                                  READY   STATUS    RESTARTS   AGE
rook-ceph-agent-tkt9g                 1/1     Running   0          2m13s
rook-ceph-operator-67f4b8f67d-9r48q   1/1     Running   0          47m
rook-discover-qqgf7                   1/1     Running   0          2m13s
```

3. 创建 ```Rook Cluster```

```bash
kubectl create -f cluster.yaml
```

检查是否安装成功:

```bash
kubectl -n rook-ceph get pod
```

如果出现下面的结果，则说明安装成功：

```bash
NAME                                  READY   STATUS      RESTARTS   AGE
rook-ceph-mgr-a-8649f78d9b-xwpqj      1/1     Running     0          58s
rook-ceph-mon-a-698678bdd9-jkmsf      1/1     Running     0          96s
rook-ceph-mon-b-6cbb786c4-p762c       1/1     Running     0          85s
rook-ceph-mon-c-9585d96cb-n6drb       1/1     Running     0          72s
rook-ceph-osd-0-7f7dd474dc-srlvw      1/1     Running     0          25s
rook-ceph-osd-prepare-promote-rsqpn   0/2     Completed   0          32s
```

##  2. <a name='Ceph'></a>使用 ```Ceph``` 存储

1. 创建 ```Block``` 存储

> storageclass.yaml

```yaml
apiVersion: ceph.rook.io/v1
kind: CephBlockPool
metadata:
  name: replicapool
  namespace: rook-ceph
spec:
  replicated:
    size: 1
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
   name: rook-ceph-block
provisioner: ceph.rook.io/block
parameters:
  blockPool: replicapool
  clusterNamespace: rook-ceph
  fstype: ext4 # 这个选项首先检查自己的文件系统是什么， 可以通过 lsblk -f 查看
```

使用下面的命令创建：

```bash
 kubectl create -f storageclass.yaml
```

2. 将 ```Rook``` 创建的 ```storageclass``` 设置成默认

首先检查是否创建成功：

```bash
kubectl get storageclass
```

结果正常应该是：

```
NAME              PROVISIONER          AGE
rook-ceph-block   ceph.rook.io/block   2m
```

然后使用下面的命令设置成默认的 ```storageclass``` ：

```bash
kubectl patch storageclass rook-ceph-block -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

然后继续用上面的命令可以看到 ```rook-ceph-block``` 已经被设置成默认的 ```storageclass``` 了：

```
NAME                        PROVISIONER          AGE
rook-ceph-block (default)   ceph.rook.io/block   4m56s
```

##  3. <a name='-1'></a>参考

<a name="ref-1"></a>[1]  [改变默认 StorageClass](https://kubernetes.io/zh/docs/tasks/administer-cluster/change-default-storage-class/#%E4%B8%BA%E4%BB%80%E4%B9%88%E8%A6%81%E6%94%B9%E5%8F%98%E9%BB%98%E8%AE%A4-storage-class)

<a name="ref-2"></a>[2] [Rook官网](https://rook.github.io/docs/rook/master/)