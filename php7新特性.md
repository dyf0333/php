# php7新特性

目前开发用到最新版的php7，只知道php7性能比5大幅提升，而具体是那些呢？

特此笔记

```
2015年10月正式发布
性能改进和新的特性，以及改进一些过时功能
源自PHP版本树中的phpng分支
```

## 1.新增的几个运算符

#### 三元运算符

    $a = $_GET['a'] ?? 1;

它相当于：

    $a = isset($_GET['a']) ? $_GET['a'] : 1;

我们知道三元运算符是可以这样用的：
    
    $a ?: 1

但是这是建立在 $a 已经定义了的前提上。新增的 ?? 运算符可以简化判断。



#### <=>  太空舱操作符（组合比较操作符）

用于比较两个表达式。当$a小于、等于或大于$b时它分别返回-1、0或1。 比较的原则是沿用 PHP 的常规比较规则进行的。
```
// Integers
echo 1 <=> 1; // 0
echo 1 <=> 2; // -1
echo 2 <=> 1; // 1

// Floats
echo 1.5 <=> 1.5; // 0
echo 1.5 <=> 2.5; // -1
echo 2.5 <=> 1.5; // 1
 
// Strings
echo "a" <=> "a"; // 0
echo "a" <=> "b"; // -1
echo "b" <=> "a"; // 1
```

#### ** 次方

a的b次方
```
echo 2 ** 3;//8
```

## 2. 函数返回值类型声明(类型严格模式)

官方文档提供的例子（注意 … 的边长参数语法在 PHP 5.6 以上的版本中才有）：
```
    <php
    function arraysSum(array ...$arrays): array
    {
        return array_map(function(array $array): int {
            return array_sum($array);
        }, $arrays);
    }

    print_r(arraysSum([1,2,3], [4,5,6], [7,8,9]));
```

从这个例子中可以看出现在函数（包括匿名函数）都可以指定返回值的类型。


这个特性可以帮助我们避免一些 PHP 的隐式类型转换带来的问题。
在定义一个函数之前就想好预期的结果可以避免一些不必要的错误。

不过这里也有一个特点需要注意。PHP 7 增加了一个 `declare` 指令：strict_types，既使用`严格模式`。


    ·非严格模式，如果返回值不是预期的类型，PHP 还是会对其进行强制类型转换。
    ·严格模式， 则会出发一个 TypeError 的 Fatal error。


#### 强制模式：
```
<php
function foo($a) : int
{
    return $a;
}

foo(1.0);
```
以上代码可以正常执行，foo 函数返回 int 1，没有任何错误。

#### 严格模式：
```
<php
declare(strict_types=1);

function foo($a) : int
{
    return $a;
}

foo(1.0);
# PHP Fatal error:  Uncaught TypeError: Return value of foo() must be of the type integer, float returned in test.php:6
```
在声明之后，就会触发致命错误。

## 3.标量类型声明（Scalar Type Declarations & Scalar Type Declarations）


PHP语言一个非常重要的特点就是“弱类型”，它让PHP的程序变得非常容易编写，新手接触PHP能够快速上手，不过，它也伴随着一些争议。
支持变量类型的定义，可以说是革新性质的变化，PHP开始以可选的方式支持类型定义。
除此之外，还引入了一个开关指令 declare(strict_type=1),当这个指令一旦开启，将会强制当前文件下的程序遵循严格的函数传参类型和返回类型。会抛 fatal 错误

建议添加4种新的标量类型声明：int，float，string和bool
declare(strict_types=1)必须是文件的第一个语句。如果这个语句出现在文件的其他地方，将会产生一个编译错误，块模式是被明确禁止的。
strict_types指令只影响指定使用的文件，不会影响被它包含（通过include等方式）进来的其他文件。该指令在运行时编译，不能修改。它的运作方式，是在opcode中设置一个标志位，让函数调用和返回类型检查符合类型约束。
NULL、arrays和resource不能接受标量类型声明

有一个例外的是，宽泛类型转换是允许int变为float的，也就是说参数如果被声明为float类型，但是它仍然可以接受int参数

```
declare(strict_types = 1);  //开启强制类型
function add(int $a , int $b) :int
{
     return $a+ $b;
}
```

