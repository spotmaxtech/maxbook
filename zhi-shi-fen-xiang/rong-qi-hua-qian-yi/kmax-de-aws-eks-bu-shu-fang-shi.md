# KMAX的AWS EKS部署方式

## **创建EKS**

使用控制台创建EKS即可，要注意以下几个内容的编排。

### VPC

vpc的指定需要结合旧系统，保证容器集群与旧集群之间是内网互通的。一个简单的指导是使用相同的vpc。

{% hint style="info" %}
使用相同的vpc是比较稳妥的做法
{% endhint %}

### Subnet

同vpc，子网subnet的选择也尽可能与旧系统相同，这样的好处是减少跨子网带来的跨az的费用问题（亚马逊会对跨az网络传输收费）。

{% hint style="info" %}
使用相同的subnet可以节省潜在的跨az网络费用
{% endhint %}

### IAM Role

#### master使用的role：kmax

master是指容器集群节点，需要事先创建好这些节点所使用的role。

{% hint style="info" %}
这里可以在账户里找到kmax这个iam role
{% endhint %}

![](../../.gitbook/assets/image%20%2849%29.png)

#### worknode使用的role：kmax-worknode

worknode是指容器工作节点，同样需要事先创建好这些节点所使用的role

{% hint style="info" %}
这里可以在账户里找到kmax-worknode这个iam role
{% endhint %}

![](../../.gitbook/assets/image%20%2897%29.png)



### Security Group

安全组需要安排给master、worknode，控制网络访问限制，防范网络攻击。因此也需要事先确认已创建好。

#### spotmax-kmax-master-sg

这个安全组是为master而创建的

#### spotmax-kmax-worknode-sg

这个安全组是为worknode而创建的



## **利用EKS的NodeGroup管理功能创建EC2模版**

### 创建kmax-example-group

这里的模版是指EC2 Launch Template，这里取了个巧，利用NodeGroup来生成模版。

{% hint style="info" %}
EKS的NodeGroup管理功能有个缺陷，无法在template里设置EC2的标签，这样如果我们利用标签统计成本就失效了。所以我们放弃了NodeGroup管理容器节点。
{% endhint %}

![](../../.gitbook/assets/image%20%2846%29.png)

说明：上面的节点组所需大小创建时是最小是1，不能置为0的，这个可以通过后台ASG编辑解决，因为它这里实际上也是启动了一个ASG。

![](../../.gitbook/assets/image%20%2859%29.png)

## 利用example的EC2模版改造自己的模版

### 新模版的命名

找到kmax-example-group使用的模版，使用它为模版创建一个新的模版，例如dsp-8c64g-kmax，其中&lt;dsp表示业务线&gt;-&lt;8c64g表示规格&gt;-&lt;kmax搜索词&gt;。

![](../../.gitbook/assets/image%20%2811%29.png)

{% hint style="danger" %}
关于规格的说明

* 8c64g：节点配置不低于此规格
* small：对照c系列，节点规格不低于2c4g
* medium：对照c系列，节点规格不低于4c8g
* large：对照c系列，节点规格不低于8c16g

规格命名的指导，如果所部署业务需要指定专属、个性化、独享的实例，建议使用8c64g这种详细的规格说明。

而不重要的、测试类的、不需要特定实例的业务可以增加和使用small/medium/large这类规格。
{% endhint %}

### 新模版增加必要的tag

| tag名 | 取值 | 说明 |
| :--- | :--- | :--- |
| team | dsp/adn/3s | 业务线名称 |
| kubernetes.io/cluster/&lt;EKS-NAME&gt; | owned | EKS节点发现 |

{% hint style="success" %}
team标签好理解，这里还有个kubernetes.io的标签，这个标签是必须要打上的，否则EKS不会把节点注册上，尽管有了注册脚本（user-data）

当然也可以在ASG里传递这个标签，在这里的优点是：结合使用max\_group时，比正常走autoscaling打tag快10秒注册进EKS
{% endhint %}

### **修改高级选项的IAM kmax-worknode** <a id="KMAX&#x5E73;&#x53F0;&#x90E8;&#x7F72;&#x4E0E;&#x4F7F;&#x7528;-&#x9AD8;&#x7EA7;&#x9009;&#x9879;&#x4E2D;&#xFF0C;&#x6CE8;&#x610F;&#x9009;&#x62E9;IAM&#x5B9E;&#x4F8B;&#x914D;&#x7F6E;&#x6587;&#x4EF6;&#x4E3A;kmax-worknode"></a>

