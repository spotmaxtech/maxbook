# AWS

### 配置MaxGroup

max\_group为已编译完成的二进制包，无系统环境依赖，下载附件并解压，将得到二进制文件max\_group和conf目录并包含license.txt、seelog.xml

配置service\_config.json文件，替换配置文件中YOUR-REGION为所使用的region地址，如us-east-2，替换YOUR-AWS-ACCOUT为aws account ID，aws account为12位数字，登录aws console，在我的账号中可以查看账号ID，替换YOUR-ASG-NAME为索要使用的autoscaling名称，如spotmax\_k8s.一下为示例配置文件。

```text
$ cat service_config.json
{
  "system":{

  },
  "global":{
		"interruption_evt_sqs":{
			"queue_url":"https://sqs.us-east-2.amazonaws.com/111111111111/spot-interruption-notice",
			"visibility_timeout_seconds":61,
			"max_number_of_messages":5,
			"wait_time_seconds":1,
      			"preaction_snapshot_interval_minutes":20
		},
		"detach_evt_sqs":{
			"queue_url":"https://sqs.us-east-2.amazonaws.com/111111111111/dettach-event",
			"visibility_timeout_seconds":61,
		  	"max_number_of_messages":5,
		  	"wait_time_seconds":1
		},
    		"termination_evt_sqs":{
      			"queue_url":"https://sqs.us-east-2.amazonaws.com/111111111111/preact-termination",
      			"visibility_timeout_seconds":61,
      			"max_number_of_messages":5,
      			"wait_time_seconds":1
    		},
    		"preaction_snapshot_interval_minutes":0
	},
  "groups":{
    "spotmax_k8s":{
        "detaching_delay_seconds":20,
        "max_num_of_terminated_one_time":2,
      	"preaction_termination_delay_seconds":10,
      	"preaction_detach_delay_seconds": 5,
        "is_enable_preaction":false
    }
  }
}
```

修改完成配置文件，将配置文件service\_config.json和经过spotmax授权的license.txt两个文件，上传至提供的s3 bucket。

```text
$ aws s3 cp service_config.json  s3://YOUR-BUCKET/
$ aws s3 cp license.txt s3://YOUR-BUCKET/
```

### 启动MaxGroup

上传完成，启动max\_group，使用./max\_group可以查看启动所使用的参数

```text
$ ./max_group
region is required.
  -K string
    	k8s config file path
  -b string
    	S3 bucket for storing the MaxGroup config
  -k string
    	[optional] the key of your credential
  -l string
    	[optional] the log file
  -p string
    	[optional] the password of your credential
  -r string
    	the region name
  -t	[optional] local test mode
  -v	current version
```

常规启动方式如下：

```text
$ ./max_group -b YOUR-S3-BUCKET -r ap-southeast-1
```

启动成功，会看到以下日志输出

```text
INFO[0000]main.go:162 main.main() &{{} {{https://sqs.us-east-2.amazonaws.com/111111111111/spot-interruption-notice 61 5 1} {https://sqs.us-east-2.amazonaws.com/111111111111/dettach-event 61 5 1} {https://sqs.us-east-2.amazonaws.com/111111111111/chao-preact-test 61 5 1} 0 false arn:aws:iam::111111111111:role/spotmax_max_group_role 0} map[spotmax-k8s:{ 45  2 10 5 false 0 false false} 
DEBU[0000]autoscaling.go:257 gitlab.mobvista.com/spotmax/max_api/pkg/autoscaling.(*AutoScaling).SuitableForMaxGroup() group spotmax-k8s is suitable for max group 
INFO[0000]main.go:200 main.main() Loaded maxgroup spotmax-k8s
DEBU[0000]main.go:205 main.main() interruption sqs:https://sqs.us-east-2.amazonaws.com/111111111111/spot-interruption-notice 
DEBU[0001]main.go:206 main.main() Detach sqs:https://sqs.us-east-2.amazonaws.com/111111111111/dettach-event                     
INFO[0001]main.go:229 main.main() start instance manager...                    
INFO[0001]main.go:235 main.main() start interruption handler...                
INFO[0001]main.go:241 main.main() start dettach handler...                     
INFO[0001]unified_instance_manager.go:303 gitlab.mobvista.com/spotmax/max_group.(*UnifiedInstanceManager).Process() Delay starting seconds 0                     
INFO[0001]unified_instance_manager.go:306 gitlab.mobvista.com/spotmax/max_group.(*UnifiedInstanceManager).Process() Starting instance attaching ...              
```

