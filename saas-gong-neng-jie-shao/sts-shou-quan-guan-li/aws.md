# AWS

如果想要使用SaaS控制台访问及管理SpotMax产品服务，又不想提供对应云商的AccessKey和Secret，则可在SpotMax官网后台设置STS的方式如下：

1. 登录SpotMax账户，并切换到对应云商
2. 进入“密钥信息”页面
3. 填写STS的角色Arn，点击“修改”，会更新并自动检测Arn权限是否完整

![](../../.gitbook/assets/image%20%28170%29.png)

{% hint style="info" %}
当前一个SpotMax账户仅能使用1种授权方式（key&secret或者STS），如果需要修改，则直接在SpotMax官网后台更新即可
{% endhint %}

#### 

