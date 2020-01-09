# Pod：运行于Kubernetes中的容器

> 要点
>
> * 创建、 启动和停止pod
> * 使用标签组织pod和其他资源
> * 使用特定标签对所有pod执行操作
> * 使用命名空间将多个pod分到不重叠的组中
> * 调度pod到指定类型的工作节点

## 为pod创建一个简单的yaml描述文件

```yaml
#cat kubia-manual.yaml
apiVersion: v1
kind: Pod                   #指明资源类型是pod
metadata:
  name: kubia-manual        #pod的名字
spec:
  containers:               #包含的容器，可以是多个啊
  - image: luksa/kubia      #容器使用的镜像
    name: kubia             #容器名字
    ports:
    - containerPort: 8080   #容器监听端口
      protocol: TCP
```

这个配置文件在基础篇出现过，这里可以了解下他们的含义了。

{% hint style="info" %}
我们不需要过多的记住yaml描述包含的每个元素，只需要了解基本组成就可以了，因为在实际使用中大家会经常copy/paste，所以不需要刻意记忆。
{% endhint %}

## 创建pod与了解它

告诉kubernetes，让它按找yaml的描述为我们部署上述资源

{% hint style="info" %}
为便于做练习，适当清理之前的资源是个好习惯，常用指令

* **kubens** 查看当前所在的命名空间，防止误操作别人的资源
* **kubectl get all**   查看已有部署
* **kubectl delete replicationcontroller/kubia** 删除副本控制器，pod也会被自动删除
* **kubectl delete pods --all** 删除全部的pod
{% endhint %}

```text
$ kubectl create -f kubia-manual.yaml
pod/kubia-manual created
```

kubectl create帮助我们创建了pod，查看下

```text
$ k get pods
NAME           READY   STATUS    RESTARTS   AGE
kubia-manual   1/1     Running   0          57s
```

### 了解pod的细节

```text
$ kubectl get pod kubia-manual -o yaml
```

{% hint style="info" %}
-o yaml：常用的选项，可以打印出非常详细的描述，比我们的yaml要详细很多，因为它输出了很多默认的参数，很多参数是我们不需要care的
{% endhint %}

### 查看pod和container的日志

```text
$ k logs kubia-manual
Kubia server starting...

# -c 可以指定查看具体名字的容器日志
$ k logs kubia-manual -c kubia
Kubia server starting...
```

### 本地访问容器服务

我们在基础篇使用了LoadBalancer来访问服务，但这个是要花钱的，同时我们绝大多数服务是内网环境。所以我们试下更安全的方式，这次不用LB（LoadBalancer简称）了。

首先打开一个本地代理，它能将你的请求转发给容器，kubectl全部帮你做这些事情

```text
$ kubectl port-forward kubia-manual 8888:8080
Forwarding from 127.0.0.1:8888 -> 8080
Forwarding from [::1]:8888 -> 8080
```

提示转发通道打开，然后从另外一个控制台访问吧

```text
$ curl http://localhost:8888/
You've hit kubia-manual
```

{% hint style="success" %}
port-forward 非常实用，大家忘记了可以常来翻翻这段的用法
{% endhint %}

{% hint style="info" %}
大家注意了没有，pod的名字就是它的hostname，像主机的hostname没有差别
{% endhint %}

## 使用命名空间对资源进行分组

命名空间，我们其实已经在用了，大家就在各自的命名空间中做实验。

创建一个命名空间很简单，可以使用指令

```text
kubectl create namespace custom-namespace
```

或使用yaml描述

```yaml
# cat custom-namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: custom-namespace
```

{% hint style="info" %}
大家可以做一些练习，清理不用命名空间也简单，但是它会删除空间内所有资源

```text
kubectl delete namespace custom-namespace
```
{% endhint %}

我们使用命名空间是用来隔离资源的，如pod，我们可以让上述服务部署到指定的命名空间。

{% hint style="danger" %}
如果不指定命名空间，那申请的资源默认使用你当前所在命名空间，可以使用 **kubens** 查看

