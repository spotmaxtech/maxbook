# 持续集成（Gitops）

## 部署并创建Bundle

进入”应用个管理“，点击 “Apply Yaml” 勾选 “将以上资源打包成一个应用”，&#x20;

![](<../../../.gitbook/assets/image (209) (1) (1) (1).png>)

```
apiVersion: apps/v1
kind: Deployment   # 我们这里引入了Deployment
metadata:
  name: kubia
  namespace: << your name space >> #用分配给您的Namespace替换
spec:
  replicas: 3
  selector:
    matchLabels:
     app: kubia
  template:
    metadata:
      name: kubia
      labels:
        app: kubia
    spec:
      containers:
      - image: luksa/kubia:v1
        name: nodejs
```

## 获取Gitops代码片段

在Bundle上点击 “Gitops” 按钮，复制Gitops代码

![](<../../../.gitbook/assets/image (206).png>)

![](<../../../.gitbook/assets/image (208) (1) (1).png>)

```
#!/bin/bash
#请将这段脚本粘贴到合适的位置，并修改这个tag，集成即完成。
tag=`date +%Y-%m-%d-%H-%M-%S`
image="luksa/kubia:${tag}"
secret="ed3dc52340a51e71a3xxxxxxx2fd7"
sign=`echo -n "${image}|${secret}" | md5sum | awk '{print $1}'`
curl --location --request POST 'https://maxcloud-api.spotmaxtech.com/api/external/bundle/upgrade' \
--header 'Content-Type: application/json' \
--data '{
    "bundle_id": 1657615234951111,
    "resource_name": "kubia",
    "resource_kind": "deployment",
    "image": "'$image'",
    "sign": "'$sign'"
}'
```

## 更改期望的版本为v2， 保存并执行

将下面的代码片段保存为v2.sh&#x20;

```
#!/bin/bash
#请将这段脚本粘贴到合适的位置，并修改这个tag，集成即完成。
tag=`date +%Y-%m-%d-%H-%M-%S`
image="luksa/kubia:v2"
secret="ed3dc52340a51e71a3xxxxxxx2fd7"
sign=`echo -n "${image}|${secret}" | md5sum | awk '{print $1}'`
curl --location --request POST 'https://maxcloud-api.spotmaxtech.com/api/external/bundle/upgrade' \
--header 'Content-Type: application/json' \
--data '{
    "bundle_id": 1657615234951111,
    "resource_name": "kubia",
    "resource_kind": "deployment",
    "image": "'$image'",
    "sign": "'$sign'"
}'
```

```
chmod +x v2.sh
./v2.sh
# 结果如下
{"status":200,"request_id":"2c1a2576ccbd4a6c996b2cad8d9d9a9d","message":"success","data":"create success","time":1657616634}
```

点击 Bundle -> Deployment 验证已经升级到V2版本

![](<../../../.gitbook/assets/image (210) (1).png>)

这个脚本也可以使用jenkins等CI/CD工具， 集成部署过程

## Helm Bundle持续集成

创建Helm仓库， 这里我们使用[https://raw.githubusercontent.com/kuiche1982/helm-example/main/kubia](https://raw.githubusercontent.com/kuiche1982/helm-example/main/kubia) （已经为大家添加到了实验环境里）

![](<../../../.gitbook/assets/image (213).png>)

部署Helm

```
$ helm repo add kubia https://raw.githubusercontent.com/kuiche1982/helm-example/main/kubia
$ helm list
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
kubiahelm       chenkui         1               2022-07-12 11:35:10.226574729 +0000 UTC deployed        kubia-0.1.0     1.16.0  
```

创建Helm Bundle

![](<../../../.gitbook/assets/image (212).png>)

配置Gitops

![](<../../../.gitbook/assets/image (216).png>)

```
#!/bin/bash

# version 为您helm的版本，必须和实际对应
# dry_run 是否测试执行
# sets helm set参数 多个参数示例： ["aa=bb","cc==dd"]
secret="9c270a2d6629xxxxxxxxx133b1a18"
sign=`echo -n "${secret}" | md5sum | awk '{print $1}'`

curl --location --request POST 'https://maxcloud-api.spotmaxtech.com/api/external/bundle/helm/upgrade' \
--header 'Content-Type: application/json' \
--data '{
    "sign": "'$sign'",
    "bundle_id": 1657625737658321,
    "version": "0.2.0",  # 这里改为0.2.0
    "dry_run": false,
    "sets": []
}'
# chmod +x v2.sh
# ./v2.sh
# {"status":200,"request_id":"c72b7b1805ed4fa4a26d2f7b01f19cd4","message":"success","data":"apply success","time":1657626107}
```

验证Helm 实例更新到了V2

```
$ helm list   
NAME            NAMESPACE       REVISION        UPDATED                                         STATUS          CHART           APP VERSION
kubiahelm       chenkui         2               2022-07-12 19:41:46.806868607 +0800 +0800       deployed        kubia-0.2.0     1.16.0   
```
