# php多进程浅析

php运行可分为cli和web两种模式运行
    
在cli模式下多进程，用pcntl扩展进行多进程操作
    

使用多进程的优点:
   
    1. 使用多进程, 子进程结束以后, 内核会负责回收资源
    2. 使用多进程,子进程异常退出不会导致整个进程Thread退出. 父进程还有机会重建流程.
    3. 一个常驻主进程, 只负责任务分发, 逻辑更清楚.


## 1.安装pcntl扩展

pcntl是process control的缩写，通常，php会默认安装这个扩展。使用phpinfo()函数查看扩展是否存在。

## 2.使用pcntl_fork() 函数创建子进程

pcntl_fork作用就是创建和当前进程一样的子进程，这个子进程代码段和当前进程一模一样，但是拥有自己的数据段。

看一个最简单的创建子进程的方法：
```

<?php/**
 *  hedong
 * @date 2017-04-03
 */

$parentPid = getmypid(); // 获取父进程id $childPid = pcntl_fork(); // 创建子进程 switch($childPid) {
    case -1:
        print "创建子进程失败!".PHP_EOL;
        exit;
    case 0:
        print "我是子进程，进程ID：{$childPid}".PHP_EOL;
        break;
    default:
        print "我是父进程，进程ID：{$parentPid},子进程ID: {$childPid}".PHP_EOL;
}
?>
```
>pcntl_fork()
调用成功以后，一个程序变成了两个程序：一个程序得到的$pid变量值是0，它是子进程；另一个程序得到的$pid的值大于0，这个值是子进程的PID，它是父进程。


## 3.子进程回收

1.阻塞方式

用ps aux加上grep命令来查找运行着的后台进程。

其中有一列STAT，标识了每个进程的运行状态。

状态Z：僵尸（Zombie）

当子进程比父进程先退出，而父进程没对其做任何处理的时候，子进程将会变成僵尸进程。

僵尸进程虽然不占什么内存，但是很碍眼。（别忘了它们还占用着PID）

一般来说，在父进程结束之前回收挂掉的子进程就可以了。在pcntl扩展里面有一个pcntl_wait()函数，通过这个方法等待进程结束，然后回收已经结束的进程。
```


<?php
$parentPid = getmypid(); // 获取父进程id $childPid = pcntl_fork(); // 创建子进程 switch($childPid) {
    case -1:
        print "创建子进程失败!".PHP_EOL;
        exit;
    case 0:
        print "我是子进程，进程ID：{$childPid}".PHP_EOL;
        break;
    default:
        pcntl_wait($status); // 子进程执行完后才执行父进程         print "我是父进程，进程ID：{$parentPid},子进程ID: {$childPid}".PHP_EOL;
}
```

2.非阻塞方式

阻塞方式失去了多进程的并行性。

还有一种方法，既可以回收已经结束的子进程，又可以并行。

这就是非阻塞的方式。
```

<?php/**
for ($i = 1; $i <= 5; ++$i) {
    $pid = pcntl_fork(); // 创建子进程

    if (!$pid) {
        sleep(1);
        print "In child $i\n";
        exit($i);
    }
}

// pcntl_waitpid 第一个参数为 0 代表处理全部子进程

while (pcntl_waitpid(0, $status) != -1) {
    $status = pcntl_wexitstatus($status);
    echo "Child $status completed\n";
}
```

如果父进程先挂了怎么办？

子进程依旧还在运行。

但是这个时候，子进程会被交给1号进程，1号进程成为了这些子进程的继父。
1号进程会很好地处理这些进程的资源，当它们结束时1号进程会自动回收资源。

所以，另一种处理僵尸进程的临时办法是关闭它们的父进程。


进程控制不能被应用在Web服务器环境，当其被用于Web服务环境时可能会带来意外的结果。 -- 摘自PHP手册
