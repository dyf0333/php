# php-fpm三种进程管理模式 

    ondemand
    static
    dynamic
    
php-fpm进程管理一共有三种模式：ondemand、static、dynamic，我们可以在同一个fpm的master配置三种模式，看下图1。php-fpm的工作模式和nginx类似，都是一个master，多个worker模型。每个worker都在accept本pool内的监听套接字（linux已不存在惊群现象）。

![|center](../master/src/php-fpm-1.png)

### ondemand

> 在php-fpm启动的时候，不会给这个pool启动任何一个worker，是按需启动，当有连接过来才会启动。

配置文件    /usr/local/php/etc/php-fpm.conf

![|center](../master/src/php-fpm-2.png)

#### 原理

![|center](../master/src/php-fpm-3.png)

```
    1. 从上图可以看出，新建worker的触发条件是连接的到来，而不是实际的请求（例如，只进行连接比如telnet，不发请求数据也会新建worker）

    2. worker的数量受限于pm.max_children配置，同时受限全局配置process.max（准确的说，三种模式都受限于全局配置）

    3.1秒定时器作用
          找到空闲worker，如果空闲时间超过pm.process_idle_timeout大小，关闭。这个机制可能会关闭所有的worker。
```
                    
配置项要求
    
    1. pm.max_children> 0
    2. pm.process_idle_timeout> 0，如果不设置，默认10s
    
优点：
>按流量需求创建，不浪费系统资源（在硬件如此便宜的时代，这个优点略显鸡肋）

缺点：
>由于php-fpm是短连接的，所以每次请求都会先建立连接，建立连接的过程必然会触发上图的执行步骤，所以，在大流量的系统上master进程会变得繁忙，占用系统cpu资源，不适合大流量环境的部署



### dynamic

>在php-fpm启动时，会初始启动一些worker，在运行过程中动态调整worker数量，worker的数量受限于pm.max_children配置，同时受限全局配置process.max

![|center](../master/src/php-fpm-4.png)

#### 原理

![|center](../master/src/php-fpm-5.png)

#### 1秒定时器作用

检查空闲worker数量，按照一定策略动态调整worker数量，增加或减少。增加时，worker最大数量<=max_children· <=全局process.max；减少时，只有idle >pm.max_spare_servers时才会关闭一个空闲worker。

idle > pm.max_spare_servers，关闭启动时间最长的一个worker，结束本次处理

idle >= pm.max_children，打印WARNING日志，结束本次处理

idle < pm.max_children，计算一个num值，然后启动num个worker，结束本次处理

#### 配置项要求

1. pm.min_spare_servers/pm.max_spare_servers有效范围(0,pm.max_children]
2. pm.max_children> 0
3. pm.min_spare_servers<=pm.max_spare_servers
4. pm.start_servers有效范围[pm.min_spare_servers,pm.max_spare_servers]如果没有配置，默认pm.min_spare_servers + (pm.max_spare_servers - pm.min_spare_servers) / 2


优点：
>动态扩容，不浪费系统资源，master进程设置的1秒定时器对系统的影响忽略不计；

缺点：
>如果所有worker都在工作，新的请求到来只能等待master在1秒定时器内再新建一个worker，这时可能最长等待1s；

### static

>php-fpm启动采用固定大小数量的worker，在运行期间也不会扩容，虽然也有1秒的定时器，仅限于统计一些状态信息，例如空闲worker个数，活动worker个数，网络连接队列长度等信息。

![|center](../master/src/php-fpm-6.png)


![|center](../master/src/php-fpm-7.png)

配置项要求

1、pm.max_children> 0 必须配置，且只有这一个参数生效


如果配置成static，只需要考虑max_children的数量，数量取决于cpu的个数和应用的响应时间

![|center](../master/src/php-fpm-8.png)

fastcgi与php-fpm的关系一句话解读：fastcgi只是通信应用协议，php-fpm就是实现了fastcig协议，并嵌入了一个 PHP 解释器。


### 一些参数名词解释
```
pm.max_children：静态方式下开启的php-fpm进程数量。
pm.start_servers：动态方式下的起始php-fpm进程数量。
pm.min_spare_servers：动态方式下的最小php-fpm进程数量。
pm.max_spare_servers：动态方式下的最大php-fpm进程数量。
```

事实上，跟Apache一样，运行的PHP程序在执行完成后，或多或少会有内存泄露的问题。
这也是为什么开始的时候一个php-fpm进程只占用3M左右内存，运行一段时间后就会上升到20-30M的原因了。

对于内存大的服务器（比如8G以上）来说，指定静态的max_children实际上更为妥当，因为这样不需要进行额外的进程数目控制，会提高效率。
因为频繁开关php-fpm进程也会有时滞，所以内存够大的情况下开静态效果会更好。数量也可以根据 内存/30M 得到，比如8GB内存可以设置为100，那么php-fpm耗费的内存就能控制在 2G-3G的样子。
如果内存稍微小点，比如1G，那么指定静态的进程数量更加有利于服务器的稳定。这样可以保证php-fpm只获取够用的内存，将不多的内存分配给其他应用去使用，会使系统的运行更加畅通。

对于小内存的服务器来说，比如256M内存的VPS，即使按照一个20M的内存量来算，10个php-cgi进程就将耗掉200M内存，那系统的崩溃就应该很正常了。因此应该尽量地控制php-fpm进程的数量，大体明确其他应用占用的内存后，给它指定一个静态的小数量，会让系统更加平稳一些。或者使用动态方式，因为动态方式会结束掉多余的进程，可以回收释放一些内存，所以推荐在内存较少的服务器或VPS上使用。具体最大数量根据 内存/20M 得到。比如说512M的VPS，建议pm.max_spare_servers设置为20。至于pm.min_spare_servers，则建议根据服务器的负载情况来设置，比较合适的值在5~10之间。