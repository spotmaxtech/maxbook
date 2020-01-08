# 开始使用Docker

## 安装Docker并运行Hello World容器

在安装好的环境中，运行现成的hello world容器

```text
docker run --rm busybox echo "Hello world"
```

{% hint style="info" %}
--rm 可以让容器随着我们使用完毕会自动删除
{% endhint %}

```javascript
# docker方式运行容器最简单语法
docker run <image>:<tag>
```

## 创建一个简单应用

一个简单的http服务代码，nodejs语法

```javascript
const http = require('http');
const os = require('os');

console.log("Kubia server starting...");

var handler = function(request, response) {
  console.log("Received request from " + request.connection.remoteAddress);
  response.writeHead(200);
  response.end("You've hit " + os.hostname() + "\n");
};

var www = http.createServer(handler);
www.listen(8080);
```

如果你本地有node环境，可以运行测试一下

```javascript
node app.js
```

在另外一个控制台 curl

```javascript
$ curl http://localhost:8080/                                                                                                                                                130 ↵
You've hit *这里显示你的机器名*
```

## 构建与运行你的第一个应用

这个Dockerfile可以打包构建应用

```javascript
FROM node:7
ADD app.js /app.js
ENTRYPOINT ["node", "app.js"]
```

在同一目录运行，注意目录，其中kubia是应用名字

```javascript
docker built -t <kubia可以起个名字> .
```

{% hint style="warning" %}
如果大家共用一个docker环境，需要避免名字冲突，如镜像名字、容器名字
{% endhint %}

看到制作过程，根据网络情况拉取node环境有快慢，耐心

![](../../../.gitbook/assets/image%20%2813%29.png)

完成后，运行 docker images 能找到刚制作好的镜像，还挺大的是吧

![](../../../.gitbook/assets/image%20%281%29.png)

运行一下容器镜像

```javascript
docker run --name kubia-container -p 8080:8080 -d kubia
```

如果不想制作可以使用课程已经制作好的镜像，对比一下

```javascript
docker run --rm --name kubia-container -p 8080:8080 -d luksa/kubia
```

{% hint style="info" %}
注意这里可以前面提到的 --rm ，临时做实验销毁比较方便，后面使用--rm时不再赘述了
{% endhint %}

开另一个控制台，访问一下8080试试吧

![](../../../.gitbook/assets/image%20%2832%29.png)

## 探索运行容器的内部

容器有他内部的环境，进去看看

```javascript
$ docker exec -it kubia-container bash
root@0eac0abefd5a:/#
```

登陆进了容器的bash环境，当然我们的node环境集成了bash环境，挨个运行以下日志，看看进程和目录

```javascript
ps aux
ps aux | grep app.js
ls /
```

![](../../../.gitbook/assets/image%20%2836%29.png)

可以知道容器就是把应用完整的环境封装起来运行了，和外部环境是独立的

## 练习常用的操作

临时停止容器

```javascript
docker stop kubia-container
```

不用了销毁容器

```javascript
docker rm kubia-container
```

## 规范镜像和仓库的管理

可以为我们的镜像打tag，并推送到仓库，如代码一样

```javascript
docker tag kubia luksa/kubia
docker push luksa/kubia
```

这里课件已经推送好了，后续大家可以直接使用，不用二遍事了：）

