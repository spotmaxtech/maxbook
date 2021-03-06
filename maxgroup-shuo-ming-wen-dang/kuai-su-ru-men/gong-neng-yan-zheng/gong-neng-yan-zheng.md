# AWS

利用spot的fleet增加和减少instance数量可以触发interrupt事件，因此使用fleet来手动验证interrupt后，max\_group的工作情况。

#### 创建fleet

![](../../../.gitbook/assets/image.png)

![](../../../.gitbook/assets/image%20%28106%29.png)

![](../../../.gitbook/assets/image%20%28144%29.png)

![](../../../.gitbook/assets/image%20%2858%29.png)

创建fleet完成后，可以查看fleet中启动的instance。等待instance状态为running后，将instance attach到autoscaling中

![](../../../.gitbook/assets/image%20%2889%29.png)

![](../../../.gitbook/assets/image%20%2835%29.png)

附加到asg后，修改fleet数量，让机器数量减少而出发interrupt。

![](../../../.gitbook/assets/image%20%28119%29.png)

![](../../../.gitbook/assets/image%20%2833%29.png)

提交以后，max\_group就开始工作了，等待instance replace结束，可以在autoscaling的**活动历史记录**中查看替换详情

![](../../../.gitbook/assets/image%20%2878%29.png)

