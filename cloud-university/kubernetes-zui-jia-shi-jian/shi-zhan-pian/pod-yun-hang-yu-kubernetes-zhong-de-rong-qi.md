# Pod：运行于Kubernetes中的容器

{% hint style="info" %}
* 创建、 启动和停止pod
* 使用标签组织pod和其他资源
* 使用特定标签对所有pod执行操作
* 使用命名空间将多个pod分到不重叠的组中
* 调度pod到指定类型的工作节点
{% endhint %}

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

