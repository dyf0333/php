# 如何解决PHP里大量数据循环时内存耗尽的问题

之前在研究生期间做的二维码相关项目，就遇到过相关问题，今天看技术帖子，又看到类似的问题及解决方法，特此记录；

当时状况与问题是：

1.一物一码的二维码数据在本地生成（C++写的AES-CBS加密的数据，并生成成文件存储在本地），然后用php程序读取并存储在数据库；数据量在1000W-10000W之间；

2.需要读取二维码数据出来展示，可能会很多，数据量在1W-10W，因此就经常会遇到php内存耗尽的错误提示，后来改为分批、分组、分页等解决办法解决（具体不是本文讨论的，后续另寻机会贴出来）。

错误提示：

	PHP Fatal error: Allowed memory size of 268 435 456 bytes exhausted 


以下是帖子解决办法；

PHP查询模式是非缓冲查询，数据库服务器会一条一条的返回数据，而不是一次全部返回，这样的结果就是PHP程序消耗较少的内存，但却增加了数据库服务器的压力，因为数据库会一直等待PHP来取数据，一直到数据全部取完。

错误信息显示允许的最大内存已经耗尽。因为用一个foreach循环语句在一个有N万条记录的表里全表搜索具有特定特征的数据，


这个问题在PHP的官方网站上叫
       
       `缓冲查询` Buffered
       
       和
       
       `非缓冲查询` Unbuffered queries

PHP的查询缺省模式是缓冲模式。

查询数据结果会一次全部提取到内存里供PHP程序处理，这样给了PHP程序额外的功能，

比如说，计算行数，将 指针指向某一行等。

更重要的是程序可以对数据集反复进行二次查询和过滤等操作。但这种缓冲查询模式的缺陷就是消耗内存，也就是用空间换速度。
另外一种PHP查询模式是非缓冲查询，数据库服务器会一条一条的返回数据，而不是一次全部返回，这样的结果就是PHP程序消耗较少的内存，但却增加了数据库服务器的压力，因为数据库会一直等待PHP来取数据，一直到数据全部取完。


很显然，缓冲查询模式适用于小数据量查询，而非缓冲查询适应于大数据量查询。

对于PHP的缓冲模式查询大家都知道，下面列举的例子是如何执行非缓冲查询API。


### 非缓冲查询方法一: mysqli


	1. <?php 
	2. $mysqli  = new mysqli("localhost", "my_user", "my_password", "world"); 
	3. $uresult = $mysqli->query("SELECT Name FROM City", MYSQLI_USE_RESULT); 
	4.  
	5. if ($uresult) { 
	6.    while ($row = $uresult->fetch_assoc()) { 
	7.        echo $row['Name'] . PHP_EOL; 
	8.    } 
	9. } 
	10. $uresult->close(); 
	11. ?> 

### 非缓冲查询方法二: pdo_mysql


	1. <?php 
	2. $pdo = new PDO("mysql:host=localhost;dbname=world", 'my_user', 'my_pass'); 
	3. $pdo->setAttribute(PDO::MYSQL_ATTR_USE_BUFFERED_QUERY, false); 
	4.  
	5. $uresult = $pdo->query("SELECT Name FROM City"); 
	6. if ($uresult) { 
	7.    while ($row = $uresult->fetch(PDO::FETCH_ASSOC)) { 
	8.        echo $row['Name'] . PHP_EOL; 
	9.    } 
	10. } 
	11. ?> 

### 非缓冲查询方法三: mysql


	1. <?php 
	2. $conn = mysql_connect("localhost", "my_user", "my_pass"); 
	3. $db   = mysql_select_db("world"); 
	4.  
	5. $uresult = mysql_unbuffered_query("SELECT Name FROM City"); 
	6. if ($uresult) { 
	7.    while ($row = mysql_fetch_assoc($uresult)) { 
	8.        echo $row['Name'] . PHP_EOL; 
	9.    } 
	10. } 
	11. ?> 

