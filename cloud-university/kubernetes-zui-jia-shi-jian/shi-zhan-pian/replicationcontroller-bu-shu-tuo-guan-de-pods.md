# ReplicationController：部署托管的pods

## 保持pod的健康

我们制作一个中途不能正常处理请求的服务版本，用于做实验

```javascript
// cat app.js
const http = require('http');
const os = require('os');

console.log("Kubia server starting...");

var requestCount = 0;

var handler = function(request, response) {
  console.log("Received request from " + request.connection.remoteAddress);
  requestCount++;
  if (requestCount > 5) {  // 注意这里，请求超过5次会500错误
    response.writeHead(500);
    response.end("I'm not well. Please restart me!");
    return;
  }
  response.writeHead(200);
  response.end("You've hit " + os.hostname() + "\n");
};

var www = http.createServer(handler);
www.listen(8080);
```

这段代码大家阅读理解就好，镜像的操作都已经做好了。直接用yaml部署它，体验一下

```yaml
# cat kubia-liveness-probe.yaml
apiVersion: v1
kind: Pod
metadata:
  name: kubia-liveness
spec:
  containers:
  - image: luksa/kubia-unhealthy
    name: kubia
    livenessProbe:     #注意这里增加了健康监测
      httpGet:         #http访问8080端口
        path: /
        port: 8080
```

create并查看这个pod

```yaml
$ kubectl create -f kubia-liveness-probe.yaml
pod/kubia-liveness created

$ kubectl describe pod kubia-liveness

Name:               kubia-liveness
Containers:
  kubia:
    Liveness:       http-get http://:8080/ delay=0s timeout=1s period=10s #success=1 #failure=3
Events:
  Type     Reason     Age                From                               Message
  ----     ------     ----               ----                               -------
  Normal   Scheduled  115s               default-scheduler                  Successfully assigned liuzongxian/kubia-liveness to cn-hongkong.10.32.100.48
  Normal   Pulling    115s               kubelet, cn-hongkong.10.32.100.48  Pulling image "luksa/kubia-unhealthy"
  Normal   Pulled     89s                kubelet, cn-hongkong.10.32.100.48  Successfully pulled image "luksa/kubia-unhealthy"
  Normal   Created    89s                kubelet, cn-hongkong.10.32.100.48  Created container kubia
  Normal   Started    89s                kubelet, cn-hongkong.10.32.100.48  Started container kubia
  Warning  Unhealthy  10s (x3 over 30s)  kubelet, cn-hongkong.10.32.100.48  Liveness probe failed: HTTP probe failed with statuscode: 500
  Normal   Killing    10s                kubelet, cn-hongkong.10.32.100.48  Container kubia failed liveness probe, will be restarted
```

上面的events内容中出现了unhealthy信息，pod要被重启，从pod信息上能看到已经有重启次数计数了。

```yaml
$ k get pod kubia-liveness
NAME             READY   STATUS    RESTARTS   AGE
kubia-liveness   1/1     Running   3          6m17s
```

### 附加属性

我们有很多服务启动是需要时间的，为了避免频繁的重启，需要设置一下延时检测时间

```yaml
# cat kubia-liveness-probe-initial-delay.yaml
apiVersion: v1
kind: Pod
metadata:
  name: kubia-liveness
spec:
  containers:
  - image: luksa/kubia-unhealthy
    name: kubia
    livenessProbe:
      httpGet:
        path: /
        port: 8080
      initialDelaySeconds: 15   # 注意这个参数
```

{% hint style="success" %}
initialDelaySeconds 这个参数很有实际用处
{% endhint %}

大家课下可以自己做这个实验，观察pod描述中的events日志验证

## 运行pod的多个实例

我们服务都是多副本的，例如100 - 200这样的规模，高可用的实现，我们实现一下。在基础部分我们尝试了一下，用的就是replicacontroller，看看这个yaml

```yaml
# cat kubia-rc.yaml
apiVersion: v1
kind: ReplicationController  #使用副本控制
metadata:
  name: kubia
spec:
  replicas: 3                #3个副本
  selector:
    app: kubia               #这里表示控制的是kubia服务
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
```

但是最新的kubernetes已经建议大家使用新的副本集控制器ReplicaSet代替了。所以我们直接做ReplicaSet的练习，他们yaml文件类似

```yaml
# cat kubia-replicaset.yaml
apiVersion: apps/v1beta2     #api有变化
kind: ReplicaSet             #和上面的不同，大家用这个
metadata:
  name: kubia
spec:
  replicas: 3
  selector:
    matchLabels:
      app: kubia             #注意，是按照这个标签控制副本的 
  template:
    metadata:
      labels:
        app: kubia           #所以，为pod打上标签
    spec:
      containers:
      - name: kubia
        image: luksa/kubia
```

创建一下这个文件

