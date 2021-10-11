# 启动maxgroup(以aws Pro版本为例)

### 配置MaxGroup

max_group为已编译完成的二进制包，无系统环境依赖，下载附件并解压，将得到二进制文件maxgroup_pro_\_aws

```
$ ls /data/spotmax/
maxgroup_pro_aws
```

### 启动MaxGroup

启动maxgroup，使用如下命令启动maxgroup。

```
$ ./maxgroup_pro_aws
```

启动成功，会看到以下日志输出

```
time="2020-04-13T17:12:21+08:00" level=info msg="license path -->/data/spotmax/license"
time="2020-04-13T17:12:21+08:00" level=info msg="Connecting to AWS..."
time="2020-04-13T17:12:21+08:00" level=info msg="region --> us-east-1\n"
time="2020-04-13T17:12:21+08:00" level=info msg="Starting max_group server..."
time="2020-04-13T17:12:21+08:00" level=info msg="MaxGroupServer is starting..."
......
```

### 自定义

执行如下命令，可以了解更详细的功能

```
$ ./maxgroup_pro_aws -help
```
