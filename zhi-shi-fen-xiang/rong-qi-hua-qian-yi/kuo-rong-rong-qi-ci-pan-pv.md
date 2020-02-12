# 扩容容器磁盘（PV）

在容器环境扩容磁盘现在变得很简单了，作为DevOps自己就可以完成，做一下实验。

## 确认云商为集群开启了扩容权限

什么意思呢，容器环境是通过api方式调整资源的，假如没有为集群设定好api权限，例如worknode所带有的权限无法扩容磁盘，就会在扩容时报错。

### 阿里云磁盘扩容权限开启

参见[说明文档](https://yq.aliyun.com/articles/724713?spm=a2c4e.11155435.0.0.6dae3312OTDwci)，尤其阿里云提供的这个权限图片

![](../../.gitbook/assets/image%20%2831%29.png)

权限的开启需要集群管理员的协助，DevOps可以反馈给管理员

### 确认StorageClass开启扩容参数

StorageClass也控制了一层是否可以针对pv做扩容，确认如下参数

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: disk
parameters:
  type: cloud_efficiency
  zoneid: us-east-1a,us-east-1b
provisioner: alicloud/disk
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true  # 需要增加这个参数，设置为true
```

如果没有增加这个参数，可以使用命令增加

```yaml
 # kubectl patch sc disk -p '{"allowVolumeExpansion": true}'
```

## 修改pvc执行扩容

有了前面的条件后，执行扩容就很简单了，修改pvc，然后重启pod就好

### 修改pvc

使用patch修改大小即可

```yaml
# kubectl patch pvc pvc-disk -p '{"spec":{"resources":{"requests":{"storage":"30Gi"}}}}'
```

当确认pvc和pv状态以后，删除pod，这样pod会重新mount扩容好的pv





