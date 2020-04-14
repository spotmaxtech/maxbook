# 依赖环境配置

### 权限配置

推荐将max\_group运行在AWS的非spot instance上。所运行max\_group的instance需要具备管理EC2、autoscaling、cloudwatch和SQS的权限，因此需要将以下policy关联到IAM的role或者user中，提供给装有max\_group的instance使用。

### **IAM role  policy**

[AmazonEC2FullAccess](https://console.aws.amazon.com/iam/home?region=us-west-1#/policies/arn%3Aaws%3Aiam%3A%3Aaws%3Apolicy%2FAmazonEC2FullAccess)

[AmazonSQSFullAccess](https://console.aws.amazon.com/iam/home?region=us-west-1#/policies/arn%3Aaws%3Aiam%3A%3Aaws%3Apolicy%2FAmazonSQSFullAccess)

[AutoScalingFullAccess](https://console.aws.amazon.com/iam/home?region=us-west-1#/policies/arn%3Aaws%3Aiam%3A%3Aaws%3Apolicy%2FAutoScalingFullAccess)

[CloudWatchEventsFullAccess](https://console.aws.amazon.com/iam/home?region=us-west-1#/policies/arn%3Aaws%3Aiam%3A%3Aaws%3Apolicy%2FCloudWatchEventsFullAccess)