{% hint style="info" %}
kubectl create这类指令大家已经熟悉了吧，后续可能不会一一列出这个指令了，有需要可以参考前面的命令。

$ k create -f kubia-replicaset.yaml
{% endhint %}

```yaml
$ k get all
NAME              READY   STATUS    RESTARTS   AGE
pod/kubia-gk6ln   1/1     Running   0          24s
pod/kubia-jbxqc   1/1     Running   0          24s
pod/kubia-wjlfj   1/1     Running   0          24s

NAME                    DESIRED   CURRENT   READY   AGE
replicaset.apps/kubia   3         3         3       25s
```

检查一下我们的副本集，状态还挺清楚的

```yaml
$ k describe rs kubia
Name:         kubia
Namespace:    liuzongxian
Selector:     app=kubia
Labels:       <none>
Annotations:  <none>
Replicas:     3 current / 3 desired
Pods Status:  3 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=kubia
  Containers:
   kubia:
    Image:        luksa/kubia
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age   From                   Message
  ----    ------            ----  ----                   -------
  Normal  SuccessfulCreate  2m9s  replicaset-controller  Created pod: kubia-gk6ln
  Normal  SuccessfulCreate  2m9s  replicaset-controller  Created pod: kubia-jbxqc
  Normal  SuccessfulCreate  2m9s  replicaset-controller  Created pod: kubia-wjlfj
```

## 节点异常自动调度pod

因为有了这个副本集控制，kubernetes会时刻帮你看管副本个数，如果某个node（注意node概念是底层节点）错误导致丢失了某些副本，控制器会补充足够的副本在其他node上。

由于把节点下掉操作对大家环境影响较大，这里先不演示了。

## 水平缩放pod

修改一下副本集个数，将replica改小和改大，例如5，观察一下

```yaml
$ kubectl edit rs kubia
replicaset.extensions/kubia edited
```

输出类似

```yaml
$ k get all
NAME              READY   STATUS    RESTARTS   AGE
pod/kubia-brjwr   1/1     Running   0          48s
pod/kubia-djttq   1/1     Running   0          48s
pod/kubia-gk6ln   1/1     Running   0          5m34s
pod/kubia-jbxqc   1/1     Running   0          5m34s
pod/kubia-wjlfj   1/1     Running   0          5m34s

NAME                    DESIRED   CURRENT   READY   AGE
replicaset.apps/kubia   5         5         5       5m35s
```

相比以前部署是不是非常简单了呢？

## 集群节点上运行系统级的pod

我们有一种部署需求，需要在**每台节点有且只有一个**副本，例如consul agent、redis、fluentd等等辅助性的组件，可以使用DaemonSet副本集。

```yaml
# cat kubia-daemonset.yaml
apiVersion: apps/v1beta2
kind: DaemonSet        #这里是Daemonset
metadata:
  name: kubia
spec:
  #replicas: 3         #不再使用副本集了
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
```

比较简单，我们不必做这个实验了，理解它的用途就好，做一个特殊的实验，见下面描述文件。

```yaml
# cat ssd-monitor-daemonset.yaml
apiVersion: apps/v1beta2
kind: DaemonSet
metadata:
  name: ssd-monitor        
spec:
  selector:
    matchLabels:
      app: ssd-monitor
  template:
    metadata:
      labels:
        app: ssd-monitor
    spec:
      nodeSelector:
        disk: ssd                #限定了pod所在节点
      containers:
      - name: main
        image: luksa/ssd-monitor
```

这个描述功能是在含有ssd的节点上部署监控器ssd-monitor

{% hint style="info" %}
这里已经给个别节点打上了标签，如

$ k label node cn-hongkong.10.32.100.47 disk=ssd
{% endhint %}

```yaml
$ k get node -L disk
NAME                       STATUS   ROLES    AGE     VERSION            DISK
cn-hongkong.10.32.100.46   Ready    <none>   7d1h    v1.14.8-aliyun.1
cn-hongkong.10.32.100.47   Ready    <none>   4d      v1.14.8-aliyun.1   ssd
cn-hongkong.10.32.100.48   Ready    <none>   2d20h   v1.14.8-aliyun.1   ssd
```

创建上面的yaml看看

```yaml
$ k create -f ssd-monitor-daemonset.yaml
daemonset.apps/ssd-monitor created
```

![](../../../.gitbook/assets/image%20%28112%29.png)

## 删除副本集

做一些清理工作吧

```yaml
$ k delete daemonset.apps/ssd-monitor
或
$ k delete all --all # 确保空间没有重要的服务
```

Well Done！到达这里你已经是半个DevOps了，这里是kubernetes的核心功能，能够自动的帮你管理副本集的状态。

下一节，我们让其他服务访问我们的副本集。

## 思考题

> * 在不同命名空间都创建的DaemonSet，会互相冲突么，比如多部署或少部署？

