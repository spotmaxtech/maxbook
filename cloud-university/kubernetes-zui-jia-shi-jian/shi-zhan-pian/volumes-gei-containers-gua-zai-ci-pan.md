# Volumes：给containers挂载磁盘

## 创建多容器pod

{% hint style="success" %}
本章开始yaml文件会慢慢变大，大家可以慢一些做，多理解其中含义。
{% endhint %}

我们看一个提供fortune服务（财富格言集锦，每次访问都给你一碗不同的鸡汤）的pod

```yaml
# cat fortune-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: fortune
spec:
  containers:
  - image: luksa/fortune
    name: html-generator
    volumeMounts:            #磁盘描述
    - name: html
      mountPath: /var/htdocs #将html磁盘mount到此
  - image: nginx:alpine
    name: web-server
    volumeMounts:            #磁盘描述 
    - name: html
      mountPath: /usr/share/nginx/html  #将html加载
      readOnly: true
    ports:
    - containerPort: 80
      protocol: TCP
  volumes:                   #磁盘的申请或是定义
  - name: html
    emptyDir: {}             #一个最简单的空磁盘
```

## 创建一个可在容器中共享磁盘存储的卷

{% hint style="success" %}
有非常多的磁盘种类，如：

* emptyDir 临时数据空目录
* hostPath 节点磁盘上的路径
* persistentVolumeClaim 动态申请外部存储磁盘
* ... ...
{% endhint %}

### emptyDir磁盘

将前面的yaml描述创建它并访问一下服务

```yaml
$ k create -f fortune-pod.yaml
pod/fortune created

$ k get all
NAME          READY   STATUS    RESTARTS   AGE
pod/fortune   2/2     Running   0          48s

$ k port-forward fortune 8080:80
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80

$ curl http://localhost:8080/
So this is it.  We're going to die.
```

体会一下他们之间的关联

### hostPath

这个我们不自己创建演示了，看看kubernetes是怎么用的

```yaml
#查看系统空间的pod
$ kubectl get pod --namespace kube-system

#描述一下proxy-worker容器
$ kubectl describe pod kube-proxy-worker-88kbk --namespace kube-system
```

可以看到有hostpath使用方式

```yaml
Volumes:
  kube-proxy-worker:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      kube-proxy-worker
    Optional:  false
  xtables-lock:
    Type:          HostPath (bare host directory volume)
    Path:          /run/xtables.lock
    HostPathType:  FileOrCreate
  lib-modules:
    Type:          HostPath (bare host directory volume)
    Path:          /lib/modules
    HostPathType:
  kube-proxy-token-h8j46:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  kube-proxy-token-h8j46
    Optional:    false
```

好了，上面都是使用临时存储、节点存储，现在我们去做一些持久化的操作

## 使用持久化声明获取磁盘

{% hint style="warning" %}
其实还有将已有磁盘挂载到pod中，因为涉及的aws云商等的操作，略显复杂会失去重点，我们不在此练习了
{% endhint %}

例如我们为mongodb数据库申请一块盘，可以这样做

```yaml
# cat mongodb-pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim     #持久化声明
metadata:
  name: mongodb-pvc
spec:
  resources:
    requests:
      storage: 20Gi     #阿里环境最小20GB，后续可能有变化
  accessModes:
  - ReadWriteOnce
  storageClassName: "disk"      #注意选择对class
```

其中storageClassName是个很重要的概念，可以这样查看环境支持的类

```yaml
$ k get storageclass
NAME                       PROVISIONER     AGE
alicloud-disk-available    alicloud/disk   7d18h
alicloud-disk-efficiency   alicloud/disk   7d18h
alicloud-disk-essd         alicloud/disk   7d18h
alicloud-disk-ssd          alicloud/disk   7d18h
disk                       alicloud/disk   6d19h
```

{% hint style="success" %}
disk类非常重要，这是我们公司统一使用的磁盘类，每个云商都有
{% endhint %}

```yaml
$ k create -f mongodb-pvc.yaml
persistentvolumeclaim/mongodb-pvc created

$ k describe pvc/mongodb-pvc
Name:          mongodb-pvc
Namespace:     liuzongxian
StorageClass:  disk
Status:        Pending
Volume:
Labels:        <none>
Annotations:   <none>
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:
Access Modes:
VolumeMode:    Filesystem
Events:
  Type       Reason                Age               From                         Message
  ----       ------                ----              ----                         -------
  Normal     WaitForFirstConsumer  5s (x3 over 17s)  persistentvolume-controller  waiting for first consumer to be created before binding
Mounted By:  <none>
```

提示我们磁盘等待消费者出现才能真正的创建和绑定，那我们挂载它到mongodb中，启动一个mongodb服务

```yaml
# cat mongodb-pod-pvc.yaml
apiVersion: v1
kind: Pod
metadata:
  name: mongodb
spec:
  containers:
  - image: mongo
    name: mongodb
    volumeMounts:
    - name: mongodb-data
      mountPath: /data/db
    ports:
    - containerPort: 27017
      protocol: TCP
  volumes:
  - name: mongodb-data
    persistentVolumeClaim:
      claimName: mongodb-pvc
```

```yaml
$ k create -f mongodb-pod-pvc.yaml
pod/mongodb created
```

 然后查看pod/pvc/pv等资源情况

![](../../../.gitbook/assets/image%20%2856%29.png)

测试一下mongodb的服务吧

```yaml
$ kubectl exec -it mongodb mongo
MongoDB shell version v4.2.2
...
...
```

Cool！现在你成功的将PVC获取的磁盘PV挂载到了mongodb容器pod中

{% hint style="info" %}
如果不指明storageClass会使用kubernetes默认指定的磁盘，我们环境中是“disk“
{% endhint %}

也可以不指定存储类，使用默认的，前提是环境已配置好

```yaml
# cat mongodb-pvc-dp-nostorageclass.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongodb-pvc2
spec:
  resources:
    requests:
      storage: 20Gi
  accessModes:
    - ReadWriteOnce
```

Well！到这里，你已经体验了如何为自己的pod申请磁盘了！东西越来越多了，后面的内容会越来越有趣！

## 思考题

> * 请谈谈StorageClass与PVC机制的好处

