# Service：让客户端发现pod并与之通信

## 创建服务，利用单个地址访问一组pod

服务是虚拟的网络管理，不涉及实实在在的pod，所以确保已经创建好了pod，可以使用前面的yaml创建，如

```
$ k create -f kubia-replicaset.yaml                                                                                                                                          130 ↵
replicaset.apps/kubia created
```

```
k create -f -<<EOF
apiVersion: apps/v1
kind: ReplicaSet             
metadata:
  name: kubia
spec:
  replicas: 3
  selector:
    matchLabels:
      app: kubia            
  template:
    metadata:
      labels:
        app: kubia           
    spec:
      containers:
      - name: kubia
        image: luksa/kubia
EOF
```

服务也有yaml描述文件

```yaml
# cat kubia-svc.yaml                                                                                                                                                           1 ↵
apiVersion: v1
kind: Service         #资源类型
metadata:
  name: kubia
spec:
  ports:
  - port: 80          #对外服务端口，很方便吧
    targetPort: 8080  #pod端口
  selector:
    app: kubia        #为含有app=kubia标签的pod创建服务
```

同样，大家不要刻意记忆，理解就好，后面用到可以到这里copy/paste。创建它

```
$ k create -f kubia-svc.yaml                                                                                                                                                 130 ↵
service/kubia created

$ k get all
NAME              READY   STATUS    RESTARTS   AGE
pod/kubia-77bv5   1/1     Running   0          10m
pod/kubia-m89k7   1/1     Running   0          10m
pod/kubia-wwkmh   1/1     Running   0          10m

NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
service/kubia   ClusterIP   172.22.2.238   <none>        80/TCP    5m39s

NAME                    DESIRED   CURRENT   READY   AGE
replicaset.apps/kubia   3         3         3       10m
```

### 使用clusterip访问

上面有个服务CLUSTER-IP（172.22.2.238），要换成你自己的，找到它访问一下，可以这样

```
$ k exec kubia-77bv5 -- curl -s http://172.22.2.238
You've hit kubia-wwkmh
```

{% hint style="info" %}
kubectl exec 是用来执行pod中指令的，所以kubia-77bv5是一个pod，大家要换成自己环境的。curl的-s是silent意思，只输出结果
{% endhint %}

### 使用集群内域名机制访问

每个服务都有唯一的域名，如这个服务对应的域名是

**kubia.\<liuzongxian换成你的命名空间>.svc.cluster.local**

这个域名在集群内部都是可以直接使用的，如：

```
$ kubectl exec kubia-77bv5 -- curl -s http://kubia.liuzongxian.svc.cluster.local                                                                                             130 ↵
You've hit kubia-m89k7
```

{% hint style="info" %}
如果在同一个集群、同一个命名空间，可以直接使用http://kubia代替，会简洁一些
{% endhint %}

### 无法ping通服务ip的原因

我们都习惯ping服务，但实际上集群内部的服务概念是虚拟的，需要结合端口才有意义。

## 将服务公开给外部客户端

上面的方式其实叫做ClusterIP方式，仅能内部访问，那怎样外部客户端访问呢？

### 使用NodePort方式

为每个节点开一个独一无二的端口供这个服务占用，客户端使用节点ip+端口方式访问，请创建下面这个服务

```yaml
# cat kubia-svc-nodeport.yaml
apiVersion: v1
kind: Service
metadata:
  name: kubia-nodeport
spec:
  type: NodePort           #这里指定了type：NodePort
  ports:
  - port: 80
    targetPort: 8080
    nodePort: 30123        #大家换成自己的（30XXX）
  selector:
    app: kubia
```

查看一下

```
# k get svc                                                                                                                                                                  130 ↵
NAME             TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
kubia            ClusterIP   172.22.2.238   <none>        80/TCP         145m
kubia-nodeport   NodePort    172.22.9.134   <none>        80:30123/TCP   21m
```

由于阿里云开的资源默认没有分配公网ip，需要使用内网测试，内网ip可以从节点的名字中找到，如下图的 10.32.100.48

```
# k get node
NAME                       STATUS   ROLES    AGE     VERSION
cn-hongkong.10.32.100.46   Ready    <none>   7d4h    v1.14.8-aliyun.1
cn-hongkong.10.32.100.47   Ready    <none>   4d3h    v1.14.8-aliyun.1
cn-hongkong.10.32.100.48   Ready    <none>   2d23h   v1.14.8-aliyun.1
```

这里稍稍复杂一些，我们启动了一个busybox的pod来测试内网端口

