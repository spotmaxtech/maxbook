# AWS

SpotMax SaaS后台会读取用户云商资源并分析使用情况，以提供有效的优化建议，因此需要用户本身有读取或操作相关资源产品的权限。

## 权限操作

权限设置操作需要登录到AWS控制台进行设置，步骤如下

* 登录AWS控制台—&gt;Identity and Access Management \(IAM\)

![](../../.gitbook/assets/image%20%28162%29.png)

* 使用AWS托管的策略

![](../../.gitbook/assets/image%20%28160%29.png)

![](../../.gitbook/assets/image%20%28163%29.png)

* 新建内联策略（用于MyCostExplorerReadOnlyAccess策略）

![](../../.gitbook/assets/image%20%28151%29.png)

![&#x70B9;&#x51FB;&#x6D4F;&#x89C8;&#x670D;&#x52A1;](../../.gitbook/assets/image%20%28155%29.png)

![&#x9009;&#x62E9;&#x670D;&#x52A1;&#x540D;](../../.gitbook/assets/image%20%28156%29.png)

![&#x914D;&#x7F6E;&#x6743;&#x9650;](../../.gitbook/assets/image%20%28158%29.png)

## **权限添加**

使用SpotMax SaaS提供的资源使用分析和一键优化的功能，则需要添加以下权限

| 权限名称 | 权限说明 |
| :--- | :--- |
| AmazonEC2FullAccess | 管理云服务器服务\(EC2\)的权限 |
| ElasticLoadBalancingFullAccess | 管理负载均衡服务\(ELB\)的权限 |
| AutoScalingFullAccess | 管理弹性伸缩服务\(AutoScaling\)的权限 |
| CloudWatchFullAccess | 管理云监控服务\(CloudWatch\)的权限 |
| _**MyCostExplorerReadOnlyAccess**_ | 管理CostExplorer服务的只读权限 |

