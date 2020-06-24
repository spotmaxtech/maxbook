# 阿里云市场使用指南

## 购买License：

访问以下云市场的按需购买我们的商品：[License](https://market.aliyun.com/products/56838014/cmgj00040678.html?#sku=yuncode3467800001)

![](../.gitbook/assets/image%20%28126%29.png)

## 购买运行镜像

访问以下云市场的按需购买我们的商品：[镜像](https://market.aliyun.com/products/52732002/cmjj00040459.html)

![](../.gitbook/assets/image%20%28125%29.png)

## 服务开启：

1、注意检查云监控和MNS，需提前启用这两个服务，否则无法创建max\_group所使用的依赖环境。（如已启用请忽略）

![](../.gitbook/assets/image%20%2826%29.png)

2、确保云监控授权给mns

![](../.gitbook/assets/image%20%2874%29.png)

## RAM角色授权：

#### **创建RAM角色**

![](../.gitbook/assets/image%20%285%29.png)

![](../.gitbook/assets/image%20%28112%29.png)

![](../.gitbook/assets/image%20%28108%29.png)

点击 完成，用户创建成功，如下内容：

![](../.gitbook/assets/image%20%28105%29.png)

#### 2、点击权限管理，授权策略如下：

[AliyunECSFullAccess](https://ram.console.aliyun.com/policies/AliyunECSFullAccess/System)

[AliyunMNSFullAcces](https://ram.console.aliyun.com/policies/AliyunMNSFullAccess/System)

[AliyunCloudMonitorFullAccess](https://ram.console.aliyun.com/policies/AliyunCloudMonitorFullAccess/System)

[AliyunESSFullAccess](https://ram.console.aliyun.com/policies/AliyunESSFullAccess/System)

[AliyunRAMFullAccess](https://ram.console.aliyun.com/policies/AliyunRAMFullAccess/System)

![](../.gitbook/assets/image%20%28117%29.png)

创建实例时，选择创建好的RAM角色

![](../.gitbook/assets/image%20%2825%29.png)



