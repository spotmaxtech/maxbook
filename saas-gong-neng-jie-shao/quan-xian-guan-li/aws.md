# AWS

SpotMax SaaS后台会读取用户云商资源并分析使用情况，以提供有效的优化建议，因此需要用户本身有读取或操作相关资源产品的权限。

## 权限操作

权限设置操作需要登录到AWS控制台进行设置，步骤如下

* 登录AWS控制台—&gt;Identity and Access Management \(IAM\)

![](../../.gitbook/assets/image%20%28157%29.png)

* 使用AWS托管的策略

![](../../.gitbook/assets/image%20%28155%29.png)

![](../../.gitbook/assets/image%20%28158%29.png)

* 新建策略（用于MyCostExplorerReadOnlyAccess策略）

![&#x65B0;&#x5EFA;&#x7B56;&#x7565;](../../.gitbook/assets/image%20%28156%29.png)

![&#x70B9;&#x51FB;&#x5E76;&#x6D4F;&#x89C8;&#x670D;&#x52A1;](../../.gitbook/assets/image%20%28152%29.png)

![&#x9009;&#x62E9;&#x670D;&#x52A1;&#x540D;](../../.gitbook/assets/image%20%28153%29.png)

![&#x914D;&#x7F6E;&#x53EA;&#x8BFB;&#x6743;&#x9650;](../../.gitbook/assets/image%20%28151%29.png)

## **权限添加**

如果用户仅使用SpotMax SaaS提供的资源使用分析功能，则需要添加以下权限

| 权限名称 | 权限说明 |
| :--- | :--- |
| AmazonEC2ReadOnlyAccess | Provides read only access to Amazon EC2 via the AWS Management Console. |
| ElasticLoadBalancingReadOnly | Provides read only access to Amazon ElasticLoadBalancing and dependent services |
| AutoScalingReadOnlyAccess | Provides read-only access to Auto Scaling. |
| CloudWatchReadOnlyAccess | Provides read only access to CloudWatch. |
| _**MyCostExplorerReadOnlyAccess**_ | User created policy. Provides read only access to Cost Explorer.   |

针对用户的资源使用，SpotMax SaaS在提供资源优化建议的同时，也提供一键优化的功能，如果用户需要使用该项功能，则需要添加以下权限

| 权限名称 | 权限说明 |
| :--- | :--- |
| AmazonEC2FullAccess | Provides full access to Amazon EC2 via the AWS Management Console. |
| ElasticLoadBalancingFullAccess | Provides full access to Amazon ElasticLoadBalancing, and limited access to other services necessary to provide ElasticLoadBalancing features. |
| AutoScalingFullAccess | Provides full access to Auto Scaling. |
| CloudWatchFullAccess | Provides full access to CloudWatch. |
| _**MyCostExplorerReadOnlyAccess**_ | User created policy. Provides read only access to Cost Explorer.   |

