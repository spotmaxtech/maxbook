# 效果展示

可以通过maxcloud平台验证服务效果，或者在本地启动docker测试服务。

## maxcloud平台测试

在maxcloud平台-&gt;项目列表-&gt;-demo项目&gt;查看服务-&gt;maxcloud-demo-web服务-&gt;访问网站

本测试在server服务端配置了三个版本，访问web服务端时，每次刷新都会切换网页颜色。

![](../.gitbook/assets/jie-ping-20210708-xia-wu-7.25.56.png)

## 本地测试

通过终端启动docker服务。

### 启动gRPC服务

```text
docker run -p 8081:8081 --env "SERVER_PORT=8081" --env "BC=654321" crazywolf/maxcloud-demo-server:latest
```

###  启动web服务

```text
docker run -p 8080:8080 --env "SERVER_URL=192.168.5.59" --env "SERVER_PORT=8081" crazywolf/maxcloud-demo-web:latest
```

 



