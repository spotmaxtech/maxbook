# 最佳实践

## 磁盘漂移



由于spot instance被竞走后，源instance包括其自带的ebs也会被回收掉，因此max\_group支持额外挂载的ebs迁移到新instance功能，以下为推荐改造方案。

#### 创建带两块磁盘镜像

新创建一个镜像，镜像要求有两个EBS，一个根卷，一个外挂附属卷。

![](../../.gitbook/assets/image%20%2889%29.png)

系统中，将另一块盘mount到指定目录，示例如下：

```text
#使用的附属卷如是从指定snapshot创建，则已经进行过格式化，可以直接使用，如果未从任意snapshot创建，则需要先进行格式化
$ lsblk
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
xvda    202:0    0  100G  0 disk 
└─xvda1 202:1    0  100G  0 part /
xvdf    202:80   0  400G  0 disk /mnt
```

推荐自定义一个初始化脚本，放置在s3，在系统中定义reboot 计划任务，拉去s3上初始化脚本执行初始化，这样可以在系统有更新是通过初始化脚本更新而无需重做镜像。

```text
$ crontab -l
......
@reboot  aws s3 cp s3://YOUR-INIT-SCRIPT-BUCKET/init_script /opt/ && cd /opt/init_script && sh init_script.sh
......
```

#### 基于新镜像制作模板并创建autoscaling

镜像制作完成后，基于两个EBS镜像，制作启动模板，制作方法跟教程开始制作模板相同。完成后，基于新镜像创建或修改autoscaling使用新的启动模板。

**注意：max\_group只能进行aws层面磁盘转移，系统内部的mount无法进行，这里推荐使用模板中的高级选项的data来植入一个脚本实现mount转移。这里采用了 tag功能，因此给模板内实例需要有get tag的IAM role。**

![](../../.gitbook/assets/image%20%2840%29.png)

![](../../.gitbook/assets/image%20%2844%29.png)

```text
#!/bin/bash
EC2_REGION=`curl -s http://169.254.169.254/latest/dynamic/instance-identity/document|grep region|awk -F\" '{print $4}'`
INSTANCE_ID="`curl -s http://169.254.169.254/latest/meta-data/instance-id`"
TARGET_DIR="/data"
aws ec2 describe-tags --filters "Name=resource-id,Values=$INSTANCE_ID" --region $EC2_REGION --output=text|grep spotmax:group &>/dev/null
if [ $? -eq 0 ] ;then
    while true
    do
        sleep 1
        lsblk |grep xvdz &>/dev/null
        if [ $? -eq 0 ] ;then
            df -h |grep $TARGET_DIR &>/dev/null
            if [ $? -eq 0 ] ;then
                umount $TARGET_DIR
            else 
                mount /dev/xvdz1 $TARGET_DIR
                break
            fi
        fi
    done
fi
```

#### 修改max\_group配置并上传s3后重启服务

修改配置文件service\_config.json中groups部分,增加**persistence\_dev**选项，例如附属卷为/dev/sdf,修改配置如下

```text
 ......
 "groups":{
    "YOUR-ASG-NAME":{
	    "dettaching_delay_seconds":20,
        "max_num_of_terminated_one_time":2,
        "preaction_termination_delay_seconds":10,
        "preaction_detach_delay_seconds": 5,
        "is_enable_preaction":false,
        "persistence_dev":"/dev/sdf"
  }
}
```

修改完成后，将配置上传到s3，并重启服务。

重启后日志含有persistence volume内容

```text
......
time="2019-09-17T12:05:15Z" level=info msg="Has persistence volume." func="main.main()" file="main.go:216"
......
```

## 节点漂移

EKS用户基于autoscaling也可以使用max\_group应对spot被竞走情况，spot被竞走前2分钟会有事件发出，需要在事件发出后新起一个node，并把将要被竞走的node上的pod驱赶到新node上来，实现无缝切换。EKS部署部分不再进行详细说明，如对EKS感兴趣请查看AWS官方文档。

修改配置文件service\_config.json中groups部分，增加**k8s\_node\_drain\_option**选项，如下：

```
......
  "groups":{
    "your autoscaling name":{
	    "dettaching_delay_seconds":20,
        "max_num_of_terminated_one_time":2,
        "preaction_termination_delay_seconds":10,
        "preaction_detach_delay_seconds": 5,
        "is_enable_preaction":false,
        "k8s_node_drain_option":true
  }
}
```

修改完成后，将配置上传到s3，并重启服务。

## Consul支持

使用consul服务，可以对max\_group管理机暴露服务端口，max\_group可以通过暴露的端口将被竞走instance注销。

修改配置文件service\_config.json中groups部分，增加**consul\_port**选项，如暴露的端口为8500，修改配置如下：

```text
......
  "groups":{
    "your autoscaling name":{
	    "dettaching_delay_seconds":20,
        "max_num_of_terminated_one_time":2,
        "preaction_termination_delay_seconds":10,
        "preaction_detach_delay_seconds": 5,
        "is_enable_preaction":false,
        "consul_port":"8500"
  }
}
```

修改完成后，将配置上传到s3，并重启服务。

