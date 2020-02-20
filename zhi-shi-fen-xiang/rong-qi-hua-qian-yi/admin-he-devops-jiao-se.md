# Admin和DevOps角色

## DevOps与Admin

标注DevOps人员是因为容器的正确打开方式是分为容器Admin与DevOps人员。

### Admin

关注集群稳定性、成本，以及SRE的工具，如增加节点规划、CA等

### DevOps

关注应用的一切，包括pod/replica/service/ingress/pvc/configmap/secret等等，这些都是和应用密切相关的资源。

## 集群基础权限管理

### AWS EKS创建初始化权限

[EKS权限说明文档](https://docs.aws.amazon.com/zh_cn/eks/latest/userguide/add-user-role.html)

EKS创建后，默认只能创建者能login这个EKS进行管理，现在我们要让DevOps或是其他管理员进行管理，来操作一下。

#### 确认aws-auth configmap是否存在

```text
kubectl describe configmap -n kube-system aws-auth
```

如果报错提示没有找到需要参考上面的权限说明文档进行配置这个configmap。正常情况只有配置了worknode都是存在的，因为worknode依赖这个配置文件。

#### **将 IAM 用户或角色添加到 Amazon EKS 集群**

使用edit编辑这个aws-auth configmap

```text
kubectl edit -n kube-system configmap/aws-auth
```

在编辑器中增加用户，参照下面的mapUsers数据项

```yaml
apiVersion: v1
data:
  mapRoles: |
    - groups:
      - system:bootstrappers
      - system:nodes
      rolearn: arn:aws:iam::818539432014:role/kmax-worknode
      username: system:node:{{EC2PrivateDNSName}}
  # 注意增加这个mapUsers数据部分，其他部分不需要改动
  mapUsers: |
    - userarn: arn:aws:iam::818539432014:user/spotmax_devops
      username: spotmax_devops
      groups:
        - system:masters
kind: ConfigMap
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","data":{"mapRoles":"- groups:\n  - system:bootstrappers\n  - system:nodes\n  rolearn: arn:aws:iam::818539432014:role/kmax-worknode\n  username: system:node:{{EC2PrivateDNSName}}\n","mapUsers":"- userarn: arn:aws:iam::818539432014:user/spotmax_devops\n  username: spotmax_devops\n  groups:\n    - system:masters\n"},"kind":"ConfigMap","metadata":{"annotations":{},"name":"aws-auth","namespace":"kube-system"}}
  creationTimestamp: "2020-02-10T01:57:20Z"
  name: aws-auth
  namespace: kube-system
  resourceVersion: "48982"
  selfLink: /api/v1/namespaces/kube-system/configmaps/aws-auth
  uid: ac2abe9a-4ba8-11ea-b8ad-0ad6d2355d45
```

{% hint style="info" %}
这里用户有spotmax\_devops，这个账户是给DevOps同学使用的，也可以增加其他管理员例如hujinbin、liuzongxian等等，可以协助管理集群资源。
{% endhint %}

### ALI ACK创建初始化权限

