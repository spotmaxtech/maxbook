# ConfigMap和Secret：配置应用程序

## 更改容器的主进程

我们需要一个可以用配置控制进程的服务，例如配置不同参数，程序表现不一样，我们看下这个程序

```bash
# cat fortuneloop.sh
#!/bin/bash
trap "exit" SIGINT

INTERVAL=$1   #默认值来自命令行参数
echo Configured to generate new fortune every $INTERVAL seconds

mkdir -p /var/htdocs

while :
do
  echo $(date) Writing fortune to /var/htdocs/index.html
  /usr/games/fortune > /var/htdocs/index.html
  sleep $INTERVAL
done
```

里面有个环境变量**INTERVAL**可以控制休眠的时间长短（默认值来自命令行参数），也就控制了fortune财富“鸡汤“的变化速度，我们来用Dockerfile制作这个服务

```erlang
# cat Dockerfile
FROM ubuntu:latest

RUN apt-get update ; apt-get -y install fortune
ADD fortuneloop.sh /bin/fortuneloop.sh

ENTRYPOINT ["/bin/fortuneloop.sh"]
CMD ["10"]
```

大家看懂服务逻辑就行了，镜像已经制作好了的。我们直接进入kubernetes部署环节

## 将命令行选项传递给应用程序

见pod描述文件

```yaml
# cat fortune-pod-args.yaml
apiVersion: v1
kind: Pod
metadata:
  name: fortune2s
spec:
  containers:
  - image: luksa/fortune:args   #生产财富鸡汤
    args: ["2"]                 #默认2秒
    name: html-generator
    volumeMounts:
    - name: html
      mountPath: /var/htdocs
  - image: nginx:alpine         #对外提供服务
    name: web-server
    volumeMounts:
    - name: html
      mountPath: /usr/share/nginx/html
      readOnly: true
    ports:
    - containerPort: 80
      protocol: TCP
  volumes:
  - name: html
    emptyDir: {}
```

create试试看(可以跳过)，因为上面的INTERVAL是来自命令行，所以结果可想而知频率是固定的2秒。

## 设置暴露给应用程序的环境变量

```yaml
# cat fortune-pod-env.yaml
apiVersion: v1
kind: Pod
metadata:
  name: fortune-env
spec:
  containers:
  - image: luksa/fortune:env
    env:               #和上面不一样，增加了环境变量
    - name: INTERVAL
      value: "30"      #30s
    name: html-generator
    volumeMounts:
    - name: html
      mountPath: /var/htdocs
  - image: nginx:alpine
    name: web-server
    volumeMounts:
    - name: html
      mountPath: /usr/share/nginx/html
      readOnly: true
    ports:
    - containerPort: 80
      protocol: TCP
  volumes:
  - name: html
    emptyDir: {}
```

create试试，我们继续使用port-forward代理访问pod端口

```
$ k create -f fortune-pod-env.yaml
pod/fortune-env created

$ k port-forward fortune-env 8080:80                                                                                                                                           1 ↵
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
```

然后在另一个控制台访问，观察是否设定的环境变量秒数

```
$ while true; do curl http://localhost:8080; sleep 3;  done
```

## 通过ConfigMap配置应用程序

{% hint style="success" %}
ConfigMap也是我们最常用的配置服务的方式了
{% endhint %}

我们现在为财富鸡汤服务创建一个配置文件

```yaml
# cat fortune-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: fortune-config
data:
  sleep-interval: "25"
```

{% hint style="info" %}
ConfigMap还有其他方式创建，不过yaml方式是我们推荐经常使用的，其他方式不做练习了。
{% endhint %}

create并get，使用-o yaml配合get可以获得最全的yaml描述

```
$ kubectl create -f fortune-config.yaml                                                                                                                                      130 ↵
configmap/fortune-config created

$ kubectl get configmap/fortune-config -o yaml
apiVersion: v1
data:
  sleep-interval: "25"
kind: ConfigMap
metadata:
  creationTimestamp: "2020-01-10T12:48:10Z"
  name: fortune-config
  namespace: liuzongxian
  resourceVersion: "86688245"
  selfLink: /api/v1/namespaces/liuzongxian/configmaps/fortune-config
  uid: 74ec7700-33a7-11ea-bb51-3a8f4bdea85b
```

上述**data**部分就是我们的配置了，现在我们使用这个配置来为服务设置环境变量（这个操作和传统开发不一样吧）

```yaml
# cat fortune-pod-env-configmap.yaml
apiVersion: v1
kind: Pod
metadata:
  name: fortune-env-from-configmap
spec:
  containers:
  - image: luksa/fortune:env
    env:
    - name: INTERVAL
      valueFrom:             #这段控制了环境变量来自配置
        configMapKeyRef:
          name: fortune-config #配置文件名
          key: sleep-interval  #具体数据的key
    name: html-generator
    volumeMounts:
    - name: html
      mountPath: /var/htdocs
  - image: nginx:alpine
    name: web-server
    volumeMounts:
    - name: html
      mountPath: /usr/share/nginx/html
      readOnly: true
    ports:
    - containerPort: 80
      protocol: TCP
  volumes:
  - name: html
    emptyDir: {}
```

```
$ k create -f fortune-pod-env-configmap.yaml
pod/fortune-env-from-configmap created

$ k get all
NAME                             READY   STATUS    RESTARTS   AGE
pod/fortune-env                  2/2     Running   0          14m
pod/fortune-env-from-configmap   2/2     Running   0          4s
```

同样使用port-forward看下调用下服务吧，这里不贴出结果

```
$ k port-forward fortune-env-from-configmap 8080:80
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
```

