# Tag与功能

max\_group功能依赖于识别Autoscaling（AWS）或者伸缩组（阿里云）所配置的tag来实现不同的功能，以下为tag key-value及功能解释

| tag-key | tag-value | 功能 |
| :--- | :--- | :--- |
| spotmax:persistence\_dev | /dev/sdf | 添加此tag可以进行ebs的漂移，无默认值，tag-value为新挂载的ebs路径，暂时仅aws平台支持 |
| spotmax:detaching\_delay\_seconds | 20 | 当触发spot回收时，间隔多少秒后，将被回收机器从asg中detach，默认为20秒 |
| spotmax:consul\_port | 8500 | 配置此参数为consul支持，在spot被回收后，consul信息也会删除，无默认值，tag-value为端口号 |
| spotmax:is\_enable\_preaction | true | 增加此tag为开启preaction功能，此功能为预测即将被回收的机器，并提前进行更替机型操作，tag-value为true表示为开启此功能 |
| spotmax:max\_num\_of\_terminated\_one\_time | 2 | preaction功能一次关闭的最大机器数，替换机器执行分批替换，每次替换的最大数量 |
| spotmax:preaction\_termination\_delay\_seconds | 600 | preaction执行terminate间隔时间 |
| spotmax:preaction\_detach\_delay\_seconds | 600 | preaction中，将被替换机器间隔多少秒后，会被detach出asg |
| spotmax:k8s\_node\_drain\_option | true | k8s drain功能，当此tag-key为true时，开启此功能，当node为spot且将要被回收时，会在新起node后，将次node上的pod drain走 |
| spotmax:is\_enable\_od\_fallback | true | 此tag-value为true表示当spot拿不到时，会用on-demand实例补充 |

**注：当启用 spotmax:k8s\_node\_drain\_option 时，建议将spotmax:detaching\_delay\_seconds 的tag-value设置为80-90之间，这样可以在保证新node ready情况下，将pod转移过去。**

