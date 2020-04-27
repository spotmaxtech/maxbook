# 开始使用Kubernetes

## 搭建本地的Kubernetes环境

略，请使用课程实验环境

## 使用AWS托管的Kubernetes集群

略，请使用课程实验环境，我们目前提供的是基于阿里云的低成本集群，接入方式见 [接入文档](http://confluence.mobvista.com/pages/viewpage.action?pageId=30369001)，有登陆权限。

测试一下环境

```text
kubectl get nodes
```

![](../../../.gitbook/assets/image%20%2880%29.png)

{% hint style="info" %}
给 kubectl 做一下alias吧，后续我们使用k

alias k=kubectl
{% endhint %}

## 部署与访问你的应用

现在正式在kubernetes中部署我们的应用

```text
kubectl run kubia --image=luksa/kubia --port=8080 --generator=run/v1
```

{% hint style="info" %}
和docker方式做下对比

docker run --name kubia -p 8080:8080 -d luksa/kubia
{% endhint %}

```text
$ kubectl get pods
NAME    READY   STATUS    RESTARTS   AGE
kubia   1/1     Running   0          6s
```

这里就能看到我们启动的pod了，现在需要让我们能访问它，大家可以先执行指令，感受效果，不用过多探究每个参数的含义。

```text
$ kubectl expose rc kubia --type=LoadBalancer --name kubia
service/kubia exposed
```

看看暴露的服务，我们申请了LoadBalancer，云商创建需要1-5分钟的时间，耐心等待下

```text
$ k get svc
NAME    TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
kubia   LoadBalancer   172.22.14.146   <pending>     8080:30019/TCP   5s
```

看到了LoadBalancer后，测试一下

```text
$ curl http://47.56.228.138:8080
You've hit kubia-9bklf
```

## 认识应用的逻辑组成

见课件

## 横向伸缩你的应用

```text
$ k get replicationcontrollers
NAME    DESIRED   CURRENT   READY   AGE
kubia   1         1         1       142m
```

{% hint style="info" %}
大家需要记住一些缩写

* rc：replicationcontrollers
* po：pods

这样能简化指令
{% endhint %}

修改期望的副本即可以完成伸缩

```erlang
$ k scale rc kubia --replicas=3
```

因为我们的应用可以返回主机名，所以每次请求会获得不同的结果

```text
$ while true; do curl http://47.56.228.139:8080; sleep 1;  done
You've hit kubia-qr7r9
You've hit kubia-qr7r9
You've hit kubia-9bklf
```

{% hint style="info" %}
负载均衡器到达上限时，请用左边那个ClusterIP
{% endhint %}

```text
$ kubectl run busybox --rm -i --tty --image busybox -- sh
$ while true; do wget -O- http://<172.22.2.107>:8080; sleep 1;  done
```

## 查看应用跑在哪个节点

```text
kubectl get pods -o wide
```

![](../../../.gitbook/assets/image%20%2822%29.png)

还可以查看pod的一些细节，还有日志，试试指令吧

```text
# kubectl describe pod <选择一个pod>
# kubectl logs <选择一个pod>
```

通过这些操作，是不是感觉比docker方式简单许多，后面还有更精彩的！

## Kubernetes的Dashboard

我们的学习环境安装了kubernetes集群的面板，可以一探究竟

{% hint style="info" %}
请使用https访问47-56-188-118，后续可能会随时变更，不准确请联系管理员
{% endhint %}

提示登陆，请选择令牌

![](../../../.gitbook/assets/image%20%2886%29.png)

这里令牌是一个用户的口令，使用正确的令牌就能代表一个用户登陆了。请按照下述命令获取它的令牌

```text
$ kubectl get secret --namespace kube-dashboard
NAME                                         TYPE                                  DATA   AGE
dashboard-kubernetes-dashboard               Opaque                                0      16m
dashboard-kubernetes-dashboard-token-tr8td   kubernetes.io/service-account-token   3      16m
default-token-xb5sv                          kubernetes.io/service-account-token   3      36m
sh.helm.release.v1.dashboard.v1              helm.sh/release.v1                    1      16m
```

上面有个token的name就是令牌了，查看下，如：

```text
k describe secrets dashboard-kubernetes-dashboard-token-tr8td --namespace kube-dashboard
```

登陆后请浏览一些资源，例如node，其他内容我们后面章节会涉及

## 使用描述文件（yaml）方式

我们换一种方式部署服务，上面的步骤需要记的略多，看这个描述文件

```yaml
# cat kubia.yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: kubia
  labels:
    run: kubia
spec:
  replicas: 1
  selector:
    run: kubia
  template:
    metadata:
      labels:
        run: kubia
    spec:
      containers:
      - name: kubia
        image: luksa/kubia:latest
        ports:
        - containerPort: 8080
          protocol: TCP
```

不需要了解每个语句含义，只需要明白，它定义了rc的名字、副本数、使用的镜像、端口映射等信息就够了

{% hint style="info" %}
在操作之前，可以先清理已经部署过的服务，如
{% endhint %}

```erlang
$ k delete service/kubia replicationcontroller/kubia
service "kubia" deleted replicationcontroller "kubia" deleted
```

使用这个yaml文件告诉kubernetes部署

```erlang
$ kubectl apply -f kubia.yaml
replicationcontroller/kubia created
```

然后看看它帮你做了什么吧

```erlang
$ kubectl get all
NAME              READY   STATUS    RESTARTS   AGE
pod/kubia-j67pf   1/1     Running   0          36s

NAME                          DESIRED   CURRENT   READY   AGE
replicationcontroller/kubia   1         1         1       36s
```

恭喜恭喜！

做到这里，docker和kubernetes你已经入门了！辛苦了！

## 思考题

> * 说说你对kubernetes的看法，例如功能、docker的区别
> * kubectl get/describe/run，请区分他们的区别
> * 怎样查看节点信息
> * 怎样查看pod信息，日志呢？
> * 怎样查看services信息，它的简写是什么？

