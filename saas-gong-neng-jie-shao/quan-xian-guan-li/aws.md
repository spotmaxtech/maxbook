# AWS

SpotMax SaaS后台会读取用户云商资源并分析使用情况，以提供有效的优化建议，因此需要用户本身有读取或操作相关资源产品的权限。

## 权限操作

权限设置操作需要登录到AWS控制台进行设置，步骤如下

* 登录AWS控制台—>Identity and Access Management (IAM)
* 新建自定义策略：MyCostExplorerReadOnlyAccess**（此步骤会获取账单信息，如不需要展示请忽略）**

![](<../../.gitbook/assets/image (197).png>)

![](<../../.gitbook/assets/image (85).png>)

![](<../../.gitbook/assets/image (4).png>)

![](<../../.gitbook/assets/image (216).png>)

* 为用户/角色授权：AmazonEC2ReadOnlyAccess、IAMReadOnlyAccess、MyCostExplorerReadOnlyAccess

![](<../../.gitbook/assets/image (253).png>)

## **权限升级**

使用SpotMax SaaS提供的资源使用分析和一键优化的功能，则需要添加以下权限

| 权限名称                               | 权限说明                         |
| ---------------------------------- | ---------------------------- |
| **AmazonEC2FullAccess**            | 管理云服务器服务(EC2)的权限             |
| IAMReadOnlyAccess                  | 只读访问身份和访问管理服务(IAM)的权限        |
| _**MyCostExplorerReadOnlyAccess**_ | 只读访问成本管理器服务(CostExplorer)的权限 |