点开模版的高级选项，修改IAM为我们事先指定好的kmax-worknode。

![](../../.gitbook/assets/image%20%2875%29.png)

{% hint style="danger" %}
这一点很重要，NodeGroup的example模版依照kmax-worknode创建了一个新的iam配置文件，我们不用它的。
{% endhint %}

### 修改用户数据保证kubernetes中的标签

我们使用模版启动的实例在注册时会告诉kubernetes需要给该worknode标记哪些标签，所以我们需要保证节点标签的正确性。

| label | 取值 | 说明 |
| :--- | :--- | :--- |
| node.spotmax/team | dsp | 业务线 |
| node.spotmax/type | spot/ondemand | spot或ondemand |
| node.spotmax/spec | 8c64g | 规格说明 |
| eks.amazonaws.com/nodegroup | dsp-8c64g-kmax | 所属的nodegroup |

这些标签是在模版的user数据中，也就是EC2被拉起后执行的操作。

![](../../.gitbook/assets/image%20%28122%29.png)

{% hint style="info" %}
日常运维会在实例初始化是调整一下登陆用户、SUDO权限相关的脚本，也可以增加上。
{% endhint %}

```yaml
#!/bin/bash 
set -ex 

add_user(){
	groupadd admin
	groupadd dev
	useradd -G admin mobsa
	useradd -G dev mobdev
	id mobdev
	id mobsa
}

setup_key(){
	su - mobsa -c "ssh-keygen  -t rsa -N '' -f ~/.ssh/id_rsa -q -b 2048"
	su - mobsa -c "touch ~/.ssh/authorized_keys"
	su - mobdev -c "ssh-keygen  -t rsa -N '' -f ~/.ssh/id_rsa -q -b 2048"
	su - mobdev -c "touch ~/.ssh/authorized_keys"

	cat > /home/mobdev/.ssh/authorized_keys <<END1
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDlMUO/Z2+VGypRLCE4QAxQLodPQSp941POXcAabH2W8v4n9OTXlpqPRLF+hGNvWOafydxjeVgAjTRfLC8Z5STqMJ6ECFULk7RuWS2D5GqCiVKXSXllU7E3Cx2eipSrKcWIO34vXUDXKtzAgxWkxeF18z//GEzXxMLg5zVjT37vrhd7uClm4vhWX0UZM6bEghYt0akaas54VT6DJ2Trf/0m5Gh6TD6F2RiQv8WajTYpjTX/j0EQWnpiKFqzc39B607ItWIiH7A/cvEA6dieGUloNNmNSmmp1AKsKJxm81cBgzThHpOjQVxjINL1uhzQvOGtL6xl94bq4N3cr4/D0qxX mobdev@ip-172-31-8-162
END1
	cat > /home/mobsa/.ssh/authorized_keys <<END2
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDBJz3tMgLCzfFdJyA/mGd3iYxXBBCqbgUSiD0V2qZJnrCZSCLtatPEFhEm38htpiHzvkclrhfC7XbiU17Kmfq7qD9jEehukr5J7VbaA/6k+kXj+hMs0v1FyXEY+uSPQvP7DyuVnunXhDbrEp5EoS0FfHAjPv3PJ0zmSa5uKavujoVlk2hdJxYR6L4RAdnvYVk23qvNK5HpxayADHkd05aE66VYSZaR13WnIjZUCP8tduS2SLhvgisqrCbdbdf6jsoEVXRSTpirx2gtL1XkuhCLBhS7QOfcjH5BHVvpQkSNOskzs8se2UQuFJeNanAMAQdy95SB4xmQy0+aRZdrplDD mobsa@ip-172-31-8-162
END2
	chown mobsa. /home/mobsa/.ssh/ -R
	chmod 600  /home/mobsa/.ssh/authorized_keys
	chown mobdev. /home/mobdev/.ssh/ -R
	chmod 600  /home/mobdev/.ssh/authorized_keys
}

setup_sudo(){
	chmod 777 /etc/sudoers
	cat >> /etc/sudoers<<END
##### OPS #####
Cmnd_Alias      NSU=/bin/su
Cmnd_Alias      NSHELLS = /bin/sh,/bin/bash
Cmnd_Alias      NCMDS = /usr/sbin/visudo,/usr/bin/chattr,/sbin/fdisk,/bin/dd,/usr/bin/passwd,/usr/sbin/usermod
Cmnd_Alias      DEV = /usr/bin/svn

Defaults:mobsa       !requiretty
User_Alias MOBADMIN = %admin
User_Alias MOBDEV = %dev
MOBADMIN       ALL=(ALL) NOPASSWD: ALL
MOBDEV         ALL=(ALL) ALL,!NSU,!NSHELLS,!NCMDS,NOPASSWD: DEV
END
	chmod 440 /etc/sudoers
}

add_user
setup_key
setup_sudo

```

