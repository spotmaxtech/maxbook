# 接入容器集群

## 亚马逊的EKS接入方式

### 环境准备

EKS使用用户方式接入，所以DevOps人员需要具备以下环境

* 安装kubectl
* 安装awscli
* ~~安装aws-iam-authenticator （这一步已经淘汰了）~~
* 配置aws configure

{% hint style="info" %}
aws configure请使用spotmax\_devops用户，如有特殊需要，请找到集群管理员操作
{% endhint %}

以上环境在mac上都可以使用brew安装，请google必要的文档，或参考这个 [EKS入门文档](https://docs.aws.amazon.com/zh\_cn/eks/latest/userguide/getting-started-console.html)

### DevOps用户添加到Kubernetes

所有集群都默认加入了spotmax\_devops角色，该角色信息请找管理员索取。如果遇到权限报错，请反馈给管理员。

```bash
# k edit configmap aws-auth --namespace kube-system
mapUsers: |
    - userarn: arn:aws:iam::xxxxxxxxx:user/spotmax_devops
      username: spotmax_devops
      groups:
        - system:masters
```

### DevOps更新EKS环境到本地config里

这里的config是kubeconfig，亚马逊提供了更新工具，执行

```
aws eks --region <us-east-1替换区域> update-kubeconfig --name <spotmax-prod-vg替换名字>
```

更新完毕后，确认下当前的context是否指向了这个kubernetes，测试下

```
kubectl config get-contexts
```

### DevOps测试下kubectl和helm是否正常

```
kubectl get ns
helm ls
```

不报错表示接入了，遇到错误请和管理员一起trouble shooting

## 阿里云的ACK接入方式

### 环境准备

阿里云比亚马逊要简单一些，因为是DevOps同学不需要具备自己的账户，直接复用管理员的提供的认证授权就可以了。

### 获得配置文本

管理员会将集群的接入配置发到你的手里，类似这样的文本

```
apiVersion: v1
clusters:
- cluster:
    server: https://42.89.182.195:6443
    certificate-authority-data: tLS1CRUdJTiJUSUZJQ0FURS0tLS0tCk1JSURHakNDQWdLZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREErTVNjd0ZBWURWUVFLRXcxaGJHbGkKWVdKaElHTnNiM1ZrTUE4R0ExVUVDaE1JYUdGdVozcG9iM1V4RXpBUkJnTlZCQU1UQ210MVltVnlibVYwWlhNdwpIaGNOTVRreE1qRXhNRFl6TlRJd1doY05Namt4TWpBNE1EWXpOVEl3V2pBK01TY3dGQVlEVlFRS0V3MWhiR2xpCllXSmhJR05zYjNWa01BOEdBMVVFQ2hNSWFHRnVaM3BvYjNVeEV6QVJCZ05WQkFNVENtdDFZbVZ5Ym1WMFpYTXcKZ2dFaU1BMEdDU3FHU0liM0RRRUJBUVVBQTRJQkR3QXdnZ0VLQW9JQkFRQ3VEK0VJTmFWaVJFTlpxblRkQ0VYNgpxQ2phZmh4SzVGQWRyMUw3TFRGdXNDcEJ2UWRsVmJYdG0va2c4L2hSSkZaZWx0cTllallLS1pXWk5BV2hIdFZvCjIySG9TL3FNS1BONEJnWUdjTHFrN0l3M0w1TkEyQmU0UGpqNzR5b3ZQNGl2UFhzMDR3c2J5azRxa1JzQlVrZTEKTDlxK0RBV1AwMzVEeXNKVURrbU43S2pTbzZodzhRbEN3Y3J0Sm5MdVZHN0JPUWRVWCtJTElTMUFXVWZuSllJSwppN3pBLzRIbXNxTlFWY0ppUTNRT1pjdEtza3ZVVHBlRXhqWVMyaG9Ha1pBeUwrUjNUR2JWZGJ6VlM0dWFkcjdECmp4M3BiSzVxcUhqYUR1Z1JUYTRwNDcvUlREOENEbEdwRUI2QTNpaVVWdmRPazF6Ly9FZTZBLzRTNms1ckpXbnYKQWdNQkFBR2pJekFoTUE0R0ExVWREd0VCL3dRRUF3SUNwREFQQmdOVkhSTUJBZjhFQlRBREFRSC9NQTBHQ1NxRwpTSWIzRFFFQkN3VUFBNElCQVFBdEVIamNaMkhOTjc4TUp1VFNVVzVJY0xua3NhejVsU2FkTE4yemtwU1h3SGxTCitOMmdUcGNkK0xzQkwrL0Zpc2RMK0R2MDc2clQrVzYraTFyOWFSVGQySW9RZWFyZ3ZRM1hRbnhkNHlnSFVzcXIKQVB0QTVtR05HZUlGdElVa1RhQ0F1YzVJUzZoRU1IMkVqNUVITHR6SUlYYXppaUE5RG9uakVmS3JObm03cFJjbwo2UVp0VUk3SDQ1eGRxT2pOVWZ5QUpNNVg5ZnF4cTc4VHowRXd5VmVkZ3JMdlBZaXA4ZDZ5RDI4WW5kZHVCeEtvCnoydGU0ZWRycVp2Rk5YRi9WNmhuMTVHU25zQjJ1amZFejIxZ2ZYTUlQWk5BL21GNU5wNkJ3cGx6eDI1RnlqaWgKbTFpTUVTRG1SUFBUeXVteDdndTN3YXZnQ09kQmo4UVVVc2ZKejJvSAotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: "240214365593356334"
  name: 240214365593356334-cc0c3d7a01a4044518df3dfb01e677aec
current-context: 240214365593356334-cc0c3d7a01a4044518df3dfb01e677aec
kind: Config
preferences: {}
users:
- name: "240214365593356334"
  user:
    client-certificate-data: tLS1CRUdJTVJUSUZJQ0FURS0tLS0tCk1JSUMyRENDQWtHZ0F3SUJBZ0lERTJrSE1BMEdDU3FHU0liM0RRRUJDd1VBTUdveEtqQW9CZ05WQkFvVElXTmoKTUdNelpEZGhNREZoTkRBME5EVXhPR1JtTTJSbVlqQXhaVFkzTjJGbFl6RVFNQTRHQTFVRUN4TUhaR1ZtWVhWcwpkREVxTUNnR0ExVUVBeE1oWTJNd1l6TmtOMkV3TVdFME1EUTBOVEU0WkdZelpHWmlNREZsTmpjM1lXVmpNQjRYCkRURTVNVEl4TVRBMk16Z3dNRm9YRFRJeU1USXhNREEyTkRNd05Wb3dTakVWTUJNR0ExVUVDaE1NYzNsemRHVnQKT25WelpYSnpNUWt3QndZRFZRUUxFd0F4SmpBa0JnTlZCQU1USFRJME1ESXhORE0yTlRVNU16TTFOak16TkMweApOVGMyTURRMk5UZzFNSUdmTUEwR0NTcUdTSWIzRFFFQkFRVUFBNEdOQURDQmlRS0JnUURNdmhIVzNGZFNyaTRkCkFvOEhaMkVOOHRuWkZsa0o4Tm0zdUpaTSswZVZzVmJFYWVobGo5TUVTMjJ0T3p2RHQ1bXI3T1R4aUdoaHNrcEUKeFAyTkxnalRTODVzUDJjcWVVMmtVQWUzNFdDRGJhUjV3VnZ3TDBlbHV0L3RsdnpBa1hFQXZtZmNOVjdCbkszOQo4TXd2a2pkdXFsd2lSSFpSMHgzaitxcnQzOGhodndJREFRQUJvNEdyTUlHb01BNEdBMVVkRHdFQi93UUVBd0lICmdEQVRCZ05WSFNVRUREQUtCZ2dyQmdFRkJRY0RBakFNQmdOVkhSTUJBZjhFQWpBQU1Ed0dDQ3NHQVFVRkJ3RUIKQkRBd0xqQXNCZ2dyQmdFRkJRY3dBWVlnYUhSMGNEb3ZMMk5sY25SekxtRmpjeTVoYkdsNWRXNHVZMjl0TDI5agpjM0F3TlFZRFZSMGZCQzR3TERBcW9DaWdKb1lrYUhSMGNEb3ZMMk5sY25SekxtRmpjeTVoYkdsNWRXNHVZMjl0CkwzSnZiM1F1WTNKc01BMEdDU3FHU0liM0RRRUJDd1VBQTRHQkFCVjVITW83aTJ2V1dXbDBlVzYxSUJnaVB2ZW0KcXJCMUNRTEpzNUFFZjdmN2p4dGhwZk50dk1heS9tNnVNdC9NTFFUYy9QUnpxa1g3Mi9HZ0lWTG1ld1hIL3hvcQpnUnJ0a0pNL1RES2hoeDN1UFptNkx0dGdLMlcrRDZRMkJqRStVMk5sNnJrRFBhMnFLQlJ1bXhZa1piUEM3U2JxCnkrWmsxbnhMRHhxOU5QUjEKLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQotLS0tLUJFR0lOIENFUlRJRklDQVRFLS0tLS0KTUlJQy96Q0NBbWlnQXdJQkFnSURFMmp3TUEwR0NTcUdTSWIzRFFFQkN3VUFNR0l4Q3pBSkJnTlZCQVlUQWtOTwpNUkV3RHdZRFZRUUlEQWhhYUdWS2FXRnVaekVSTUE4R0ExVUVCd3dJU0dGdVoxcG9iM1V4RURBT0JnTlZCQW9NCkIwRnNhV0poWW1FeEREQUtCZ05WQkFzTUEwRkRVekVOTUFzR0ExVUVBd3dFY205dmREQWVGdzB4T1RFeU1URXcKTmpJNU1EQmFGdzB6T1RFeU1EWXdOak0wTlRCYU1Hb3hLakFvQmdOVkJBb1RJV05qTUdNelpEZGhNREZoTkRBMApORFV4T0dSbU0yUm1ZakF4WlRZM04yRmxZekVRTUE0R0ExVUVDeE1IWkdWbVlYVnNkREVxTUNnR0ExVUVBeE1oClkyTXdZek5rTjJFd01XRTBNRFEwTlRFNFpHWXpaR1ppTURGbE5qYzNZV1ZqTUlHZk1BMEdDU3FHU0liM0RRRUIKQVFVQUE0R05BRENCaVFLQmdRQzdHVzJ4Y2pUZGJKd2wybzg5RnpOMnM1bVF2bk9sSFhJQXVBK0kzeEpNSm5kQgpLdnFJWldvRllQSjhVVGQvR1NFUmN0eWU1YVFacFdlRlhGZW9PL1h5YWhiSVdYdHdSWE9uV1lTalhvTWw3anhPClpjSFova3pyTjlzVzcyN0FDczM0OXpLemVVb2hraDRyaWZQUE4yR3B1RUhwV0d3RHFmTEJXMzQ1NE9wa2ZRSUQKQVFBQm80RzZNSUczTUE0R0ExVWREd0VCL3dRRUF3SUNyREFQQmdOVkhSTUJBZjhFQlRBREFRSC9NQjhHQTFVZApJd1FZTUJhQUZJVmEvOTBqelNWdldFRnZubTFGT1p0WWZYWC9NRHdHQ0NzR0FRVUZCd0VCQkRBd0xqQXNCZ2dyCkJnRUZCUWN3QVlZZ2FIUjBjRG92TDJObGNuUnpMbUZqY3k1aGJHbDVkVzR1WTI5dEwyOWpjM0F3TlFZRFZSMGYKQkM0d0xEQXFvQ2lnSm9Za2FIUjBjRG92TDJObGNuUnpMbUZqY3k1aGJHbDVkVzR1WTI5dEwzSnZiM1F1WTNKcwpNQTBHQ1NxR1NJYjNEUUVCQ3dVQUE0R0JBQnEyTGVhTXBycURjV2xsbk02ZHhVOGN4VjdvbVE0RUJjTG1xQVp4CmIzSkE0ZEgvUVRPR3ZlODB0eUwya1dmVlU4eXh0V3dnQUYxdkEyS0l2Ni9xNXdVTE1sNi8wcGd4ckN6TGFRS0sKVGJYNHRYdUJwZzVPdDBBRDRIaFFMeko3ZEhScVBMcEZ3UCtyRVErdlltYTZseVppdFNKMGdQYVVLaEJnN3ZYUgplVDRpCi0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
    client-key-data: LS0tLS1CTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlDWEFJQkFBS0JnUURNdmhIVzNGZFNyaTRkQW84SFoyRU44dG5aRmxrSjhObTN1SlpNKzBlVnNWYkVhZWhsCmo5TUVTMjJ0T3p2RHQ1bXI3T1R4aUdoaHNrcEV4UDJOTGdqVFM4NXNQMmNxZVUya1VBZTM0V0NEYmFSNXdWdncKTDBlbHV0L3RsdnpBa1hFQXZtZmNOVjdCbkszOThNd3ZramR1cWx3aVJIWlIweDNqK3FydDM4aGh2d0lEQVFBQgpBb0dBWVF5amRoNWh3ZzVRUzI0QUVEbGZsdllMYjB5WmpnMjlsY21JYlJzYkZvakdJVG8yYjVYYUo0bjloZ1N5CjBwWC80Sy9jNGVTUDNlZGVMdlRWWHd0NElKNmxMSlhyT0RFUGxrV2pLYUx0dnVPd25oOEJUa2F5QlhZYlV0WFIKaW9Wd2VwNUFkS2tJbVl5OUE1UDFTTG51dGpNY04xVEtCak9qTXJwWU82RVNpSWtDUVFEMGtaTStQNXJCU2R1agpGQlduRWNWYU1lQmtRL3dPSHQ4TU5ra2VtYVZ6djZRTmd6Wlg2Y2svR0djb1piMTF2cWxhWmovYnZQb1lxMzFzCnRURDk2WHZ6QWtFQTFrL3lxbUZ6dHR6bXI4cUsrdEZDOUgyM3F3b2w0eXR0cC9DaENub0hhN0VkUjhYcEZ1Qm0KaG8ySS93ZU1KbnNQZzR0Y1FkMmltMXY5cTJtdGVrNnlCUUpBY2NRK01GaTZEbXZqQmN0VC96R2ZFa1BkVkFiagorMVdWQUVOSVpEbW80MTBrWFR6S1RMN3Q1TEhmV3NWcENwcTBnTjdMbWRZZ3FOVXROU0pjTmVFa3pRSkFmYUNWCjNseUw2VWlxamFmTU9tVUt1NmtxVGovL241L29nc2Fpa0RLaVFQV3M5VkxGWlJ5YjNRb0FvWWE2R0NDUklvcEIKeFhaM2lGeXZZWmpzRVVNcVJRSkJBS3FOT3ZVRjFValdPRy9wQmUvZmNCQ05maG1NeXh1MHpvQno0eVU0LzNFOQp1aklKYUhzQjBJcFg1ZTVIbXpFQ0luNVZaaUVlcVRvblIya0lvQ0lFWTVnPQotLS0tLUVORCBSU0EgUFJJVkFURSBLRVktLS0tLQo=

```

示例文本修改过，是假的

### 合并到自己的kubeconfig

拿到文本后是config模式的，需要将内容自己merge到配置文件中。

#### 保存到临时文件

将上述文本存储到文件 \~/.kube/config\_temp

{% hint style="danger" %}
主要修改 cluster/user/context的name，以便于合并，不然前后有相同的key会有问题！
{% endhint %}

#### 执行语句合并

```
KUBECONFIG=~/.kube/config:~/.kube/config_temp kubectl config view --flatten > ~/.kube/config_new
```

#### 检查并覆盖默认文件

仔细检查一下config\_new

```
# cat ~/.kube/config_new
cp ~/.kube/config_new ~/.kube/config
```

## 管理好kubeconfig

{% hint style="info" %}
集群变多时需要管理好自己的配置文件，便于集群间互相切换
{% endhint %}

![](<../../.gitbook/assets/image (193).png>)
