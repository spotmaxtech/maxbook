---
description: Chart
---

# 使用仓库管理chart

{% hint style="info" %}
本小节

* 准备一个仓库，并加入到helm repo列表
* 将我们的chart文件夹做成一个helm包，package命令
* 上传到仓库
* 从仓库中安装
{% endhint %}

我们需要一个charts仓库

```
# 这个仓库是mobvista的一个内部仓库
http://harbor-v2.mobvista.com/

# 登录后请创建自己的一个项目这里是
maxcloud

# 添加到helm repo
$ helm repo add maxcloud http://harbor-v2.mobvista.com/chartrepo/maxcloud

# 非Public的则需要认证
$ helm repo add maxcloud --username=leon@mobvista.com --password=xxx http://harbor-v2.mobvista.com/chartrepo/maxcloud

$ helm repo list                                                                                                                                                             130 ↵
NAME                    URL                                               
bitnami                 https://charts.bitnami.com/bitnami                
elastic                 https://helm.elastic.co                           
prometheus-community    https://prometheus-community.github.io/helm-charts
maxcloud                http://harbor-v2.mobvista.com/chartrepo/maxcloud 
```

将webapp文件夹package一下

```
$ helm package webapp
# 会生成一个压缩包
ls ./
webapp           webapp-0.1.0.tgz
```

上传到仓库中

![](<../../../../.gitbook/assets/image (207) (1).png>)

刷一下仓库中的chart列表

```
# 更新下本地缓存
$ helm repo update maxcloud

$ helm search repo maxcloud 
NAME                                    CHART VERSION   APP VERSION             DESCRIPTION                                 
maxcloud/elasticsearch                  7.15.0          7.15.0                  MaxCloud helm chart for Elasticsearch       
maxcloud/engineplus-operator-helm       2.0.0           1.16.0                  engineplus-operator-helm                    
maxcloud/fluentd                        0.1.0           1.16.0                  A Helm chart for Kubernetes                 
maxcloud/hostname                       0.1.4           16412783828847696       [] Generate by MaxCloud                     
maxcloud/kibana                         7.15.0          7.15.0                  Official Elastic helm chart for Kibana      
maxcloud/rediscache                     6.2.0                                   [redis bundle 复制演示] Generate by MaxCloud
maxcloud/webapp                         0.1.0           1.16.0                  A Helm chart for Kubernetes 

```

安装测试一下

```
$ helm install webapp --version 0.1.0 maxcloud/webapp
NAME: webapp
LAST DEPLOYED: Thu May 12 16:25:22 2022
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
1. Get the application URL by running these commands:
  export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=webapp,app.kubernetes.io/instance=webapp" -o jsonpath="{.items[0].metadata.name}")
  export CONTAINER_PORT=$(kubectl get pod --namespace default $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl --namespace default port-forward $POD_NAME 8080:$CONTAINER_PORT

```

查看资源以及可以在maxcloud做个绑定

![](<../../../../.gitbook/assets/image (208) (1) (1).png>)

设置image版本升级一下试试

```
$ helm upgrade webapp --version 0.1.0 maxcloud/webapp

$ helm upgrade webapp --version 0.1.0 maxcloud/webapp --set image.tag=1.21.0
```



{% hint style="info" %}
常用命令

* helm package webapp
* helm repo update maxcloud
* helm search repo maxcloud
* helm install webapp --version 0.1.0 maxcloud/webapp
{% endhint %}
