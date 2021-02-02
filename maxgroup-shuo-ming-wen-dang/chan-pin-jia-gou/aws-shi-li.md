# AWS示例

## 带有2块磁盘的，非root盘需要继续保留的：

![](../../.gitbook/assets/image%20%28103%29.png)

由于这里解决的是物理层磁盘挂载，而在系统中，需要对磁盘进行重新mount，这里提供自动化脚本植入系统，帮助完成系统层面的mount。

```text
#!/bin/bash
EC2_REGION=`curl -s http://169.254.169.254/latest/dynamic/instance-identity/document|grep region|awk -F\" '{print $4}'`
INSTANCE_ID="`curl -s http://169.254.169.254/latest/meta-data/instance-id`"
aws ec2 describe-tags --filters "Name=resource-id,Values=$INSTANCE_ID" --region $EC2_REGION --output=text |grep spotmax:group &>/dev/null
if [ $? -ne 0 ] ;then
	exit 1
fi
MOUNT_PATH=$1
OLD_DEV=`lsblk|grep $MOUNT_PATH|awk '{print $1}'`
DEV_TYPE=`lsblk|grep $MOUNT_PATH|awk '{print $6}'`
while true
do
	NEW_DEV=`lsblk|grep $DEV_TYPE |grep -Ev "xvda|${OLD_DEV}|nvme0n1"|awk '{print $1}'`
	if [ ! -z $NEW_DEV ] ;then 
		umount $MOUNT_PATH &>/dev/null 
		sleep 1
		mount /dev/$NEW_DEV $MOUNT_PATH && break
	else
		sleep 3
	fi
done
```

## 需要consul解注册功能：

![](../../.gitbook/assets/image%20%28102%29.png)

## 需要k8s drain功能：

使用k8s用户需要进行如下两步操作：



1.在使用k8s的autoscaling上打上如下标签：

spotmax:k8s\_node\_drain\_option    true

spotmax:detaching\_delay\_seconds    80

![](../../.gitbook/assets/image%20%2823%29.png)

2. 需要给maxgroup所在的IAM role授权EKS的master权限，并导出config，放到/root/.kube/目录下

```text
kubectl describe configmap -n kube-system aws-auth   ###查看当前eks授权情况
kubectl edit -n kube-system configmap/aws-auth       ###编辑授权表，添加权限


# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: v1
data:
  mapRoles: |
    - groups:
      - system:masters
      rolearn: arn:aws:iam::<your aws account>:role/max_group_role
      username: system:node:{{EC2PrivateDNSName}}
      
      
aws eks --region <region-code> update-kubeconfig --name <cluster_name> --kubeconfig /root/.kbue/config
```

## cloudwatch指标监控

cloudwatch集成了max\_group事件指标监控，可根据时间指标查询到max\_group的工作记录，如下图所示

![](../../.gitbook/assets/image%20%283%29.png)



