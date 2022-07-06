# 使用Helm

## helm search

我们使用search查看有没有redis可以直接安装到kubernetes

```shell
$ helm search hub redis
URL                                                     CHART VERSION   APP VERSION             DESCRIPTION                                       
https://artifacthub.io/packages/helm/choerodon/...      16.4.1          6.2.6                   Redis(TM) is an open source, advanced key-value...
https://artifacthub.io/packages/helm/wenerme/redis      16.8.5          6.2.6                   Redis(TM) is an open source, advanced key-value...
https://artifacthub.io/packages/helm/bitnami-ak...      16.8.9          6.2.7                   Redis(TM) is an open source, advanced key-value...
https://artifacthub.io/packages/helm/wener/redis        16.8.5          6.2.6                   Redis(TM) is an open source, advanced key-value...
https://artifacthub.io/packages/helm/pascaliske...      0.0.3           6.2.6                   A Helm chart for Redis                            
https://artifacthub.io/packages/helm/bitnami/redis      16.8.9          6.2.7                   Redis(TM) is an open source, advanced key-value...
```

查到了很多chart可以用来安装redis，来自不同的维护组织。

> * `helm search hub` 搜索的是 [https://artifacthub.io/](https://artifacthub.io/)，这个hub索引了几十个存储库的内容，所以能搜多非常多的chart

{% hint style="info" %}
helm search命令说明
{% endhint %}

```shell
$ helm search

Usage:
  helm search [command]

Available Commands:
  hub         search for charts in the Artifact Hub or your own hub instance
  repo        search repositories for a keyword in charts
```

也可以搜索自己本地管理的存储库，我们添加一个

```bash
# 添加bitnami，一个维护的比较好的扩展库
$ helm repo add bitnami https://charts.bitnami.com/bitnami
"bitnami" has been added to your repositories

$ helm repo list
NAME   	URL
bitnami	https://charts.bitnami.com/bitnami
```

使用repo再搜索一下redis

```bash
$ helm search repo redis

NAME                    CHART VERSION   APP VERSION     DESCRIPTION                                       
bitnami/redis           16.8.9          6.2.7           Redis(TM) is an open source, advanced key-value...
bitnami/redis-cluster   7.5.2           6.2.7           Redis(TM) is an open source, scalable, distribu...
```

使用repo也能查到redis，数量少一些。

> * `helm search repo` 搜索的是本地管理的repositories存储库，helm客户端会从这些库里搜索(添加方法 `helm repo add`). 这个搜索操作读区本地的缓存列表，有必要时可以更新一下`helm repo update`

## helm install

使用helm安装一下redis。

```bash
$ helm install myredis bitnami/redis

NAME: myredis
LAST DEPLOYED: Thu May  5 07:51:13 2022
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: redis
CHART VERSION: 16.8.9
APP VERSION: 6.2.7

** Please be patient while the chart is being deployed **

Redis&trade; can be accessed on the following DNS names from within your cluster:

    myredis-master.default.svc.cluster.local for read/write operations (port 6379)
    myredis-replicas.default.svc.cluster.local for read-only operations (port 6379)



To get your password run:

    export REDIS_PASSWORD=$(kubectl get secret --namespace default myredis -o jsonpath="{.data.redis-password}" | base64 --decode)

To connect to your Redis&trade; server:

1. Run a Redis&trade; pod that you can use as a client:

   kubectl run --namespace default redis-client --restart='Never'  --env REDIS_PASSWORD=$REDIS_PASSWORD  --image docker.io/bitnami/redis:6.2.7-debian-10-r0 --command -- sleep infinity

   Use the following command to attach to the pod:

   kubectl exec --tty -i redis-client \
   --namespace default -- bash

2. Connect using the Redis&trade; CLI:
   REDISCLI_AUTH="$REDIS_PASSWORD" redis-cli -h myredis-master
   REDISCLI_AUTH="$REDIS_PASSWORD" redis-cli -h myredis-replicas

To connect to your database from outside the cluster execute the following commands:

    kubectl port-forward --namespace default svc/myredis-master : &
    REDISCLI_AUTH="$REDIS_PASSWORD" redis-cli -h 127.0.0.1 -p
```

输出了许多信息，有兴趣可以多看几眼，不难理解。

{% hint style="info" %}
其中myredis是发版release的名字，指定不同的名字可以多次发版，他们互相不影响
{% endhint %}

```bash
# list(ls) 列出使用helm管理的发版
$ helm ls
NAME    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
myredis default         1               2022-05-05 07:51:13.977288731 +0000 UTC deployed        redis-16.8.9    6.2.7
```

```bash
# status 会打印安装后的状态，如同一开始install后输出的信息一样
$ helm status myredis

NAME: myredis
LAST DEPLOYED: Thu May  5 07:51:13 2022
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: redis
CHART VERSION: 16.8.9
APP VERSION: 6.2.7
```

这样我们就安装了reids数据库（1个master3个replica）

<img src="../../../../.gitbook/assets/image (206).png" alt="" data-size="original">

{% hint style="info" %}
此时可以关联到maxcloud bundle展示
{% endhint %}

## helm upgrade

当新版本的chart release时，或者当您想要更改release版的配置时，可以使用helm upgrade命令。

{% hint style="info" %}
升级需要现有版本并根据您提供的信息进行升级。 因为Kubernetes chart可能很大而且很复杂，所以Helm会尝试执行最不具有侵入性的升级。 它只会更新自上次发布以来发生更改的内容。
{% endhint %}

我们来做一下升级，只改动一个参数

```yaml
$ helm upgrade myredis bitnami/redis --set image.tag=6.2.6
```

```bash
NAME: myredis
LAST DEPLOYED: Thu May  5 08:16:16 2022
NAMESPACE: default
STATUS: deployed
REVISION: 2
TEST SUITE: None
NOTES:
CHART NAME: redis
CHART VERSION: 16.8.9
APP VERSION: 6.2.7
```

观察helm的输出，REVISION变成了2，同时helm提供了查询values

{% hint style="info" %}
values：是指在install的时候指定的个性化参数，后文的定制化安装会在讲到
{% endhint %}

```bash
# 查看在发版中我们覆盖的参数
$  helm get values myredis
USER-SUPPLIED VALUES:
image:
  tag: 6.2.6
```

好了，我们升级完成了！

![](<../../../../.gitbook/assets/image (207) (1) (1).png>)

## helm rollback

当我们需要回滚时，可以做rollback。回滚前，先看看发版的历史

```bash
$ helm history myredis
REVISION        UPDATED                         STATUS          CHART           APP VERSION     DESCRIPTION     
1               Thu May  5 07:51:13 2022        superseded      redis-16.8.9    6.2.7           Install complete
2               Thu May  5 08:16:16 2022        deployed        redis-16.8.9    6.2.7           Upgrade complete
```

{% hint style="info" %}
helm history \<release-name> 可以查看发版的历史信息
{% endhint %}

有两个发版了，我们回滚到第一个版本

```bash
$ helm rollback myredis 1                                                                                                                                                  1 ↵
Rollback was a success! Happy Helming!

# 验证没有了指定values
$ helm get values myredis
USER-SUPPLIED VALUES:
null
```

{% hint style="info" %}
Usage: helm rollback \<RELEASE>  \[REVISION] \[flags]

注意需要指定版本
{% endhint %}

这样我们完成了回滚！

## helm uninstall

现在我们要卸载掉这个发版

```bash
$ helm uninstall myredis                                                                                                                                                   1 ↵
release "myredis" uninstalled

$ helm ls
NAME	NAMESPACE	REVISION	UPDATED	STATUS	CHART	APP VERSION
```

能看到已经完全卸载掉了！

## helm repo

有时候我们想要用的chart没有在默认的repository存储库里，这个时候需要添加个性化的存储库.

### helm repo list &#x20;

列出所有的存储库列表

```bash
$ helm repo list
NAME   	URL
bitnami	https://charts.bitnami.com/bitnami
```

### helm repo add&#x20;

添加存储库

```bash
$ helm repo add bitnami https://charts.bitnami.com/bitnami
```

### helm repo remove

删除存储库

```bash
$ helm repo remove bitnami
"bitnami" has been removed from your repositories
```

### helm repo update

更新本地缓存（metadata）

```bash
$ helm repo update                                                                                                                                                             1 ↵
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "bitnami" chart repository
Update Complete. ⎈ Happy Helming!⎈
```

## 使用参数定制化安装helm install

前面有提到使用values机制可以给一些开源的chart修改参数，按照我们实际的环境安装这个chart。上面的myredis已经用过一次，这里再补充一下。

```bash
# 使用show values来查看都有哪些参数可以修改
$ helm show values bitnami/redis
... ...
## @section Redis&trade; Sentinel configuration parameters
##

sentinel:
  ## @param sentinel.enabled Use Redis&trade; Sentinel on Redis&trade; pods.
  ## IMPORTANT: this will disable the master and replicas services and
  ## create a single Redis&trade; service exposing both the Redis and Sentinel ports
  ##
  enabled: false
  ## Bitnami Redis&trade; Sentinel image version
  ## ref: https://hub.docker.com/r/bitnami/redis-sentinel/tags/
  ## @param sentinel.image.registry Redis&trade; Sentinel image registry
  ## @param sentinel.image.repository Redis&trade; Sentinel image repository
  ## @param sentinel.image.tag Redis&trade; Sentinel image tag (immutable tags are recommended)
  ## @param sentinel.image.pullPolicy Redis&trade; Sentinel image pull policy
  ## @param sentinel.image.pullSecrets Redis&trade; Sentinel image pull secrets
  ## @param sentinel.image.debug Enable image debug mode
  ##
  image:
    registry: docker.io
    repository: bitnami/redis-sentinel
    tag: 6.2.7-debian-10-r0
... ...
```

能看到非常多的参数，这时候可以采用myredis.yaml模式：

```bash
# myredis.yaml
image: 
    tag: 6.2.6
```

{% hint style="info" %}
覆盖参数myredis.yaml的操作留作练习
{% endhint %}
