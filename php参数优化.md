# php参数优化

## 1.PHP函数禁用

disable_functions

需注意:如果您的服务器中含有一些CentOS系统状态检测的PHP程序,
则不要禁用
例如：
>shell_exec

>proc_open

>proc_get_status

等函数

## 2 max_execution_time 

php脚本执行相应时间，默认30s

## 3 memory_limit 

php脚本占用最大内存，默认8m
不宜过高，考虑到高并发

## 4 upload_max_filesize 

文件上传限制

文件上传基本都会考虑，特别是大文件上传

前端考虑百度开源的webuploader
>http://fex.baidu.com/webuploader/

## 5.display_errors

报错的配置

相当于开启debug

## 6.magic_quotes_gpc

魔术方法转译

## 7.php 缓存优化

opcache
apc

牵扯到php缓存层，会另开一个文章讨论opcode相关


## 8 php-fpm相关

### 8.1.监听

listion = /tmp/php-cgi.sock

原本监听 127.0.0.1:9000 

sock 比 端口快

### 8.2.文件描述符：

rlimit_files = 1024

文件描述符可与linux的IO select、poll、epoll相关的综合考虑

### 8.3 php-fpm进程管理方式：

详细参考本人github
>https://github.com/dyf0333/php/blob/master/php-fpm%E4%B8%89%E7%A7%8D%E8%BF%9B%E7%A8%8B%E7%AE%A1%E7%90%86%E6%A8%A1%E5%BC%8F.md

静态（节约开销）
```
pm = static
pm.max_children = 24     静态方式下开启的php-fpm进程数量
```
动态
```
pm = dynamic
pm.start_servers = 16      动态方式下的起始php-fpm进程数量
pm.min_spare_servers = 12     动态方式下的最小php-fpm进程数
pm.max_spare_servers = 24     动态方式下的最大php-fpm进程数量
```
