# 阿里云

SpotMax SaaS后台会读取用户云商资源并分析使用情况，以提供有效的优化建议，因此需要用户本身有读取或操作相关资源产品的权限。

## 权限操作

权限设置操作需要登录到阿里云控制台进行设置，步骤如下：

* 登录阿里云控制台—访问控制

![](../../.gitbook/assets/image%20%28134%29.png)

* 添加权限

![](../../.gitbook/assets/image%20%28136%29.png)

## 权限添加

如果用户仅使用SpotMax SaaS提供的资源使用分析功能，则需要添加以下权限

| 权限名称 | 权限说明 |
| :--- | :--- |
| AliyunECSReadOnlyAccess | 只读访问云服务器服务\(ECS\)的权限 |
| AliyunSLBReadOnlyAccess | 只读访问负载均衡服务\(SLB\)的权限 |
| AliyunEIPReadOnlyAccess | 只读访问弹性公网IP（EIP）的权限 |
| AliyunESSReadOnlyAccess | 只读访问弹性伸缩服务\(ESS\)的权限 |
| AliyunBSSReadOnlyAccess | 只读访问费用中心\(BSS\)的权限 |

针对用户的资源使用，SpotMax SaaS在提供资源优化建议的同时，也提供一键优化的功能，如果用户需要使用该项功能，则需要添加以下权限

| 权限名称 | 权限说明 |
| :--- | :--- |
| AliyunECSFullAccess | 管理云服务器服务\(ECS\)的权限 |
| AliyunSLBFullAccess | 管理负载均衡服务\(SLB\)的权限 |
| AliyunEIPFullAccess | 管理弹性公网IP\(EIP\)的权限 |

