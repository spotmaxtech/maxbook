# Deployment: 声明式地升级应用

## 使用Deployment资源升级pod

现在我们要考虑pod升级的情况了，如v1升级到v2和回滚等操作，这个场景需要改动下我们的服务。返回值里标记版本信息。

```javascript
// cat app.js
const http = require('http');
const os = require('os');

console.log("Kubia server starting...");

var handler = function(request, response) {
  console.log("Received request from " + request.connection.remoteAddress);
  response.writeHead(200);
  response.end("This is v1 running in pod " + os.hostname() + "\n");
};

var www = http.createServer(handler);
www.listen(8080);
```

不同的版本服务已经做好了，可以直接部署使用，我们体验一下。

```yaml
# cat kubia-deployment-v1.yaml
apiVersion: apps/v1beta1
kind: Deployment   # 我们这里引入了Deployment
metadata:
  name: kubia
spec:
  replicas: 3
  template:
    metadata:
      name: kubia
      labels:
        app: kubia
    spec:
      containers:
      - image: luksa/kubia:v1
        name: nodejs
```

```yaml
$ kubectl create -f kubia-deployment-v1.yaml --record
deployment.apps/kubia created

$ k get all
NAME                         READY   STATUS    RESTARTS   AGE
pod/kubia-5dfcbbfcff-f5c5t   1/1     Running   0          80s
pod/kubia-5dfcbbfcff-nrrxq   1/1     Running   0          80s
pod/kubia-5dfcbbfcff-tthwm   1/1     Running   0          80s

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/kubia   3/3     3            3           80s

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/kubia-5dfcbbfcff   3         3         3       81s
```

观察下，Deployment自动帮我们创建了rs/pod这些。

```yaml
$ kubectl get deployment kubia
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
kubia   3/3     3            3           53s

$ kubectl describe deployment kubia
Name:                   kubia
Namespace:              liuzongxian
... ...
... ...

#下面指令表示发版完成了 ：）
$ kubectl rollout status deployment kubia                                                                                                                                    127 ↵
deployment "kubia" successfully rolled out
```

{% hint style="success" %}
Deployment 是我们常用的发版方式，且--record记录发版历史，这样我们能够回滚很便利
{% endhint %}

好像deployment没带来什么好处，下面看看升级、回滚

## 执行滚动升级

为了演示v1升级v2过程，我们做一下服务监控工作。这次连通服务service一起创建出来。

```yaml
# cat kubia-deployment-and-service-v1.yaml                                                                                                                                   130 ↵
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: kubia
spec:
  replicas: 3
  template:
    metadata:
      name: kubia
      labels:
        app: kubia
    spec:
      containers:
      - image: luksa/kubia:v1
        name: nodejs
---   # 注意这个写法
apiVersion: v1
kind: Service
metadata:
  name: kubia
spec:
  type: LoadBalancer
  selector:
    app: kubia
  ports:
  - port: 80
    targetPort: 8080
```

```yaml
#清理掉前面的deployment
$ k delete deployment.apps/kubia
deployment.apps "kubia" deleted
```

```yaml
#使用带有服务的deployment部署
$ k create -f kubia-deployment-and-service-v1.yaml
deployment.apps/kubia created
service/kubia created

#设置服务滚动升级的速度，就是不要太快了
$ kubectl patch deployment kubia -p '{"spec": {"minReadySeconds": 10}}'                                                                                                      130 ↵
deployment.extensions/kubia patched
```

服务部署好后，因为有负载均衡器地址，可以访问到服务

{% hint style="info" %}
还记得获取LB的地址么，使用k get svc试试
{% endhint %}

```yaml
#打开服务监控脚本，不停的访问服务，会显示版本号
while true; do curl http://<130.21.109.222替换自己的>; done
```

执行升级操作，这里是将deployment中的image由v1改为了v2

```yaml
$ kubectl set image deployment kubia nodejs=luksa/kubia:v2
deployment.extensions/kubia image updated
```

我们观察到curl返回结果由全部是v1，开始陆续出现了v2，直至全都是v2

```yaml
This is v1 running in pod kubia-5dfcbbfcff-b2445
This is v1 running in pod kubia-5dfcbbfcff-78m6h
This is v1 running in pod kubia-5dfcbbfcff-pvwdj
This is v2 running in pod kubia-7c699f58dd-b2fng
This is v2 running in pod kubia-7c699f58dd-b2fng
This is v1 running in pod kubia-5dfcbbfcff-b2445
This is v1 running in pod kubia-5dfcbbfcff-b2445
This is v2 running in pod kubia-7c699f58dd-b2fng
This is v1 running in pod kubia-5dfcbbfcff-pvwdj
```

