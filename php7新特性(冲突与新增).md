# php7新特性 -- 冲突与新增

## 不兼容性
###1.foreach 不再改变内部指针


```
$arr = [‘a’,’b’,’c'];
foreach($arr as &$v){
     echo current($v) . ‘,';
}
```

低于7.0版本会 打印 1，2，false
7.0版本以上会打印  0，0，0

### 2.16进制字符串不再认为是数字


	‘0x123'


### 3.new操作符创建对象，不能以引用方式赋值给变量

	$a = new b();

	new 前不可以有取地址符，会直接报解析错误。比如
	
	$a = &new b();

### 4.移除了asp 和 script 标签

	asp： <%  %>
	script：<script language=“php”> echo  time();</script>

(还有其他两种，最常见的 <?php ?> ，<?= ?>，第二种在Yii中比较常见)

### 5.不再提供系统变量 HTTP_RAW_POST_DATA功能

请使用 php://input 替代

### 6.php.ini 的注释符可以用 ；


## 新特性：
1.标量类型声明（参数类型声明）
php7以前，只支持数组、类的标量类型声明（强制声明），如下：

```
function test(Arrar $arr){}
function test1(ClassName $arr){}
```

php7后可以扩充到 int 、 string、float、bool等类型

### 2.返回值类型声明

可以类似标量类型，扩充到 int 、 string、float、bool等类型，比如
```
function test(string $string):string{}
```

### 3.null合并运算符 ??

由于经常会用到 isset() 方法，新增了 null合并运算符 ??  ；
如果变量存在且不为null，就返回自身的值，否则返回第二个值

### 4.太空船操作符

太空船操作符用于比较两个表达式。当$a小于、等于或大于$b时它分别返回-1、0或1。 比较的原则是沿用 PHP 的常规比较规则进行的。

```
int float string array
```

### 5.define 定义常量数组（7之前的版本不能）


	define(’testArr’,array(1,2,3));

### 6.匿名类

可以用new class 实例化一个匿名类，实现一些阅后即焚的行为，可以一次性创造一个简单类

### 7.unicode codepoint 转译语法 （主要是多语言时候用到）

接受一个16进制形式的unicode codepoint，并打印出一个双引号，或者heredoc包围的UTF-8编码格式的字符串。可以接受任何有效的codepoint，并开头的0是可以省略的

### 8 更加简洁的 闭包 closure::call()

	class A { private $x=1; }


原本：


	$getClass = function() { return $this->x;};
	$getX = $getClass->bindTo(new A , ‘A’);


PHP7

	$getX = function() { return $this->x;};
	echo $getX->call(new A);


### 9 为反序列化进行过滤unserialize() （不常用）

### 10 use 一次性导入多个