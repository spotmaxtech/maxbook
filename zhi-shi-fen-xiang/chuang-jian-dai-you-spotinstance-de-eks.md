# 创建带有SpotInstance的EKS

## 这篇文章解决什么问题？

如题，创建带有Spot Instance的EKS学习起来有点费时间，笔者前后做了些实验，也参考了说明文档、Workshop等资料，将最佳的实践在这里记录下来分享给大家。让大家少走些弯路。

通过本文使你掌握独自创建EKS+Spot的技能。

## 面向的读者

管理员、DevOps

## 前期准备

即将部署的EKS，包含Cluster和Worker Node Group，其中Node Group我们将以Autoscaling Group形式创建和管理。从规划角度Cluster、Node Group使用独立的安全组、角色，有利于安全管控，仅需要保证好双方互联即可。

### 网络

Cluster和Node Group在一个VPC中创建，如：vpc-123

Subnet，如：sub-1、sub-2

### 安全组

Cluster：sg-123

Node：sg-456

{% hint style="info" %}
确保能够互相访问
{% endhint %}

### 角色

Cluster IAM Role，如：eks-demo-cluster

Node IAM Role，如：eks-demo-node

## 创建EKS Cluster

使用EKS控制台创建EKS，按上述信息填好：Role/VPC/Subnet/SecurityGroup

## 创建Worker Node Group

关键点在这里，使用CloudFormation方式进行创建。

{% embed url="https://eksworkshop.com/spotworkers/workers/" %}

在页面找到CloudFormation模版

![](../.gitbook/assets/image%20%2856%29.png)

填写信息注意以下几点

* ClusterName：要和上面创建的Cluster保持一致
* NodeInstanceProfile：无法下拉选择，所以复制过来吧，稍微麻烦一点
* ExistingNodeSecurityGroups：无法下拉选择，也要复制过来的，注意是ID，如sg-456
* NodeImageId：这个要选择AWS在Region指定的AMI，它内置了k8s一些工具，具体要查一下。AMI列表在这里：[https://docs.aws.amazon.com/eks/latest/userguide/eks-optimized-ami.html](https://docs.aws.amazon.com/eks/latest/userguide/eks-optimized-ami.html)

其他比较简单，稍微注意下就好了

之后会根据CloudFormation创建出一个完整的Autoscaling Group，同时还有一个配套的模版，是不是很有趣！

![](../.gitbook/assets/image%20%2833%29.png)

## 将Worker Node Group加入到Cluster

### 客户端连接Cluster

```text
$ aws eks --region us-west-2 update-kubeconfig --name <cluster的名字>

$ kubectl get svc
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   172.20.0.1   <none>        443/TCP   87m
```

### 将Node Group加入到Cluster

```text
$ curl -o aws-auth-cm.yaml https://amazon-eks.s3-us-west-2.amazonaws.com/cloudformation/2019-10-08/aws-auth-cm.yaml
$ #编辑文件 vim aws-auth-cm.yaml, 替换正确的Role ARN
$ kubectl apply -f aws-auth-cm.yaml
```

上述指令将IAM Role相关信息写入到了k8s的configmap里，这样Node和Cluster可以互通了

```text
$ k get nodes
NAME                                          STATUS     ROLES    AGE   VERSION
ip-10-11-112-206.us-west-2.compute.internal   NotReady   <none>   4s    v1.14.7-eks-1861c5
ip-10-11-20-148.us-west-2.compute.internal    NotReady   <none>   0s    v1.14.7-eks-1861c5
ip-10-11-68-48.us-west-2.compute.internal     NotReady   <none>   1s    v1.14.7-eks-1861c5
```

## 结束

Done！所以核心点是VPC/SG/Role这些都安排清楚，使用正确的AMI、aws-auth-cm.yaml。

