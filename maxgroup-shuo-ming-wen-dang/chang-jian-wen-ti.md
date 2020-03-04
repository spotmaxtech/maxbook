# 常见问题

所谓preaction，就是查找中断率相对比较低的spot替换中断率高的spot，其实执行的频率是由参数preaction\_snapshot\_interval\_minutes 来决定，例如设置preaction\_snapshot\_interval\_minutes:20 即20分钟执行一次判断是否替换中断率较高的spot

### 配置文件groups中选项解释

| options | explain |
| :--- | :--- |
| detaching\_delay\_seconds | when spot interrupt event triggered,how long you want to wait to detach tis instance,default is 20s |
| max\_num\_of\_terminated\_one\_time | max number your want to terminate instance number |
| preaction\_termination\_delay\_seconds | when precation your asg instance,how long you want to wait to terminate old instance,default is 10 |
| preaction\_detach\_delay\_seconds | when precation your asg instance,how long you want to wait to detach old instance,default is 10 |
| is\_enable\_preaction | precision is enable or not, default is false,this option is false make preaction\_termination\_delay\_seconds and preaction\_detach\_delay\_seconds disable |
| persistence\_dev | if you have and extra EBS don't want delete,use this option can remove EBS to an new startup instance |
| consul\_port | if you use consul,type port number here |
| k8s\_node\_drain\_option | if you use autoscaling for K8S,this option set true |