这样升级就完成了，一个指令就陆续的完成了部署，是不是很简单呢！

## 回滚pod到上个版本

我们准备了一个有错误的版本v3，它不能正常工作，在从v2到v3升级时看看发生什么

```yaml
$ kubectl set image deployment kubia nodejs=luksa/kubia:v3
deployment.extensions/kubia image updated
```

能看到错误的版本出现了

```yaml
This is v2 running in pod kubia-7c699f58dd-b2fng
This is v2 running in pod kubia-7c699f58dd-c9wpf
This is v3 running in pod kubia-5c98f77977-n8bk4
This is v2 running in pod kubia-7c699f58dd-c9wpf
This is v2 running in pod kubia-7c699f58dd-np59f
This is v2 running in pod kubia-7c699f58dd-b2fng
Some internal error has occurred! This is pod kubia-5c98f77977-n8bk4
This is v2 running in pod kubia-7c699f58dd-c9wpf
This is v2 running in pod kubia-7c699f58dd-np59f
Some internal error has occurred! This is pod kubia-5c98f77977-n8bk4
This is v2 running in pod kubia-7c699f58dd-np59f
This is v2 running in pod kubia-7c699f58dd-c9wpf
This is v2 running in pod kubia-7c699f58dd-np59f
```

此时我们做一下回滚操作

```yaml
$ kubectl rollout undo deployment kubia
deployment.extensions/kubia rolled back
```

然后看到服务回滚到v2了，恢复正常

```yaml
This is v2 running in pod kubia-7c699f58dd-lxc4w
This is v2 running in pod kubia-7c699f58dd-6mq49
Some internal error has occurred! This is pod kubia-5c98f77977-n8bk4
This is v2 running in pod kubia-7c699f58dd-d96vg
This is v2 running in pod kubia-7c699f58dd-d96vg
This is v2 running in pod kubia-7c699f58dd-d96vg
This is v2 running in pod kubia-7c699f58dd-6mq49
```

做到这里是不是觉得升级与回滚都特别便利了呢？我们看看它是怎么做到的

```yaml
$ k get all
NAME                         READY   STATUS    RESTARTS   AGE
pod/kubia-5c98f77977-hn48c   1/1     Running   0          8m17s
pod/kubia-5c98f77977-n8bk4   1/1     Running   0          8m33s
pod/kubia-5c98f77977-vdh69   1/1     Running   0          8m1s

NAME            TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)        AGE
service/kubia   LoadBalancer   172.22.11.131   47.52.107.163   80:32456/TCP   32m

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/kubia   3/3     3            3           32m

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/kubia-5c98f77977   3         3         3       8m35s
replicaset.apps/kubia-5dfcbbfcff   0         0         0       32m
replicaset.apps/kubia-7c699f58dd   0         0         0       28m
```

还记得--record参数么，它帮我们记录了发版历史，从replicaset能看出来已经有了三个版本，所以在回滚时，它能知道我们回滚到哪个历史版本

{% hint style="info" %}
deployment能够控制升级速度、出错暂停升级、回滚到指定版本等等，非常强大，大家可以多做做练习
{% endhint %}

我们回滚到最初的版本v1玩一下

```yaml
$ kubectl rollout history deployment kubia
deployment.extensions/kubia
REVISION  CHANGE-CAUSE
1         <none>
3         <none>
4         <none>

$ kubectl rollout undo deployment kubia --to-revision=1
deployment.extensions/kubia rolled back
```

回到版本v1了

```yaml
This is v1 running in pod kubia-5dfcbbfcff-z86v5
This is v1 running in pod kubia-5dfcbbfcff-pztrs
This is v2 running in pod kubia-7c699f58dd-lxc4w
This is v1 running in pod kubia-5dfcbbfcff-qcvjx
This is v1 running in pod kubia-5dfcbbfcff-pztrs
This is v1 running in pod kubia-5dfcbbfcff-qcvjx
This is v1 running in pod kubia-5dfcbbfcff-z86v5
This is v1 running in pod kubia-5dfcbbfcff-pztrs
This is v1 running in pod kubia-5dfcbbfcff-qcvjx
```

So！Deployment是管理发版的利器。做完这里，你已经完成了绝大部分的容器管理内容了！

{% hint style="info" %}
最后，在做下一节前，注意清理资源
{% endhint %}

## 思考题

> * 记录发版历史有个重要参数，是什么？怎么从v2回滚到v1版本呢？

