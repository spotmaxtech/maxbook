# 启动max\_group

### 配置MaxGroup

max\_group为已编译完成的二进制包，无系统环境依赖，下载附件并解压，将得到二进制文件max\_group和license文件放在同一目录



```text
$ ls /data/spotmax/
max_group license
```

### 启动MaxGroup

启动max\_group，使用./max\_group即可启动max\_group。

```text
$ ./max_group
```

启动成功，会看到以下日志输出

```text
time="2020-04-13T17:12:21+08:00" level=info msg="license path -->/data/spotmax/license"
time="2020-04-13T17:12:21+08:00" level=info msg="Connecting to AWS..."
time="2020-04-13T17:12:21+08:00" level=info msg="region --> us-east-1\n"
time="2020-04-13T17:12:21+08:00" level=info msg="Starting max_group server..."
time="2020-04-13T17:12:21+08:00" level=info msg="MaxGroupServer is starting..."
......
```

