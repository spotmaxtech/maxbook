# 配置consul client

迁移过程中，consul的server集群会保持独立，所以需要为kubernetes集群配置client节点，即将kubernetes集群内每个node上都部署一个consul client。

我们操作一下。

## 确认所在集群与区域

```text
kubectx
```

## 创建命名空间

```bash
$ k create namespace consul-client

# 切换
$ kubens consul-client
```

## 下载consul的helm charts

我们使用charts进行发布

```bash
# 请git clone到本地
git clone https://github.com/spotmaxtech/consul-helm.git
```

## 获取consul server的信息

### 注册的data center

> Datacenter is the name of the datacenter that the agents should register as.

```bash
# 可以从现有服务节点初始化程序中找到，如
datacenter: mv-se-consul
```

### join的server ip地址

```bash
# 指向server的其中一个ip地址
fk-consul-ali.rayjump.com
sg-consul-ali.rayjump.com
172.31.4.74
```

## 根据data center信息修改charts

### global enable改成false

我们只需要安装client

```bash
global:
  # enabled is the master enabled switch. Setting this to true or false
  # will enable or disable all the components within this chart by default.
  # Each component can be overridden using the component-specific "enabled"
  # value.
  enabled: false
```

### 指定好data center信息

```bash
# global子项
datacenter: mv-se-consul
```

### 配置好client项

```bash
client:
  enabled: true   # 改成true
  join:
    - 172.31.4.74 # 改成server的ip
```

## 安装consul client charts

```bash
$ helm install consul-client ./
```

## 为consul client增加服务

```bash
$ cd addons
$ k create -f consul-client-svc.yaml
```

## 验证

查看资源

```bash
# 查看资源
$ k get all
NAME                             READY   STATUS    RESTARTS   AGE
pod/consul-client-consul-7gwgz   1/1     Running   0          9d
pod/consul-client-consul-qt8xj   1/1     Running   0          9d

NAME                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/consul-client   ClusterIP   10.100.31.205   <none>        8500/TCP   2m54s

NAME                                  DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/consul-client-consul   2         2         2       2            2           <none>          9d
```

查看consul pod日志，提示很多连接信息表示已经连接上了

```bash
$ k logs consul-client-consul-7gwgz
    2020/01/08 14:27:50 [INFO] serf: attempting reconnect to se_adserver_online:52.78.188.59 172.31.16.87:8301
    2020/01/08 14:28:07 [INFO] serf: EventMemberFailed: se_adserver_rankerv2:13.209.13.28 172.31.2.43
    2020/01/08 14:28:30 [INFO] serf: attempting reconnect to se_adserver_online:54.180.9.218 172.31.8.31:8301
```

访问服务

```bash
kubectl run busybox --rm -i --tty --image busybox -- sh

$ wget -qO- http://consul-client.consul-client.svc.cluster.local:8500/v1/catalog/nodes
```

## FAQ

有问题请留言

