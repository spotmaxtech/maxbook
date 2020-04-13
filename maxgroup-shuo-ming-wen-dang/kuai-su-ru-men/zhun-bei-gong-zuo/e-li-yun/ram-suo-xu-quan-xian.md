# RAM用户所需权限

1、建议使用RAM用户运行，**权限策略名称**如下：

[AliyunECSFullAccess](https://ram.console.aliyun.com/policies/AliyunECSFullAccess/System)

[AliyunMNSFullAcces](https://ram.console.aliyun.com/policies/AliyunMNSFullAccess/System)

[AliyunCloudMonitorFullAccess](https://ram.console.aliyun.com/policies/AliyunCloudMonitorFullAccess/System)

[AliyunESSFullAccess](https://ram.console.aliyun.com/policies/AliyunESSFullAccess/System)

2、MaxGroup建议运行在ECS实体机上，运行在k8s的pod上会出现权限问题

3、注意云监控要授权MNS，报警规则运行maxGroupwfc会自动创建

![](../../../../.gitbook/assets/image%20%2890%29.png)







