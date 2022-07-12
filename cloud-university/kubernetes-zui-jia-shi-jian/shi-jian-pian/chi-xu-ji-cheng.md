# 持续集成

## 部署并创建Bundle

进入”应用个管理“，点击 “Apply Yaml” 勾选 “将以上资源打包成一个应用”，&#x20;

![](<../../../.gitbook/assets/image (209).png>)

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

![](<../../../.gitbook/assets/image (208).png>)

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

![](<../../../.gitbook/assets/image (210).png>)

这个脚本也可以使用jenkins等CI/CD工具， 集成部署过程

\
