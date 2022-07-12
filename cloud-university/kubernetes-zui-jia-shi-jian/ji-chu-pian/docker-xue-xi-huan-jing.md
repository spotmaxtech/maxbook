# Docker学习环境

## 认识MaxCloud上的学习环境

登录MaxCloud之后，右上角切换到<mark style="color:blue;">KubernetesWorkshop</mark>团队空间

![](<../../../.gitbook/assets/image (210).png>)

![](<../../../.gitbook/assets/image (209).png>)

MaxCloud -> 应用管理 -> Deployment -> YAML部署 部署自己的Docker实例

<img src="../../../.gitbook/assets/image (208).png" alt="" data-size="original">

![](<../../../.gitbook/assets/image (207).png>)

MaxCloud -> 资源管理 -> Deployment -> 点击Docker Deployment -> 进入终端 学习Docker使用

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: docker
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
