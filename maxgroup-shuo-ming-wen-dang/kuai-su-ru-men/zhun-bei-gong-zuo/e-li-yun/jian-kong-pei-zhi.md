# 监控配置

MaxGroup依赖通过阿里云MNS消息队列服务来接收Spot中断的通知，并以此来管理Spot，所以用户需要配置消息队列与监控规则。

### 消息队列

在阿里云控制台页面，选择“MNS消息队列”，选择“创建队列”，具体内容如下：

![](../../../../.gitbook/assets/image%20%283%29.png)

### 事件监控

在阿里云控制台页面，选择“云监控”，“事件监控”，点击“创建报警规则”，具体操作步骤如下：