```
$ kubectl run busybox --rm -i --tty --image busybox -- sh
# 然后使用wget测试
wget -O- http://10.32.100.48:<30123换成你自己的端口30XXX>
```

效果如图

![](<../../../.gitbook/assets/image (145).png>)

### 使用负载均衡器

{% hint style="warning" %}
最简单但是要换钱的方式 :sweat\_smile:&#x20;
{% endhint %}

```yaml
# cat kubia-svc-loadbalancer.yaml
apiVersion: v1
kind: Service
metadata:
  name: kubia-loadbalancer
spec:
  type: LoadBalancer       #负载均衡
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: kubia
```

create它，然后看看，然后请找到自己的External IP，就是外网负载均衡器了。

```
$ k create -f kubia-svc-loadbalancer.yaml
service/kubia-loadbalancer created

$ k get svc
NAME                 TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)        AGE
kubia                ClusterIP      172.22.2.238    <none>          80/TCP         153m
kubia-loadbalancer   LoadBalancer   172.22.10.243   47.56.234.108   80:31269/TCP   20s
kubia-nodeport       NodePort       172.22.9.134    <none>          80:30123/TCP   29m

$ curl http://47.56.234.108/
You've hit kubia-77bv5
```

### 通过Ingress反向代理

这里ingress名字比较诡异，但其实就是nginx服务帮你转发。

{% hint style="success" %}
ingress方式非常常用
{% endhint %}

```yaml
# cat kubia-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: kubia
spec:
  rules:
  - host: <kubia换成自己的>.example.com
    #域名这个要配置本地的/etc/hosts
    #如：<Nginx外网IP>    kubia.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: kubia-nodeport
            port:
              number: 80
```

创建效果如下

{% hint style="info" %}
ingress 也是一种资源，同pod/svc类似的
{% endhint %}

```
$ k get ingress                                                                                                                                                              130 ↵
NAME    HOSTS               ADDRESS         PORTS   AGE
kubia   kubia.example.com   47.52.155.217   80      60s
```

从上面的Address代表了nginx外网ip，配置到/etc/hosts下，我们就可以用浏览器输入域名访问了，对必须输入域名。 :stuck\_out\_tongue:&#x20;

{% hint style="danger" %}
这里有个扩展，就是配置nginx的443安全访问，我们不在课程中讲解了
{% endhint %}

### 清理资源

到现在创建了不少资源了，清空一下自己空间的资源吧。

{% hint style="danger" %}
$ k delete all --all 这个指令一定要在自己空间内使用
{% endhint %}

## 就绪探针Readiness Probe

pod成功用的是健康探测，但是服务还不一定准备好，没有准备好服务的pod不应该对外提供服务，所以要加readiness检测。

```yaml
# cat kubia-rc-readinessprobe.yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: kubia
spec:
  replicas: 3
  selector:
    app: kubia
  template:
    metadata:
      labels:
        app: kubia
    spec:
      containers:
      - name: kubia
        image: luksa/kubia
        ports:
        - name: http
          containerPort: 8080
        readinessProbe: #就绪检测探针，可以定制化
          exec:
            command:
            - ls
            - /var/ready  
```

上面描述指明，只要/var/ready不存在，就还不能对外提供服务

```yaml
$ k create -f kubia-rc-readinessprobe.yaml
replicationcontroller/kubia created

$ k get all                                                                                                                                                                  130 ↵
NAME              READY   STATUS    RESTARTS   AGE
pod/kubia-7268z   0/1     Running   0          86s
pod/kubia-czvm6   0/1     Running   0          86s
pod/kubia-r9q55   0/1     Running   0          86s

NAME                          DESIRED   CURRENT   READY   AGE
replicationcontroller/kubia   3         3         0       86s
```

能看到全都没有提供服务，没有进入READY状态，我们进入一个pod满足它，然后再看状态

```yaml
$ kubectl exec kubia-7268z -- touch /var/ready

$ k get all
NAME              READY   STATUS    RESTARTS   AGE
pod/kubia-7268z   1/1     Running   0          3m12s
pod/kubia-czvm6   0/1     Running   0          3m12s
pod/kubia-r9q55   0/1     Running   0          3m12s

NAME                          DESIRED   CURRENT   READY   AGE
replicationcontroller/kubia   3         3         1       3m12s
```

已经ready了，此时service才能让它对外提供服务

好的，至此你完成非常难的一个小节了，应该给自己鼓鼓掌！本章节也是日常经常用到的内容。

## 思考题

> * 请说说就绪探针、健康探针



##
