# 创建屏蔽云商的StorageClass

每个云商有自己的StorageClass，为了便于yaml描述跨云有效，我们创建自己的class。

## AWS的StorageClass调整

### 取消gp2的默认

默认的是叫做gp2

```text
$ k get sc
NAME             PROVISIONER             AGE
gp2 (default)    kubernetes.io/aws-ebs   13h
```

通过取消它的annotation取消默认

```yaml
$ k edit storageclass gp2
# 编辑处设置storageclass.kubernetes.io/is-default-class为false
```

保存退出即可

### 增加disk并设置默认

```yaml
# cat aws-disk.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: disk
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
parameters:
  fsType: ext4
  type: gp2
provisioner: kubernetes.io/aws-ebs
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
```

创建这个class

### 最终效果

```yaml
$ k get sc
NAME             PROVISIONER             AGE
disk (default)   kubernetes.io/aws-ebs   5m12s
gp2              kubernetes.io/aws-ebs   13h
```

## 阿里云的StorageClass调整

### 确认集群节点的可用区

阿里云要求在class中指定节点都含有哪些可用区，如果不指明可用区，对应节点可能会获取不到磁盘。这点不如aws智能一些。

{% hint style="danger" %}
阿里云要求在storageclass中指定可用区
{% endhint %}

### 增加disk并设置默认

```yaml
# cat ali-disk.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: disk
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
parameters:
  type: cloud_efficiency
  # zoneid: us-east-1a,us-east-1b # 填写正确可用区
provisioner: alicloud/disk
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
```

### 最终效果

```yaml
$ k get sc                                                                                                                                                                   130 ↵
NAME                     PROVISIONER     AGE
alicloud-disk-essd       alicloud/disk   35d
alicloud-disk-ssd        alicloud/disk   35d
available-delete-wait    alicloud/disk   30d
disk (default)           alicloud/disk   12d
efficiency-delete-wait   alicloud/disk   30d
```



## 磁盘storageclass的选择

大家没有特殊需要请使用“disk“，因为是默认，不指定class也会采用“disk“；如果有类似ssd的需求可以依照上述自己创建，或是寻求管理员的协助。



