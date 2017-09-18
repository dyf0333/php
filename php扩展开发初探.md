# php扩展开发初探

>ext_skel

>phpize

>php-config

后面两个脚本主要配合autoconf、automake生成Makefile

## 1 ext_skel

这个脚本位于PHP源码/ext目录下，它的作用是用来生成扩展的基本骨架，帮助开发者快速生成一个规范的扩展结构，可以通过以下命令生成一个扩展结构：

>./ext_skel --extname=扩展名称

执行完以后会在ext目录下新生成一个扩展目录，比如extname是mytest，则将生成以下文件：

```
|---mytest 
|   |---config.m4     //autoconf规则的编译配置文件
|   |---config.w32    //windows环境的配置
|   |---CREDITS
|   |---EXPERIMENTAL
|   |---include       //依赖库的include头文件，可以不用
|   |---mytest.c      //扩展源码
|   |---php_mytest.h  //头文件
|   |---mytest.php    //用于在PHP中测试扩展是否可用，可以不用
|   |---tests         //测试用例，执行make test时将执行、验证这些用例
|       |---001.phpt
```

这个脚本主要生成了编译需要的配置以及扩展的基本结构，初步生成的这个扩展可以成功的编译、安装、使用，实际开发中我们可以使用这个脚本生成一个基本结构，然后根据具体的需要逐步完善。


```


To use your new extension, you will have to execute the following steps:

1.  $ cd ..
2.  $ vi ext/dyftest/config.m4
3.  $ ./buildconf
4.  $ ./configure --[with|enable]-dyftest
5.  $ make
6.  $ ./sapi/cli/php -f ext/dyftest/dyftest.php
7.  $ vi ext/dyftest/dyftest.c
8.  $ make

Repeat steps 3-6 until you are satisfied with ext/dyftest/config.m4 and
step 6 confirms that your module is compiled into PHP. Then, start writing
code and repeat the last two steps as often as necessary.
```

## 2.php-config

这个脚本为PHP源码中的/script/php-config.in，PHP安装后被移到安装路径的/bin目录下，并重命名为php-config，这个脚本主要是获取PHP的安装信息的，

主要有：
```
	* PHP安装路径
	* PHP版本
	* PHP源码的头文件目录： main、Zend、ext、TSRM中的头文件，编写扩展时会用到这些头文件，这些头文件保存在PHP安装位置/include/php目录下
	* LDFLAGS： 外部库路径，比如：-L/usr/bib -L/usr/local/lib
	* 依赖的外部库： 告诉编译器要链接哪些文件，-lcrypt -lresolv -lcrypt等等
	* 扩展存放目录： 扩展.so保存位置，安装扩展make install时将安装到此路径下
	* 编译的SAPI： 如cli、fpm、cgi等
	* PHP编译参数： 执行./configure时带的参数
	* ...

```

这个脚本在编译扩展时会用到，执行./configure --with-php-config=xxx生成Makefile时作为参数传入即可，它的作用是提供给configure.in获取上面几个配置，生成Makefile。



## 概述：编写扩展的基本步骤

编写一个PHP扩展主要分为以下几步：
```
	* 通过ext目录下ext_skel脚本生成扩展的基本框架：./ext_skel --extname；
	* 修改config.m4配置：设置编译配置参数、设置扩展的源文件、依赖库/函数检查等等；
	* 编写扩展要实现的功能：按照PHP扩展的格式以及PHP提供的API编写功能；
	* 生成configure：扩展编写完成后执行phpize脚本生成configure及其它配置文件；
	* 编译&安装：./configure、make、make install，然后将扩展的.so路径添加到php.ini中。
```

