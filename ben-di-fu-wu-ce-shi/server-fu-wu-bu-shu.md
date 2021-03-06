# server服务部署

配置server服务的docker镜像，可访问端口8081，仅内网访问，通过gRPC框架实现与web服务通信。

## 编写.proto的服务定义文件

定义消息（响应），定义消息（请求），服务接口定义，服务端和客户端都要遵守该接口进行通信。

![](../.gitbook/assets/jie-ping-20210708-xia-wu-3.34.34.png)

执行命令，生成基于go语言的调用背景颜色的方法。

```text
protoc --go_out=plugins=grpc:. colour.proto
```

##  部署server服务镜像

在dockerfile中配置server服务镜像的环境变量。

```text
ENV SERVER_PORT=8081 \
    BG=123456
```

 在脚本中交叉编译。

```text
GOARCH=amd64 GOOS=linux go build -ldflags "-s -w" -o maxcloud-demo-server main.go
```

 赋予可执行权限，创建docker镜像，并将镜像push到DockerHub，删除本地镜像。

```text
chmod +x maxcloud-demo-server
docker build -t crazywolf/maxcloud-demo-server:latest .
docker push crazywolf/maxcloud-demo-server:latest
rm maxcloud-demo-server
```

##  编写server服务程序

server服务用于处理web服务，处于挂起待处理状态，要通过内网访问。

* 默认端口8081，可以使用 SERVER\_PORT 环境变量修改默认端口
* BC环境变量设置默认返回的颜色

![](../.gitbook/assets/jie-ping-20210708-xia-wu-6.10.43.png)

建立gRPC服务器，并监听服务器端口。

![](../.gitbook/assets/jie-ping-20210708-xia-wu-6.13.38.png)

{% hint style="info" %}
注：初次部署服务要在高级高级中勾选内网访问，之后发布的版本自动勾选。
{% endhint %}

