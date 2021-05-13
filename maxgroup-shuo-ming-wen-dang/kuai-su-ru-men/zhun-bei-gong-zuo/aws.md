# AWS

## 权限配置

1. 进入aws IAM role界面， [https://console.aws.amazon.com/iam/home\#/roles](https://console.aws.amazon.com/iam/home#/roles) 
2. 所需policy为： [AmazonEC2FullAccess](https://console.aws.amazon.com/iam/home?region=us-west-1#/policies/arn%3Aaws%3Aiam%3A%3Aaws%3Apolicy%2FAmazonEC2FullAccess)，[AmazonSQSFullAccess](https://console.aws.amazon.com/iam/home?region=us-west-1#/policies/arn%3Aaws%3Aiam%3A%3Aaws%3Apolicy%2FAmazonSQSFullAccess)，[AutoScalingFullAccess](https://console.aws.amazon.com/iam/home?region=us-west-1#/policies/arn%3Aaws%3Aiam%3A%3Aaws%3Apolicy%2FAutoScalingFullAccess)，[CloudWatchEventsFullAccess](https://console.aws.amazon.com/iam/home?region=us-west-1#/policies/arn%3Aaws%3Aiam%3A%3Aaws%3Apolicy%2FCloudWatchEventsFullAccess)。如下图：

![](../../../.gitbook/assets/image%20%2848%29.png)

3、添加，创建IAM-GetRole-PassRole策略，目的用于AWS api对信息进行解码，并且向用户授予权限以将角色传递给 AWS 服务，创建流程如下：

![](../../../.gitbook/assets/image%20%28147%29.png)

![](../../../.gitbook/assets/image%20%28148%29.png)

![](../../../.gitbook/assets/image%20%28146%29.png)

![](../../../.gitbook/assets/image%20%28149%29.png)

![](../../../.gitbook/assets/image%20%28150%29.png)

## 启动实例

启动一个ec2实例，并关联上述IAM role。

![](../../../.gitbook/assets/image%20%28142%29.png)

