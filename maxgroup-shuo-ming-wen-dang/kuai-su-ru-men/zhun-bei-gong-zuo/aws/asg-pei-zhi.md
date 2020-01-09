# ASG配置

针对AWS平台，MaxGroup的工作依赖于AutoScalingGroup的配置。当前AutoScalingGroup的启动方式有两种：基于启动模板和基于启动配置，由于使用配置的方式不适合使用多Spot机型组合模式，因此，MaxGroup推荐使用**启动模板**的方式来创建AutoScalingGroup。

如果您已掌握创建AWS AutoScalingGroup的方式，可以跳过以下步骤，直接阅读[依赖环境配置](https://docs.spotmaxtech.com/maxgroup-shuo-ming-wen-dang/kuai-su-ru-men/aws/zhun-bei-gong-zuo/yi-lai-huan-jing-pei-zhi)

### 创建启动模板



![](../../../../.gitbook/assets/image%20%2810%29.png)

{% hint style="warning" %}
启动模板中无法定义vpc和子网，但可以选择安全组，在选择安全组时，一定要选择将要在AutoScalingGroup使用的vpc相同的安全组。
{% endhint %}

### 创建AutoScalingGroup



![](../../../../.gitbook/assets/image%20%2841%29.png)



![](../../../../.gitbook/assets/image%20%2828%29.png)

![](../../../../.gitbook/assets/image%20%2814%29.png)

其余配置与常规AutoScalingGroup配置相同。

已有AutoScalingGroup但使用了启动配置创建的可以在编辑中修改为启动模板。



![](../../../../.gitbook/assets/image%20%2848%29.png)

