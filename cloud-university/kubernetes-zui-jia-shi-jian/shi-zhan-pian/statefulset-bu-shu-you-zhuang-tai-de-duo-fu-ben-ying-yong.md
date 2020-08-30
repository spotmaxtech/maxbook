# StatefulSet：部署有状态的多副本应用

很高兴大家能做到这里，这是容器kubernetes内容最后一个章节了！而且非常简单，恭喜大家！

## 部署有状态的集群应用

针对有状态的应用，多数是数据有状态，例如mongodb副本集等，我们希望数据能够在pod扩容、缩容时磁盘数据不会被丢弃，能够复用磁盘。我们使用StatefulSet完成这些。

```yaml
# cat kubia-statefulset.yaml
apiVersion: apps/v1beta1
kind: StatefulSet         #使用了stateful
metadata: 
  name: kubia
spec:
  serviceName: kubia
  replicas: 2
  template:
    metadata:
      labels:
        app: kubia
    spec:
      containers:
      - name: kubia
        image: luksa/kubia-pet
        ports:
        - name: http
          containerPort: 8080
        volumeMounts:
        - name: data
          mountPath: /var/data
  volumeClaimTemplates:   #为每个pod申请了磁盘  
  - metadata:
      name: data
    spec:
      resources:
        requests:
          storage: 20Gi
      accessModes:
      - ReadWriteOnce
      storageClassName: "disk"
```

创建它看看，大家能够注意到，每个pod名字不再是随机码了，而是有了编号，他们也就是有了固定的名字了，所以对应的磁盘也是有了名字

```yaml
$ k create -f kubia-statefulset.yaml
statefulset.apps/kubia created

$ k get all
NAME          READY   STATUS    RESTARTS   AGE
pod/kubia-0   1/1     Running   0          61s
pod/kubia-1   1/1     Running   0          46s

NAME                     READY   AGE
statefulset.apps/kubia   2/2     61s

$ k get pvc
NAME           STATUS   VOLUME                   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
data-kubia-0   Bound    d-j6c3dilgxismppu271ed   20Gi       RWO            disk           7m39s
data-kubia-1   Bound    d-j6c9x6t7z31woktn8jl3   20Gi       RWO            disk           7m24s
```

这样他们在随着容量变化时，总是能够对号入座，是不是很简单呢？另外StatefulSet相比ReplicaSet，操作基本都是相同的，所以扩缩容等管理我们就不重复了

```yaml
# 如
$ kubectl scale statefulset kubia --replicas=3                                                                                                                                 1 ↵
statefulset.apps/kubia scaled
```

{% hint style="success" %}
StatefulSet在缩容时，pod卸下的磁盘不会被清理掉，因为扩容时还会再被复用，这是固定的机制。大家要记得这个点。
{% endhint %}

Cool！其实StatefulSet是很简单的，但是很多服务都会用到它，需要多练习。

好了，课程到这里告一段落，再次恭喜大家完成了这么多的内容。



## 思考题

> * 谈谈你对ReplicaSet/StatefulSet的理解
> * Deployment和上面两个set有关联么，谈谈你的理解
> * 谈谈你对kubernetes的体验感受

