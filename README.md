# php

关于php的相关；

### 如何解决PHP里大量数据循环时内存耗尽的问题
```
之前在研究生期间做的二维码相关项目，就遇到过相关问题，今天看技术帖子，又看到类似的问题及解决方法，特此记录；
利用php非缓存查询方式（默认是缓存查询形式）
```


### php7新特性
```
摒弃与不兼容部分；
新增升级部分；
冲突与新增
```

### 好用的string处理方法
```
    ·生成UUID 单机使用
    ·生成Guid主键
    ·检查字符串是否是UTF8编码
    ·字符串截取，支持中文和其他编码
    ·产生随机字串，可用来自动生成密码(默认长度6位 字母和数字混合 支持中文)
    ·生成一定数量的随机数，并且不重复
    ·带格式生成随机字符 支持批量生成
    ·获取一定范围内的随机数字 位数不足补零
    ·自动转换字符集 支持数组转换
```

### php生命周期


### php-fpm的三种进程管理模式

    ondemand、
    static、
    dynamic
    
### 垃圾回收机制浅析（5.2-5.3版本）

### php多进程浅析
    
    cli模式下多进程
    用pcntl扩展进行多进程操作
    
### php优化，参数配置
    
    通过php的参数配置，达到性能优化的目的
    
### php文件操作

    文件操作函数
    目录操作函数

### curl方法获取htpp码 或者header头信息
    
    利用curl_exec通常获取get或者post数据
    但是获取header数据的比较少，----curl_getinfo
    
### php获取文件后缀的N个方法
    
    获取文件后缀有N方法
    后续继续补充
    但是常用也就是前几个效率比较高的
    
### php的文件下载
        
    1.下载网络资源到本地
    2.下载自己服务器文件到本地
    3.下载网络资源到本服务器
    
    无论如何，基本思路都是修改header为强制下载行为