PHP 7 中的函数的形参类型声明可以是标量了。在 PHP 5 中只能是类名、接口、array 或者 callable (PHP 5.4，即可以是函数，包括匿名函数)，现在也可以使用 string、int、float和 bool 了。
官方示例：

	1. <php 
	2. // Coercive mode 
	3. function sumOfInts(int ...$ints) 
	4. { 
	5.     return array_sum($ints); 
	6. } 
	7.  
	8. var_dump(sumOfInts(2, '3', 4.1)); 

需要注意的是上文提到的严格模式的问题在这里同样适用：强制模式（默认，既强制类型转换）下还是会对不符合预期的参数进行强制类型转换，严格模式下则触发 TypeError 的致命错误。


## 4.use 批量声明

PHP 7 中 use 可以在一句话中声明多个类或函数或 const 了：

	<php 
	use some/namespace/{ClassA, ClassB, ClassC as C}; 
	use function some/namespace/{fn_a, fn_b, fn_c}; 
	use const some/namespace/{ConstA, ConstB, ConstC}; 

但还是要写出每个类或函数或 const 的名称
需要留意的问题是：如果你使用的是基于 composer 和 PSR-4 的框架，这种写法是否能成功的加载类文件？其实是可以的，composer 注册的自动加载方法是在类被调用的时候根据类的命名空间去查找位置，这种写法对其没有影响。

## 5.核心错误可以通过异常捕获了
            
```
try {
    non_exists_func();
    } catch(EngineException $e) {
    echo “Exception: {$e->getMessage();}\n”;
}
```



## 6.迭代器

通过添加 yield 关键字支持了 generators，Generators 提供了一个更简单的方法实现迭代器，不需要实现 Iterator 接口 
```
$gen = (function() {
    yield 1;
    yield 2;

    return 3;
})();

foreach ($gen as $val) {
    echo $val, PHP_EOL;
}

echo $gen->getReturn(), PHP_EOL;
 
以上例程会输出：
1
2
3
```

## 7.匿名类 
现在支持通过new class 来实例化一个匿名类，这可以用来替代一些“用后即焚”的完整类定义。

```
interface Logger {
    public function log(string $msg);
}

class Application {
    private $logger;

    public function getLogger(): Logger {
         return $this->logger;
    }

    public function setLogger(Logger $logger) {
         $this->logger = $logger;
    }
}

$app = new Application;
$app->setLogger(new class implements Logger {
    public function log(string $msg) {
        echo $msg;
    }
});

var_dump($app->getLogger());
以上例程会输出：
object(class@anonymous)#2 (0) {
}
```
 
## 8.闭包 Closure::call()

Closure::call() 现在有着更好的性能，简短干练的暂时绑定一个方法到对象上闭包并调用它。
```
class A {private $x = 1;}

// Pre PHP 7 code
$getXCB = function() {return $this->x;};
$getX = $getXCB->bindTo(new A, 'A'); // intermediate closure
echo $getX();

// PHP 7+ code
$getX = function() {return $this->x;};
echo $getX->call(new A);
 
以上例程会输出：
1
1
```

## 9.使用 ... 运算符定义变长参数函数 (PHP 5 >= 5.6.0, PHP 7)

现在可以不依赖 func_get_args()， 使用 ... 运算符 来实现 变长参数函数。
```
<?php
function f($req, $opt = null, ...$params) {
 // $params 是一个包含了剩余参数的数组
 printf('$req: %d; $opt: %d; number of params: %d'."\n",
   $req, $opt, count($params));
}
 
f(1);
f(1, 2);
f(1, 2, 3);
f(1, 2, 3, 4);
f(1, 2, 3, 4, 5);
?>
以上例程会输出：
$req: 1; $opt: 0; number of params: 0
$req: 1; $opt: 2; number of params: 0
$req: 1; $opt: 2; number of params: 1
$req: 1; $opt: 2; number of params: 2
$req: 1; $opt: 2; number of params: 3

```
在调用函数的时候，使用 ... 运算符， 将 数组 和 可遍历 对象展开为函数参数。 在其他编程语言，比如 Ruby中，这被称为连接运算符，。
```
<?php
function add($a, $b, $c) {
 return $a + $b + $c;
}
 
$operators = [2, 3];
echo add(1, ...$operators);
?>
以上例程会输出：
6
```
## 10.AST（Abstract Syntax Tree，抽象语法树）
AST在PHP编译过程作为一个中间件的角色，替换原来直接从解释器吐出opcode的方式，让解释器（parser）和编译器（compliler）解耦，可以减少一些Hack代码，同时，让实现更容易理解和可维护。

