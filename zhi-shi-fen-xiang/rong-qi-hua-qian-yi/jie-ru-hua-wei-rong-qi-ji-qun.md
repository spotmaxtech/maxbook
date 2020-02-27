# 接入华为容器集群

## 获取配置

华为集群接入方式也是直接使用kubeconfig，管理员可从console下载，如图示：

![](../../.gitbook/assets/image%20%2854%29.png)

## 修改配置

默认提供了内网、外网两个网络接入地址，这里我们使用外网接入，所以在合并到我们默认配置文件时，可以先删除internalCluster方式

![](../../.gitbook/assets/image%20%286%29.png)

## 合并技巧

上述配置文件是json格式，但kubectl也是支持的，可以直接合并到一个临时文件里，如语句

```text
KUBECONFIG=~/.kube/config:~/.kube/config_hwi kubectl config view --flatten > ~/.kube/config_new
```

测试一下新文件的可用性

```text
KUBECONFIG=~/.kube/config_new kubectl get node
```



