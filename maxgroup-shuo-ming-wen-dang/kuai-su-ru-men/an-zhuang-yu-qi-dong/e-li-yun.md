# 阿里云

### 配置MaxGroup

max\_group为已编译完成的二进制包，无系统环境依赖，下载附件并解压，将得到二进制文件max\_group和conf目录并包含license.txt

配置service\_config.json文件，替换配置文件中region地址，如cn-hongkong，替换interruption\_evt\_sqs地址，该地址为MNS（消息服务）接收spot中断的事件地址，queue\_name为接收的事件名称，detach\_evt\_sqs同理，用于接收spot detach事件，替换YOUR-ASG-NAME为使用的伸缩组ID，例如：asg-bp1etmq6p74vtgsdtx00。以下为示例配置文件。

```text
{
  "system": {
    "region": "cn-hongkong",
    "access_keyid": "please override",
    "access_secret": "please override"
  },
  "global": {
    "interruption_evt_sqs": {
      "queue_url": "http://1111111111111111.mns.cn-hangzhou.aliyuncs.com/",
      "queue_endpoint": "http://1111111111111111.mns.cn-hangzhou.aliyuncs.com/",
      "queue_name": "spotmax-interruption-notice",
      "visibility_timeout_seconds": 61,
      "max_number_of_messages": 5,
      "wait_time_seconds": 1,
      "preaction_snapshot_interval_minutes": 20
    },
    "detach_evt_sqs": {
      "queue_url": "http://1111111111111111.mns.cn-hangzhou.aliyuncs.com/",
      "queue_endpoint": "http://1111111111111111.mns.cn-hangzhou.aliyuncs.com/",
      "queue_name": "spotmax-detach-notice",
      "visibility_timeout_seconds": 61,
      "max_number_of_messages": 5,
      "wait_time_seconds": 1
    },
    "termination_evt_sqs": {
      "queue_url": "",
      "visibility_timeout_seconds": 61,
      "max_number_of_messages": 5,
      "wait_time_seconds": 1
    },
    "preaction_snapshot_interval_minutes": 20
  },
  "groups": {
    "YOUR-ASG-NAME": {
      "detaching_delay_seconds": 20,
      "max_num_of_terminated_one_time": 1,
      "preaction_termination_delay_seconds": 20,
      "preaction_detach_delay_seconds": 10,
      "k8s_node_drain_option": true,
      "k8s_node_drain_grace_second": 120
    }
  }
}
```

修改完成配置文件，将配置文件service\_config.json上传至阿里云的OSS，OSS的endpoint和bucket，要与license位置相同，例如：oss-us-east-1.aliyuncs.com\|spotmax-xxxx

![](../../../.gitbook/assets/image%20%2871%29.png)

### 启动MaxGroup

上传完成，启动max\_group，使用./ali\_maxgroup可以查看启动所使用的参数

```text
$ ./ali_maxgroup
-L string
        The license file path
```

常规启动方式如下：

```text
$ ./ali_maxgroup -L ./conf/license.txt
```

启动成功，会看到以下日志输出:

```text
INFO[0001] group asg-xxxxxxxx is registered 
INFO[0001] start dettach handler...                     
INFO[0001] server started                               
INFO[0001] controller started                           
INFO[0001] global message receiver started              
INFO[0001] region us-east-1 instance manager started    
INFO[0001] Delay starting seconds 6                     
INFO[0007] Starting instance attaching ...    
```