至此我们准备好了一种规格的模版！之所以称之为一种规格是因为我们需要根据不通的产品拉起不通的节点，因此也需要不同的模版了。

{% hint style="warning" %}
假如使用同一模版拉起不通的ASG，会出现什么问题？

答：因为模版的用户数据是统一的，这样会模版与实例管理混乱问题，例如出现kubernetes标签与实际实例不一致等问题。
{% endhint %}

## 创建AutoscalingGroup

### 选择模版

有了dsp-8c64g-kmax模版，我们就可以创建对应的ASG了，这样每一个节点都能根据用户数据（user data）注册到容器集群里。

### 创建与模版对应的ASG

为了便于区分管理，我们创建与模版一致名称的ASG

{% hint style="info" %}
选择模版&lt;dsp-8c64-kmax&gt;创建伸缩组&lt;dsp-8c64g-kmax&gt;
{% endhint %}

#### **对ASG增加必要的tag**

| **tag** | 取值 | 说明 |
| :--- | :--- | :--- |
| k8s.io/cluster-autoscaler/enabled | true | CA自动发现使用 |
| k8s.io/cluster-autoscaler/&lt;EKS-NAME&gt; | owned | CA自动发现使用 |

至此，创建好的ASG会自动拉取节点，此时使用kubectl客户端可以看到节点已经加入了！

## 启用CA（ClusterAutoscaler）

CA的基本概念与详细说明请参考[说明文档](https://amazonaws-china.com/cn/premiumsupport/knowledge-center/eks-cluster-autoscaler-setup/)

{% hint style="info" %}
这个模块的操作可以和kmax确认后操作
{% endhint %}

### 确保worknode的IAM含有ASG操作权限

如果没有意外，当前worknode所使用的IAM角色已经含有了ASG权限，可以查看到

![](../../.gitbook/assets/image%20%2886%29.png)

### 安装CA

CA即Cluster Autoscaler，是kubernetes社区的组件，支持不同的云商，大家找到自己的云商可以直接安装，这里我们使用的亚马逊的CA。

#### 下载YAML文件

要在 GitHub 上下载 Cluster Autoscaler 项目提供的示例部署文件，请运行以下命令：

```text
wget https://raw.githubusercontent.com/kubernetes/autoscaler/master/cluster-autoscaler/cloudprovider/aws/examples/cluster-autoscaler-autodiscover.yaml
```

#### 修改YAML文件添加ASG组

打开下载的 YAML 文件并根据下例设置 EKS 集群名称 \(**awsExampleClusterName**\) 和环境变量 \(**us-east-1**\)。然后保存更改。

```text
...          
          command:
            - ./cluster-autoscaler
            - --v=4
            - --stderrthreshold=info
            - --cloud-provider=aws
            - --skip-nodes-with-local-storage=false
            - --expander=least-waste
            - --node-group-auto-discovery=asg:tag=k8s.io/cluster-autoscaler/enabled,k8s.io/cluster-autoscaler/awsExampleClusterName
          env:
            - name: AWS_REGION
              value: us-east-1
...
```

#### 使用YAML部署CA

要创建 Cluster Autoscaler 部署，请运行以下命令：

```text
kubectl apply -f cluster-autoscaler-autodiscover.yaml
```

要检查 Cluster Autoscaler 部署日志以查看部署错误，请运行以下命令：

```text
kubectl logs -f deployment/cluster-autoscaler -n kube-system
```

## 启用HPA（Horizontal Pod Autoscaler）

关于HPA的详细介绍可以参考[说明文档](https://docs.aws.amazon.com/zh_cn/eks/latest/userguide/horizontal-pod-autoscaler.html)

所以让HPA可用，我们只需要在集群里安装好Metrics Server即可

### Metrics-Server安装

{% hint style="danger" %}
如果已经集群已经安装过了，这个环节可以省去，例如prometheus也会使用到这个模块。
{% endhint %}

```yaml
helm install stable/metrics-server --generate-name --namespace kube-system
```



