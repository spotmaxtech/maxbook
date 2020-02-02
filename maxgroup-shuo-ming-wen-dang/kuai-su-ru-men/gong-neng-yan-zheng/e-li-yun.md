# 阿里云

首先创建一个弹性伸缩组，如下图自动创建3个实例

![](../../../.gitbook/assets/image%20%281%29.png)



利用阿里云的监控进行调试测试，阿里云控制台页面，选择“云监控”，点击调试

![](../../../.gitbook/assets/image%20%2845%29.png)

输入要回收的实例ID，例如：i-bp1emy14ktx2mhb3q186，点击“确认”后，max\_group会接收中断消息

![](../../../.gitbook/assets/image%20%2818%29.png)

中断实例将被detach出伸缩组，并且创建一个新的实例进行替换

![](../../../.gitbook/assets/image%20%2836%29.png)

如果弹性伸缩组填写了“伸缩组内期望实例数”，按照[阿里云文档描述](https://yq.aliyun.com/articles/727372?spm=5176.2020520114.0.0.51b8558aT22hMI)，当手动添加实例时，“伸缩组内期望实例数”会被修改，但max\_group会智能的判断，变更为配置“伸缩组内期望实例数”的值。

![](../../../.gitbook/assets/image%20%2835%29.png)

实例i-bp1emy14ktx2mhb3q186，被detach伸缩组，等待回收，此时该实例不受伸缩组规则约束

![](../../../.gitbook/assets/image%20%2839%29.png)

 



