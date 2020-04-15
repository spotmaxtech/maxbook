# 阿里云

## 方法一：RAM角色授权

1、推荐使用阿里云ecs运行max\_group,使用RAM角色对ecs进行授权， 授权策略如下：

[AliyunECSFullAccess](https://ram.console.aliyun.com/policies/AliyunECSFullAccess/System)

[AliyunMNSFullAcces](https://ram.console.aliyun.com/policies/AliyunMNSFullAccess/System)

[AliyunCloudMonitorFullAccess](https://ram.console.aliyun.com/policies/AliyunCloudMonitorFullAccess/System)

[AliyunESSFullAccess](https://ram.console.aliyun.com/policies/AliyunESSFullAccess/System)

![](../../../.gitbook/assets/image%20%2838%29.png)

![](../../../.gitbook/assets/image%20%2897%29.png)

创建完成后为角色授权，如下图：

![](../../../.gitbook/assets/image%20%28101%29.png)

2、注意检查云监控和MNS，需提前启用这两个服务，否则无法创建max\_group所使用的依赖环境。（如已启用请忽略）

## 方法二：RAM用户授权

在角色授权方面不方便，也可以使用RAM的用户进行授权，授权方法相同，创建RAM的用户并给用户授权。

![](../../../.gitbook/assets/image%20%2850%29.png)

完成用户授权后，需要在系统中指定accessId和accessKeySecret。

```text
echo 'export ALI_ACCESS_KEY_ID="accessId"' >> /etc/profile
echo 'export ALI_ACCESS_SECRET="accessKeySecret"' >> /etc/profile
source /etc/profile
```