此实验容易在多人环境中名字冲突，大家可以尝试自定义命名空间。如果理解了，也可以不用做，毕竟命名空间操作的时候并不多。
{% endhint %}

![](../../../.gitbook/assets/image%20%284%29.png)

## 给pod打上标签

namespace中的pod会越来越多，为了筛选，我们都会使用标签

```yaml
# cat kubia-manual-with-labels.yaml                                                                                                                                          130 ↵
apiVersion: v1
kind: Pod
metadata:
  name: kubia-manual-v2
  labels:
    creation_method: manual    # key: value的形式
    env: prod                  # 都是文本格式，自由命名
spec:
  containers:
  - image: luksa/kubia
    name: kubia
    ports:
    - containerPort: 8080
      protocol: TCP
```

创建它

```text
$ kubectl create -f kubia-manual-with-labels.yaml
pod/kubia-manual-v2 created
```

运行如下指令体会下标签功效

```text
$ k get pod --show-labels
NAME              READY   STATUS    RESTARTS   AGE   LABELS
kubia-manual      1/1     Running   0          52m   <none>
kubia-manual-v2   1/1     Running   0          66s   creation_method=manual,env=prod


$ k get pod -L creation_method,env
NAME              READY   STATUS    RESTARTS   AGE    CREATION_METHOD   ENV
kubia-manual      1/1     Running   0          52m
kubia-manual-v2   1/1     Running   0          108s   manual            prod


$ k get pod -l creation_method
NAME              READY   STATUS    RESTARTS   AGE
kubia-manual-v2   1/1     Running   0          2m4s


$ k get pod -l creation_method=auto
No resources found.
```

如果对标签不满意，可以增删改

### 修改标签

```text
$ kubectl label pod kubia-manual-v2 env=debug --overwrite                                                                                                                      1 ↵
pod/kubia-manual-v2 labeled
```

{% hint style="warning" %}
如果不带上 --overwrite选项，代表新增，如果已经存在会提示操作失败。
{% endhint %}

### 增加标签

上述修改指令中标签名字不存在会自动添加标签

### 删除标签

修改标签时，将value置空就代表了删除标签

```text
$ kubectl label pod kubia-manual-v2 env= --overwrite                                                                                                                           1 ↵
pod/kubia-manual-v2 labeled

$ k get pod -L creation_method,env
NAME              READY   STATUS    RESTARTS   AGE     CREATION_METHOD   ENV
kubia-manual      1/1     Running   0          59m
kubia-manual-v2   1/1     Running   0          8m51s   manual
```

### 使用标签做选择

前面已经有指令练习过了，常用的是含有指定标签的资源，也可以列出不含标签的资源，如

```text
$ k get pod -l '!creation_method'
NAME           READY   STATUS    RESTARTS   AGE
kubia-manual   1/1     Running   0          63m
```

{% hint style="info" %}
标签过滤是重要功能，等大家在实际工作中要多搜索一下使用方式，过多细节这里不展开了
{% endhint %}

## 将pod部署到指定节点

{% hint style="success" %}
这个非常重要，我们经常会用到，要掌握。因为我们每个团队有各自的节点，不要互相争抢。
{% endhint %}



## 停止和删除pod

```text
$ kubectl delete po cat-manual-v2
$ kubectl delete ns custom-namespace
$ kubectl delete po -l creation_method=manual
$ kubectl delete po --all
$ kubectl delete all --all
```

> 小结
>
> * 如何决定是否应将某些容器组合在一个pod中。
> * pod 可以运行多个进程， 这和非容器世界中的物理主机类似。
> * 可以编写 YAML 或 JSON 描述文件用于创建 pod, 然后查看 pod 的规格及其
>
>   当前状态。
>
> * 使用标签来组织 pod, 并且 一 次在多个 pod 上执行操作。
> * 可以使用节点标签将pod 只调度到提供某些指定特性的节点上。
> * 注解允许入们、 工具或库将更大的数据块附加到 pod。
> * 命名空间可用千允 许不同团队使 用同 一 集 群， 就像它 们使 用单独的Kubemetes 集群一样。
> * 使用 kubectl explain 命令快速查看任何 Kubernetes 资源的信息。



