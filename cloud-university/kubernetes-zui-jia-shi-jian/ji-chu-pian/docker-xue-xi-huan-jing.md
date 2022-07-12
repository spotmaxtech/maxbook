# Docker学习环境

> 本小节是准备Docker的学习环境，可以提前于线下完成

## 认识MaxCloud上的学习环境

登录MaxCloud之后，右上角切换到<mark style="color:blue;">KubernetesWorkshop</mark>团队空间

![](<../../../.gitbook/assets/image (212) (1).png>)

![](<../../../.gitbook/assets/image (211) (1).png>)

MaxCloud -> 应用管理 -> Apply Yaml

<img src="../../../.gitbook/assets/image (208).png" alt="" data-size="original">

并将下面这段代码粘贴到代码框里（代码语言为Yaml）

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: docker
  namespace: <请替换成自己的命名空间，并去掉括号>
  labels:
    app: docker
spec:
  replicas: 1
  selector:
    matchLabels:
      app: docker
  template:
    metadata:
      labels:
        app: docker
    spec:
      containers:
      - name: docker
        image: docker:dind
        securityContext:
          privileged: true

```

![](<../../../.gitbook/assets/image (214) (1).png>)

如上图提示apply后，就会在应用管理页面初始化好环境了（我们称这个应用为：<mark style="color:blue;">Bundle</mark>），点击进入终端

![](<../../../.gitbook/assets/image (210).png>)

![](<../../../.gitbook/assets/image (209) (1) (1).png>)

进入终端会看到黑屏就成功了，可以预装几个常用软件

```
apk add bash vim curl
```

![](<../../../.gitbook/assets/image (215) (1).png>)

好了，到此我们准备好了docker学习环境。We are ready！

## 思考题

> * 这里提到的Bundle是什么？
> * Yaml的语法风格是？

