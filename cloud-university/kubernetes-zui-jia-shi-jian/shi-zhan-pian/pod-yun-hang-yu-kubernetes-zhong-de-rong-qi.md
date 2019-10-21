# Pod：运行于Kubernetes中的容器

> 要点
>
> * 创建、 启动和停止pod
> * 使用标签组织pod和其他资源
> * 使用特定标签对所有pod执行操作
> * 使用命名空间将多个pod分到不重叠的组中
> * 调度pod到指定类型的工作节点

## 使用命名空间对资源进行分组

```bash
$ kubectl create -f cat-manual.yaml -n custom-namespace

$ kubectl get po -n custom-namespace
NAME         READY   STATUS    RESTARTS   AGE
cat-manual   1/1     Running   0          24s
```

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



