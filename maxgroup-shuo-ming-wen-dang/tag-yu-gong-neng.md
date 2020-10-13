# Tag与功能

max\_group功能依赖于识别Autoscaling（AWS）或者伸缩组（阿里云）所配置的tag来实现不同的功能，以下为tag key-value及功能解释

| tag-key | tag-value | 功能 | 版本支持 |
| :---: | :---: | :---: | :---: |
| spotmax:detaching\_delay\_seconds | 30 | 当触发spot回收时，间隔多少秒后，将被回收机器从asg中detach，默认为30秒 | Lite/Pro |
| spotmax:is\_enable\_preaction | true | 增加此tag为开启集群防退化功能，此功能为预测即将被回收的机器，并提前进行更替机型操作，tag-value为true表示为开启此功能 | Lite/Pro |
| spotmax:max\_num\_of\_terminated\_one\_time | 2 | 集群防退化功能一次关闭的最大机器数，替换机器执行分批替换，每次替换的最大数量 | Lite/Pro |
| spotmax:preaction\_termination\_delay\_seconds | 600 | 集群防退化功能执行terminate间隔时间 | Lite/Pro |
| spotmax:preaction\_detach\_delay\_seconds | 600 | 集群防退化功能中，将被替换机器间隔多少秒后，会被detach出asg | Lite/Pro |
| spotmax:is\_enable\_od\_fallback | true | 此tag-value为true表示，在前述中断预补偿机制中，当竞价实例无法获取时，会用按需实例补充 | Lite/Pro |
| spotmax:persistence\_dev | /dev/sdf | 添加此tag可以进行ebs的漂移，无默认值，tag-value为非root盘在instance上的映射路径，暂时仅aws平台支持 | Pro |
| spotmax:consul\_port | 8500 | 配置此参数为consul支持，无默认值，tag-value为consul agent本地端口号 在实例中断并经过detaching\_delay\_seconds时间后，该实例将会从consul的服务发现列表中移除 | Pro |
| spotmax:k8s\_node\_drain\_option | true | kubernetes pod预迁移功能，当此tag-key为true时，开启此功能，当node为spot且将要被回收时，会在新起node后，并将被中断node上的pod迁移 | Pro |
| spotmax:spot\_price\_limit | 0.01 - 0.99 | spot价格限制，例如 0.9， 当spot机型价格超过按需机型价格的90%，从替换机型列表中移出这个机型 |  ali Lite/Pro |

**注：当启用 spotmax:k8s\_node\_drain\_option 时，建议将spotmax:detaching\_delay\_seconds 的tag-value设置为80-90之间，这样可以在保证新node ready情况下，将pod转移过去。**

