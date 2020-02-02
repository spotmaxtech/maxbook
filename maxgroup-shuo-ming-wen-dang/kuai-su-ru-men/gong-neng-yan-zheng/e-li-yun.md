# 阿里云

首先创建一个弹性伸缩组，如下图自动创建3个实例

![](../../../.gitbook/assets/image%20%281%29.png)



利用阿里云的监控进行调试测试，阿里云控制台页面，选择“云监控”，点击调试

![](../../../.gitbook/assets/image%20%2844%29.png)

输入要回收的实例ID，例如：i-bp1emy14ktx2mhb3q186，点击“确认”后，max\_group会接收中断消息

![](../../../.gitbook/assets/image%20%2818%29.png)

中断实例将被detach出伸缩组，并且创建一个新的实例进行替换

![](../../../.gitbook/assets/image%20%2835%29.png)

![](../../../.gitbook/assets/image%20%2812%29.png)

实例i-bp1emy14ktx2mhb3q186，被detach伸缩组，等待回收

![](../../../.gitbook/assets/image%20%2838%29.png)