从上面的实例可以看出ConfigMap是一个含有数据的资源，它能借助kubernetes转为容器所使用的环境变量、参数、甚至是文件。

### 将ConfigMap暴露为pod中的文件

拿上述服务中的nginx做这个实验，先看看它的配置文件，我们尝试控制它是否开启压缩

```
# cat my-nginx-config.conf
server {
    listen              80;
    server_name         www.kubia-example.com;

    gzip on;
    gzip_types text/plain application/xml;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

}
```

可以将这个文件直接创建成ConfigMap，见

{% hint style="info" %}
出现命名冲突，记得清理自己空间以前部署的资源
{% endhint %}

```bash
#删掉重建
$ k delete configmap fortune-config                                                                                                                                            1 ↵
configmap "fortune-config" deleted

$ kubectl create configmap fortune-config --from-file=configmap-files
configmap/fortune-config created
```

我们使用的是文件夹，里面含有两个文件，从configmap中能看出

```
$ kubectl get configmap fortune-config -o yaml
apiVersion: v1
data:
  my-nginx-config.conf: |
    server {
        listen              80;
        server_name         www.kubia-example.com;

        gzip on;
        gzip_types text/plain application/xml;

        location / {
            root   /usr/share/nginx/html;
            index  index.html index.htm;
        }

    }
  sleep-interval: |
    25
kind: ConfigMap
metadata:
  ... ...
  ... ...
```

有my-nginx-config.conf和sleep-interval两个，那我们使用他们

{% hint style="info" %}
后面yaml文件都可能都会有点大，大家能理解大概就可以了，不过这个文件比较典型，先努力读懂它
{% endhint %}

```yaml
# cat fortune-pod-configmap-volume.yaml
apiVersion: v1
kind: Pod
metadata:
  name: fortune-configmap-volume
spec:
  containers:
  - image: luksa/fortune:env
    env:
    - name: INTERVAL
      valueFrom:
        configMapKeyRef:
          name: fortune-config
          key: sleep-interval
    name: html-generator
    volumeMounts:
    - name: html
      mountPath: /var/htdocs
  - image: nginx:alpine
    name: web-server
    volumeMounts:
    - name: html
      mountPath: /usr/share/nginx/html
      readOnly: true
    - name: config
      mountPath: /etc/nginx/conf.d
      readOnly: true
    - name: config
      mountPath: /tmp/whole-fortune-config-volume
      readOnly: true
    ports:
      - containerPort: 80
        name: http
        protocol: TCP
  volumes:
  - name: html
    emptyDir: {}
  - name: config      #guan
    configMap:
      name: fortune-config
```

部署上，看看效果

```yaml
$ kubectl create -f fortune-pod-configmap-volume.yaml                                                                                                                          1 ↵
pod/fortune-configmap-volume created

$ k port-forward fortune-configmap-volume 8080:80
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
```

在另外一个控制台，curl能看到开启了gzip

```yaml
$ curl -H "Accept-Encoding: gzip" -I localhost:8080
HTTP/1.1 200 OK
Server: nginx/1.17.7
Date: Fri, 10 Jan 2020 13:59:24 GMT
Content-Type: text/html
Last-Modified: Fri, 10 Jan 2020 13:59:03 GMT
Connection: keep-alive
ETag: W/"5e188327-5c"
Content-Encoding: gzip
```

Well！现在我们体验了如何在kubernetes中为自己的应用使用配置文件，这样一来我们发版管理配置是不是更方便了呢？

## 通过Secret传递敏感配置信息

有时我们有一些账号密码不能明文，kubernetes提供了Secret资源和ConfigMap使用非常相似，只是对文本加密了而已。

我们常见的是给阿里云、亚马逊的密钥做加密处理，这些密钥是用来控制权限访问云商资源的。

首先创建一个secret

```yaml
k create secret generic ali-key --from-literal=key=1234567890abcdefghijk --from-literal=secret=secret1234567890abcdefg
```

查看已经创建的secret

```yaml
$ k get secret                                                                                                                                          1 ↵
NAME                  TYPE                                  DATA   AGE
ali-key               Opaque                                2      9s

$ k describe secret ali-key
Name:         ali-key
Namespace:    liuzongxian
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
key:     21 bytes
secret:  23 bytes
```

我们现在就可以使用secret了，写一个yaml

```yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: package-test
spec:
  schedule: "*/2 * * * *"
  jobTemplate:
    spec:
      template:
        metadata:
          labels:
            app: package-test
        spec:
          restartPolicy: OnFailure
          containers:
            - name: main
              image: registry.cn-hangzhou.aliyuncs.com/hio-open-image/hio-open:1.0.7
              imagePullPolicy: Always
              command: ["sh"]
              args: ["-c", "echo $accesskeyid $accesskeysecret"]
              env:
                - name: env
                  value: "tests"
                - name: accesskeyid
                  valueFrom:
                    secretKeyRef:
                      name: ali-key
                      key: key
                - name: accesskeysecret
                  valueFrom:
                    secretKeyRef:
                      name: ali-key
                      key: secret
                - name: parentPackageId
                  value: "2"
                - name: taskId
                  value: "3"
                - name: parentPackageOssKey
                  value: "channelpkg/2020/05/27/01/channelDemo_1.0.0.apk"
                - name: packageInfos
                  value: "11:1111,22:2222,33:3333"

```

将它保存到文件 package-test.yaml，执行

```yaml
k apply -f package-test.yaml

```

它是一个cronjob，每两分钟运行一次作业（启动一个pod），我们的作业就是简单的打印环境变量 key和secret。

## 思考题

> * 请谈谈都有哪几种配置应用的方式，和传统开发的区别

