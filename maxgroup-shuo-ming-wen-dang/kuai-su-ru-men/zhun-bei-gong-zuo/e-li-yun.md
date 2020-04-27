# 阿里云

## 方法一：RAM角色授权

#### **1、推荐使用，创建RAM角色**

![](../../../.gitbook/assets/image%20%285%29.png)

![](../../../.gitbook/assets/image%20%28109%29.png)

![](../../../.gitbook/assets/image%20%28105%29.png)

点击 完成，用户创建成功，如下内容：

![](../../../.gitbook/assets/image%20%28102%29.png)

#### 2、点击权限管理，授权策略如下：

[AliyunECSFullAccess](https://ram.console.aliyun.com/policies/AliyunECSFullAccess/System)

[AliyunMNSFullAcces](https://ram.console.aliyun.com/policies/AliyunMNSFullAccess/System)

[AliyunCloudMonitorFullAccess](https://ram.console.aliyun.com/policies/AliyunCloudMonitorFullAccess/System)

[AliyunESSFullAccess](https://ram.console.aliyun.com/policies/AliyunESSFullAccess/System)

![](../../../.gitbook/assets/image%20%28104%29.png)

## 方法二：RAM用户授权，需要AK

1、使用RAM用户对ecs进行授权， 授权策略如下：

[AliyunECSFullAccess](https://ram.console.aliyun.com/policies/AliyunECSFullAccess/System)

[AliyunMNSFullAcces](https://ram.console.aliyun.com/policies/AliyunMNSFullAccess/System)

[AliyunCloudMonitorFullAccess](https://ram.console.aliyun.com/policies/AliyunCloudMonitorFullAccess/System)

[AliyunESSFullAccess](https://ram.console.aliyun.com/policies/AliyunESSFullAccess/System)



![](../../../.gitbook/assets/image%20%2853%29.png)



完成用户授权后，需要在系统中指定accessId和accessKeySecret。

```text
echo 'export ALI_ACCESS_KEY_ID="accessId"' >> /etc/profile
echo 'export ALI_ACCESS_SECRET="accessKeySecret"' >> /etc/profile
source /etc/profile
```

2、注意检查云监控和MNS，需提前启用这两个服务，否则无法创建max\_group所使用的依赖环境。（如已启用请忽略）



