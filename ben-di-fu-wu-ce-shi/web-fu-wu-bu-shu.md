# web服务部署

配置web服务的docker镜像，可访问8080端口，设置SERVER_URL供外网访问。

## 部署web服务镜像

在dockerfile中配置web服务镜像的环境变量与外网访问域名。

```
ENV PORT=8080 \
    SERVER_URL=maxcloud-demo-server.demo.svc.cluster.local \
    SERVER_PORT=80
```

 在脚本中交叉编译。

```
GOARCH=amd64 GOOS=linux go build -ldflags "-s -w" -o maxcloud-demo-web main.go
```

 赋予可执行权限，创建docker镜像，并将镜像push到DockerHub，删除本地镜像。

```
chmod +x maxcloud-demo-web
docker build -t crazywolf/maxcloud-demo-web:latest .
docker push crazywolf/maxcloud-demo-web:latest
rm maxcloud-demo-web
```

##  编写web服务程序

web服务是客户端，外网访问。

* 默认端口 8080 可以使用 PORT 环境变量修改
* SERVER_URL 和 SERVER_PORT 分别制定远程服务的地址默认 SERVER_URL ： maxcloud-demo-server.demo.svc.cluster.local， SERVER_PORT : 80 （这里必须使用80 而不是gRPC服务的启动端口8081 或者自定义端口）

初始化gRPCserver服务，指定server服务的端口，由客户端访问服务端进行通信。

![](<../.gitbook/assets/截屏2021-07-08 下午7.20.22.png>)

通过gRPC接收server服务的回应访问web服务。

```
func handler(w http.ResponseWriter, r *http.Request)
```

##  部署web服务

在项目列表新建项目，添加web服务，web服务需在demo命名空间下，通过HTTP访问，端口为8080。

![](<../.gitbook/assets/截屏2021-07-28 上午11.50.05.png>)

部署完后提交服务。
