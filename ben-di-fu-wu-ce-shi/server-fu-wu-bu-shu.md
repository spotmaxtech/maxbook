# server服务部署

配置server服务的docker镜像，可访问端口8081，仅内网访问，通过gRPC框架实现与web服务通信。

## 编写.proto的服务定义文件

定义消息（响应），定义消息（请求），服务接口定义，服务端和客户端都要遵守该接口进行通信。

![](<../.gitbook/assets/截屏2021-07-08 下午3.34.34.png>)

执行命令，生成基于go语言的调用背景颜色的方法。

```
protoc --go_out=plugins=grpc:. colour.proto
```

##  部署server服务镜像

在dockerfile中配置server服务镜像的环境变量。

```
ENV SERVER_PORT=8081 \
    BG=123456
```

 在脚本中交叉编译。

```
GOARCH=amd64 GOOS=linux go build -ldflags "-s -w" -o maxcloud-demo-server main.go
```

 赋予可执行权限，创建docker镜像，并将镜像push到DockerHub，删除本地镜像。

```
chmod +x maxcloud-demo-server
docker build -t crazywolf/maxcloud-demo-server:latest .
docker push crazywolf/maxcloud-demo-server:latest
rm maxcloud-demo-server
```

##  编写server服务程序

server服务用于处理web服务，处于挂起待处理状态，要通过内网访问。

* 默认端口8081，可以使用 SERVER_PORT 环境变量修改默认端口
* BC环境变量设置默认返回的颜色

![](<../.gitbook/assets/截屏2021-07-08 下午6.10.43.png>)

建立gRPC服务器，并监听服务器端口。

![](<../.gitbook/assets/截屏2021-07-08 下午6.13.38.png>)

## 部署server服务

在项目列表新建项目，添加server服务，server服务需在demo命名空间下,服务名称为maxcloud-demo-server，端口为8081，设置环境变量，通过gRPC模式访问，勾选仅内网访问。

![](<../.gitbook/assets/截屏2021-07-28 上午11.29.53.png>)

设置环境变量BC。

![](<../.gitbook/assets/截屏2021-07-28 上午11.32.04.png>)

选择内部访问并提交。

![](<../.gitbook/assets/截屏2021-07-28 上午11.32.42.png>)

设置其他两个不同版本的server，仅环境变量不同。

![](<../.gitbook/assets/截屏2021-07-28 上午11.25.26.png>)

