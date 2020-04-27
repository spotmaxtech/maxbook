# AWS示例

## 带有2块磁盘的，非root盘需要继续保留的：

![](../../.gitbook/assets/image%20%28102%29.png)

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

![](../../.gitbook/assets/image%20%28101%29.png)

## 需要k8s drain功能：

![](../../.gitbook/assets/image%20%2823%29.png)

## cloudwatch指标监控

cloudwatch集成了max\_group事件指标监控，可根据时间指标查询到max\_group的工作记录，如下图所示

![](../../.gitbook/assets/image%20%283%29.png)



