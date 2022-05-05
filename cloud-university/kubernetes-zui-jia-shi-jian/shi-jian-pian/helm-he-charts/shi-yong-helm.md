# 使用Helm

## helm search

我们使用search查看有没有redis可以直接安装到kubernetes

```
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

> * `helm search hub` 搜索的是 [https://artifacthub.io/](https://artifacthub.io)，这个hub索引了几十个存储库的内容，所以能搜多非常多的chart

{% hint style="info" %}
helm search命令说明
{% endhint %}

```
$ helm search

Usage:
  helm search [command]

Available Commands:
  hub         search for charts in the Artifact Hub or your own hub instance
  repo        search repositories for a keyword in charts
```

也可以搜索自己本地管理的存储库，我们添加一个

```
# 添加bitnami，一个维护的比较好的扩展库
$ helm repo add bitnami https://charts.bitnami.com/bitnami
"bitnami" has been added to your repositories

$ helm repo list
NAME   	URL
bitnami	https://charts.bitnami.com/bitnami
```

使用repo再搜索一下redis

```
$ helm search repo redis

NAME                    CHART VERSION   APP VERSION     DESCRIPTION                                       
bitnami/redis           16.8.9          6.2.7           Redis(TM) is an open source, advanced key-value...
bitnami/redis-cluster   7.5.2           6.2.7           Redis(TM) is an open source, scalable, distribu...
```

使用repo也能查到redis，数量少一些。

> * `helm search repo` 搜索的是本地管理的repositories存储库，helm客户端会从这些库里搜索(添加方法 `helm repo add`). 这个搜索操作读区本地的缓存列表，有必要时可以更新一下`helm repo update`

## helm install

使用helm安装一下mysql，现在社区版本是mariadb了。

```
$ helm install <happy-panda这里可以改名字> stable/mariadb                                                                                                                                    130 ↵
NAME: happy-panda
LAST DEPLOYED: Mon Feb 10 16:28:14 2020
NAMESPACE: liuzongxian
STATUS: deployed
REVISION: 1
NOTES:
Please be patient while the chart is being deployed

Tip:

  Watch the deployment status using the command: kubectl get pods -w --namespace liuzongxian -l release=happy-panda

Services:

  echo Master: happy-panda-mariadb.liuzongxian.svc.cluster.local:3306
  echo Slave:  happy-panda-mariadb-slave.liuzongxian.svc.cluster.local:3306

Administrator credentials:

  Username: root
  Password : $(kubectl get secret --namespace liuzongxian happy-panda-mariadb -o jsonpath="{.data.mariadb-root-password}" | base64 --decode)
```

输出了许多信息，有兴趣可以多看几眼，不难理解。

{% hint style="info" %}
其中happy-panda是发版release的名字，指定不同的名字可以多次发版，他们互相不影响
{% endhint %}

```bash
# list(ls) 列出使用helm管理的发版
$ helm ls
NAME       	NAMESPACE  	REVISION	UPDATED                             	STATUS  	CHART        	APP VERSION
happy-panda	liuzongxian	1       	2020-02-10 16:28:14.695377 +0800 CST	deployed	mariadb-7.3.1	10.3.21
```

```bash
# status 会打印安装后的状态，如同一开始install后输出的信息一样
$ helm status happy-panda
NAME: happy-panda
LAST DEPLOYED: Mon Feb 10 16:28:14 2020
NAMESPACE: liuzongxian
STATUS: deployed
REVISION: 1
NOTES:
Please be patient while the chart is being deployed
```

这样我们就安装了mysql数据库

## helm upgrade

当新版本的chart release时，或者当您想要更改release版的配置时，可以使用helm upgrade命令。

{% hint style="info" %}
升级需要现有版本并根据您提供的信息进行升级。 因为Kubernetes chart可能很大而且很复杂，所以Helm会尝试执行最不具有侵入性的升级。 它只会更新自上次发布以来发生更改的内容。
{% endhint %}

我们来做一下升级，使用这个yaml，只改动一个参数

```yaml
# happy-panda.yaml
mariadbUser: user1
```

```bash
# 升级指令
$ helm upgrade -f happy-panda.yaml happy-panda stable/mariadb
Release "happy-panda" has been upgraded. Happy Helming!
NAME: happy-panda
LAST DEPLOYED: Mon Feb 10 21:07:36 2020
NAMESPACE: liuzongxian
STATUS: deployed
REVISION: 2
NOTES:
Please be patient while the chart is being deployed
```

观察helm的输出，REVISION变成了2，同时helm提供了查询values

{% hint style="info" %}
values：是指在install的时候指定的个性化参数，后文的定制化安装会在讲到
{% endhint %}

```bash
# 查看在发版中我们覆盖的参数
$  helm get values happy-panda
USER-SUPPLIED VALUES:
mariadbUser: user1
```

好了，我们升级完成了！

## helm rollback

当我们需要回滚时，可以做rollback。回滚前，先看看发版的历史

```bash
$ helm history happy-panda
REVISION	UPDATED                 	STATUS    	CHART        	APP VERSION	DESCRIPTION
1       	Mon Feb 10 16:28:14 2020	superseded	mariadb-7.3.1	10.3.21    	Install complete
2       	Mon Feb 10 21:07:36 2020	deployed  	mariadb-7.3.1	10.3.21    	Upgrade complete
```

{% hint style="info" %}
helm history \<release-name> 可以查看发版的历史信息
{% endhint %}

有两个发版了，我们回滚到第一个版本

```bash
$ helm rollback happy-panda 1                                                                                                                                                  1 ↵
Rollback was a success! Happy Helming!

# 验证没有了指定values
$ helm get values happy-panda
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
$ helm uninstall happy-panda                                                                                                                                                   1 ↵
release "happy-panda" uninstalled

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
stable 	https://kubernetes-charts.storage.googleapis.com/
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
...Successfully got an update from the "stable" chart repository
...Successfully got an update from the "bitnami" chart repository
Update Complete. ⎈ Happy Helming!⎈
```

## 使用参数定制化安装helm install

前面有提到使用values机制可以给一些开源的chart修改参数，按照我们实际的环境安装这个chart。上面的happy-panda已经用过一次，这里再补充一下。

```bash
# 使用show values来查看都有哪些参数可以修改
$ helm show values stable/mariadb
... ...
image:
  registry: docker.io
  repository: bitnami/mariadb
  tag: 10.3.22-debian-10-r0
... ...
```

能看到非常多的参数，这时候可以采用如上面的happy-panda.yaml模式：

```bash
# happy-panda.yaml
mariadbUser: user1
```

{% hint style="info" %}
覆盖参数有一些较复杂点的方式，如--set指令，留作扩展练习
{% endhint %}
