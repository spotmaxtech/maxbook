# 阿里云

## 一、OSS对象存储开启SLS日志服务

在开通的bucket详情中，选择日志管理，添加实时查询，添加立刻开通

![](<../../.gitbook/assets/image (223).png>)

## 二、Saas页面添加日志收集

1、进入[spotmax成本优化](https://manage.spotmaxtech.com/) 点击 **立刻添加** 按钮

**注意！**该功能需要授权两个权限：

**AliyunLogReadOnlyAccess** 用于有权限分析用户日志数据

**AliyunOSSFullAccess** 用于有权限对文件进行层级转换

![](<../../.gitbook/assets/image (48).png>)

2、添加成功后，按钮变成 设置生命周期

![](<../../.gitbook/assets/image (98).png>)

3、对生命周期进行设置

![](<../../.gitbook/assets/image (85).png>)

![](<../../.gitbook/assets/image (13).png>)

![](<../../.gitbook/assets/image (118).png>)

选择完成后，点击开启，自动开启文件层级转换

**注意！存储类型的转换(可多选)：**

**全部：当发现文件可以优化时，按照成本最优的转换层级**

**标准：只允许文件转换 标准 层级**

**低频：只允许文件转换 低频 层级**

**归档：只允许文件转换 归档 层级，当访问时，需要把文件解冷才能访问**

![](<../../.gitbook/assets/image (230).png>)

## 三、注销SLS日志收集

阿里云一个Account ID可以收集每个区域的SLS日志，因此如果A子账号开启了 上海 区域的智能存储，B子账号如果在 上海 区域开启就会提示，已经有子账号开启，如果B账号需要上海区域的智能存储，A账号需要先注销，才能绑定**，**因为子账号A、B同属一个Account ID

![](<../../.gitbook/assets/image (22).png>)
