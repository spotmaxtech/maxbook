# Docker学习环境

## 进入下面这个学习网站

{% embed url="https://labs.play-with-docker.com/" %}

## 学习一下基本操作

![](<../../../.gitbook/assets/image (188).png>)

## 在MaxCloud上学习Docker基本命令

MaxCloud -> 资源管理 -> Deployment -> YAML部署 部署自己的Docker实例

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
