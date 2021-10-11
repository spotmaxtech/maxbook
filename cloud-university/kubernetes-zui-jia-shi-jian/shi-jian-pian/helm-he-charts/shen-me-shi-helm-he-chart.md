# 什么是Helm和Chart？

什么是Helm和Charts？

> ### What is Helm?
>
> Helm helps you manage Kubernetes applications — Helm Charts help you define, install, and upgrade even the most complex Kubernetes application.
>
> Helm帮助你管理Kubernetes的应用，Helm Charts帮助你定义、安装、升级从简单到最复杂的Kubernetes应用。
>
> Charts are easy to create, version, share, and publish — so start using Helm and stop the copy-and-paste.
>
> Charts可以很容易的创建、版本管理、分享、公开，因此从现在开始使用Helm而不要再复制粘贴了。

![](<../../../../.gitbook/assets/image (81).png>)

## 三个组成概念

![](<../../../../.gitbook/assets/image (83).png>)

### chart

Chart是helm包,它包含在Kubernetes集群内部运行应用程序，工具或服务所需的所有资源定义。 可以把它想象成一个Homebrew公式，一个Apt dpkg或一个Yum RPM文件的Kubernetes等价物。

### repostory

存储库是可以收集和共享Chart的地方。 这就像Perl的CPAN档案或Fedora软件包数据库，对于Kubernetes来说是软件包。

### release

Release是发布在k8s集群中运行的chart的实例.一个chart通常可以多次安装到同一个集群中。每次安装都会创建一个新版本，例如一个nginx chart。如果你想在集群运行两个nginx实例,则可以安装该chart两次.每个实例都有自己的release，而每个release又都有属于自己的release名。

### 概念之间的关系

记住这些概念后，我们现在可以像这样解释Helm：

> Helm将chart安装到Kubernetes中，为每个安装创建一个新版本。 要找到新的chart，您可以搜索Helm chart存储库。
