# 开始使用Kubernetes

## 搭建本地的Kubernetes环境

> 略，请使用课程实验环境
>
> 搭建k8s环境不是我们课程重点，现在都是开箱即用的环境

## 使用MaxCloud托管的Kubernetes集群

### **建立自己专属的实验环境**

登录MaxCloud账号进入课程实验环境，切换团队到KubernetesWorkshop团队，创建自己的项目，关联集群MC-KubernetesWorkshop-playground-new，添加自己的命名空间。我们目前提供的是基于公有云低成本集群（如亚马逊、阿里云）。大家可以通过MaxCloud -> 系统管理->项目管理->终端 来进行后续操作。

<figure><img src="../../../.gitbook/assets/image (98).png" alt=""><figcaption></figcaption></figure>

如上图进入终端后测试一下环境，下述指令可以看到集群含有哪些节点

```
kubectl get nodes
```

<figure><img src="../../../.gitbook/assets/image (91).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
终端环境已经给 kubectl 做了alias别名，后续我们使用k

alias k=kubectl
{% endhint %}

## 部署与访问你的应用

现在正式在kubernetes中部署我们的应用

```
kubectl run kubia --image=luksa/kubia --port=8080
```

{% hint style="info" %}
和docker方式做下对比

docker run --name kubia -p 8080:8080 -d luksa/kubia
{% endhint %}

```
$ kubectl get pods
NAME    READY   STATUS    RESTARTS   AGE
kubia   1/1     Running   0          6s
```

这里就能看到我们启动的pod了，现在需要让我们能访问它，大家可以先执行指令，感受效果，不用过多探究每个参数的含义。

```
$ kubectl expose po kubia --type=LoadBalancer --name kubia
service/kubia exposed
```

看看暴露的服务，我们申请了LoadBalancer，云商创建需要1-5分钟的时间，耐心等待下

```
$ k get svc
NAME    TYPE           CLUSTER-IP       EXTERNAL-IP                                                              PORT(S)          AGE
kubia   LoadBalancer   10.100.114.238   aa4ace9d6de084763b9ca80bcccf121e-357969387.ap-east-1.elb.amazonaws.com   8080:30207/TCP   16s
```

看到了LoadBalancer后，测试一下

```
$ curl http://aa4ace9d6de084763b9ca80bcccf121e-357969387.ap-east-1.elb.amazonaws.com:8080
You've hit kubia
```

{% hint style="success" %}
这里因为暴露了服务给外网，在自己本地电脑也可以curl的，建议体会下
{% endhint %}

## 认识应用的逻辑组成

见课件

## 横向伸缩你的应用

首先清理之前的pod和service

```
kubectl delete svc kubia
kubectl delete po kubia
```

用文件（Yaml语法）方式重新部署kubia服务

```
kubectl apply -f -<<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kubia
  labels:
    app: kubia
spec:
  replicas: 1
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
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: kubia
  name: kubia
spec:
  externalTrafficPolicy: Cluster
  ports:
  - nodePort: 31958
    port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: kubia
  sessionAffinity: None
  type: LoadBalancer
EOF
```

```
$ k get replicaset
NAME    DESIRED   CURRENT   READY   AGE
kubia-69f875bf56   1         1         0       25s
```

```
$ k get deployment 
NAME READY UP-TO-DATE AVAILABLE AGE 
kubia 0/3 3 0 3m5s
```

```
$ k get svc
NAME         TYPE           CLUSTER-IP       EXTERNAL-IP      PORT(S)          AGE
kubia        LoadBalancer   192.168.24.186   39.106.128.174   8080:31958/TCP   28s
```

大家需要记住一些缩写

{% hint style="info" %}
* rs：replicaset
* po：pods
* deploy： deployment

这样能简化指令
{% endhint %}

修改期望的副本即可以完成伸缩

```erlang
$ k scale deploy kubia --replicas=3
deployment.apps/kubia scaled
$ k get po
NAME                                READY   STATUS              RESTARTS   AGE
kubia-546dd5d8b6-76lx9   1/1     Running             0          5m14s
kubia-546dd5d8b6-tchln   0/1     ContainerCreating   0          30s
kubia-546dd5d8b6-zptbt   0/1     ContainerCreating   0          30s
```

因为我们的应用可以返回主机名，所以每次请求会获得不同的结果

```
$ while true; do curl http://39.106.128.174:8080; sleep 1;  done
You've hit kubia-qr7r9
You've hit kubia-qr7r9
You've hit kubia-9bklf
```

{% hint style="warning" %}
负载均衡器到达上限时，请用左边那个ClusterIP
{% endhint %}

```
$ kubectl run busybox --rm -i --tty --image busybox -- sh
$ while true; do wget -O- http://<39.106.128.174>:8080; sleep 1;  done
```

## 查看应用跑在哪个节点

```
kubectl get pods -o wide
```

![](<../../../.gitbook/assets/image (100).png>)

还可以查看pod的一些细节，还有日志，试试指令吧

```
# kubectl describe pod <选择一个pod>
# kubectl logs <选择一个pod>
```

通过这些操作，是不是感觉比docker方式简单许多，后面还有更精彩的！



恭喜恭喜！

做到这里，docker和kubernetes你已经入门了！辛苦了！

{% hint style="info" %}
课程中全部的Yaml代码都能够在如下git地址里找到

git clone https://github.com/liuzoxan/kubernetes-in-action.git

cd kubernetes-in-action/workshop
{% endhint %}

## 思考题

> * 说说你对kubernetes的看法，例如功能、docker的区别
> * kubectl get/describe/run，请区分他们的区别
> * 怎样查看节点信息
> * 怎样查看pod信息，日志呢？
> * 怎样查看services信息，它的简写是什么？