![|center](../master/src/php7-1.png)

## 11.PHP数组的变化（HashTable和Zend Array）
在编写PHP程序过程中，使用最频繁的类型莫过于数组，PHP5的数组采用HashTable实现。如果用比较粗略的概括方式来说，它算是一个支持 双向链表的HashTable，不仅支持通过数组的key来做hash映射访问元素，也能通过foreach以访问双向链表的方式遍历数组元素。
PHP5的HashTable：

![|center](../master/src/php7-2.png)

这个图看起来很复杂，各种指针跳来跳去，当我们通过key值访问一个元素内容的时候，有时需要3次的指针跳跃才能找对需要的内容。而最重要的一点， 就在于这些数组元素存储，都是分散在各个不同的内存区域的。同理可得，在CPU读取的时候，因为它们就很可能不在同一级缓存中，会导致CPU不得不到下级 缓存甚至内存区域查找，也就是引起CPU缓存命中下降，进而增加更多的耗时。
PHP7的Zend Array：

![|center](../master/src/php7-3.png)

新版本的数组结构，非常简洁，让人眼前一亮。最大的特点是，整块的数组元素和hash映射表全部连接在一起，被分配在同一块内存内。如果是遍历一个 整型的简单类型数组，效率会非常快，因为，数组元素（Bucket）本身是连续分配在同一块内存里，并且，数组元素的zval会把整型元素存储在内部，也 不再有指针外链，全部数据都存储在当前内存区域内。当然，最重要的是，它能够避免CPU Cache Miss（CPU缓存命中率下降）。

![|center](../master/src/php7-4.png)

ZVAL结构的重构

左边是PHP5的zval(24字节)，右边是PHP7的zval(16字节);

在C语言中struct的每一个成员变量要各自占据一块独立的内存空间,而union里的成员变量是共用一块内存空间（php7中大量使用union替换了struct）。因此,虽然成员变量看起来多了不少,但是实际占据的内存空间有很多都是公用的却下降了。

使用新的Zend Array替换之前的HashTale结构

我们php程序中使用最多、最有用、最方便、最灵活的就是数组了，而php5它的底层就是HashTable实现的，php7使用了新的Zend Array类型，性能和访问速度上都有了大幅度提升

## 12.64位的INT支持

支持存储大于2GB的字符串

支持上传大小大于2GB的文件

保证字符串在所有平台上【64位】都是64bit

## 13.foreach循环的改进
```
//PHP5
$a = array(1, 2, 3);foreach ($a as $v){var_dump(current($a));}
int(2)
int(2)
int(2)
 
$a = array(1, 2, 3);$b=&$a;foreach ($a as $v){var_dump(current($a));}
int(2)
int(3)
bool(false)
 
$a = array(1, 2, 3);$b=$a;foreach ($a as $v){var_dump(current($a));}
int(1)
int(1)
int(1)
 
//PHP7：不再操作数据的内部指针了
$a = array(1, 2, 3);foreach ($a as $v){var_dump(current($a))}
int(1)
int(1)
int(1)
 
$a = array(1, 2, 3);$b=&$a;foreach ($a as $v){var_dump(current($a))
int(1)
int(1)
int(1)
 
$a = array(1, 2, 3);$b=$a;foreach ($a as $v){var_dump(current($a))}
int(1)
int(1)
int(1)
```


## 13 intdiv整数书法函数
PHP7引入了intdiv()的新函数，它执行操作数的整数除法并返回结果为 int 类型。
示例


```
<?php
$value = intdiv(10,3);
var_dump($value);
print("
");
print($value);
?>
```
这将在浏览器产生以下输出 -

int(3) 
3



todo`````````````````````后续补充。。。