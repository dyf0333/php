# php文件操作

>文件操作函数

>目录操作函数

## 1.文件操作函数

### basename()
输出文件名

basename(path,suffix)

suffix
>可选。规定文件扩展名。如果文件有 suffix，则不会输出这个扩展名。


### dirname();

输入路径

### pathinfo();

=> dirname; basename; extension; filename;

### filetype();

文件类型

### fstat();

获取文件信息

例子：
```
$fp = fopen('/a/b/c/d.php');
$fstat = stat($fp);
fclose($fstat);
var_dump($fstat);
```

### filesize();

文件大小

### disk_free_space();

文件目录磁盘空间可用大小

### disk_total_space();

文件目录磁盘空间总大小


### fopen($filepath,$mode);

```
$mode:

"r"     只读方式打开，将文件指针指向文件头。
"r+"    读写方式打开，将文件指针指向文件头。
"w"     写入方式打开，将文件指针指向文件头并将文件大小截为零。如果文件不存在则尝试创建之。
"w+"    读写方式打开，将文件指针指向文件头并将文件大小截为零。如果文件不存在则尝试创建之。
"a"     写入方式打开，将文件指针指向文件末尾。如果文件不存在则尝试创建之。
"a+"    读写方式打开，将文件指针指向文件末尾。如果文件不存在则尝试创建之。
"x"     创建并以写入方式打开，将文件指针指向文件头。如果文件已存在，则 fopen() 调用失败并返回 FALSE，并生成一条 E_WARNING 级别的错误信息。如果文件不存在则尝试创建之。
        这和给底层的 open(2) 系统调用指定 O_EXCL|O_CREAT 标记是等价的。
        此选项被 PHP 4.3.2 以及以后的版本所支持，仅能用于本地文件。
"x+"    创建并以读写方式打开，将文件指针指向文件头。如果文件已存在，则 fopen() 调用失败并返回 FALSE，并生成一条 E_WARNING 级别的错误信息。如果文件不存在则尝试创建之。
        这和给底层的 open(2) 系统调用指定 O_EXCL|O_CREAT 标记是等价的。
        此选项被 PHP 4.3.2 以及以后的版本所支持，仅能用于本地文件。
```



### file();

返回内容，数组形式，文件每行一个对应一个值

### file_get_content();

返回内容

### fget();
 
 取1024字节 大文件，按行读取

例子：
```
$handle = fopen(‘filepath’);
if($handle){
    while(!feof($handle)){
        $buffer = fget($handle,4096);
        echo $buffer;
    }
    fclose($handle);
}
```


### flock( $handle , LOCK_SH | LOCK_EX | LOCK_UN );

文件锁
>可选参数： 共享锁（读用到）| 排他锁（写用到）|解锁

### fwrite( $handle , 'content' ); 

### is_readable()

可读

### is_writebale()

可写


### chown($filepath, $user);

### chmod($filepath,$mode);

### chgrp($filepath, $group);



## 2.目录操作函数

### is_dir()；

是否一个目录

### mkdir($path,$mode);
$mode =0700

### opendir();

打开文件句柄

### readdir();

读取文件句柄

例子：
```
if($handle = opendir('./test')){
    echo 'Files:';
    while ( false != ($file= readdir($handle))) {
            echo $file.PHP_EOL;
    }
}
```

## 遍历目录

```
$dir = './';

if(is_dir($dir)){
    if($dh = opendir($dir)){
        while ($file=readdir($dh) !== false) {
            if($file == '.'|| $file=='..'){
                continue;
            }
            echo "file:{$file}".PHP_EOL;
            echo "filetype" . filetype($dir.$file) . PHP_EOL;
        }
    }
}
```

## 遍历目录与子目录
```
public function scanAll($dir)
{
    if(is_dir($dir)){
        echo 'DIR'.$dir.PHP_EOL;
        $child = scandir($dir);
        foreach ($child as $c) {
            scanAll($dir . '/' .$c);
        }
    }

    if(is_file($dir)){
        echo 'File:' . $dir .PHP_EOL;
    }
}

```


## 复制目录

文件：copy函数（注意权限）

    copy($souce ,$dest)

目录：遍历方法

```
function copyIdr($sourceDir , $destDir)
{
    if(!is_dir($sourceDir)){
        return false;
    }


    if(!is_dir($destDir)){
        if(!mkdir($destDir)){
            return false;
        }
    }

    $dir = opendir($sourceDir);
    if(!$dir){
        return false;
    }

    while (false!==($file = readdir($dir))) {
        if($file != '.' && $file != '..'){
            if(is_dir($sourceDir . '.' . $file)){
                if(!copydir($sourceDir.'/'.$file,$destDir.'/'.$file)){
                    return false;
                }
            }else{
                    if(!copy($sourceDir.'/' . $file ,$destDir.'/'.$file)){
                        return false;
                    }
                }
        }
    }
    closedir($dir);
    return true;
}
```

## 删除目录

文件： unlink函数

目录：
```

function delDir($dir)
{
    $dh = opendir($dir);
    while ($file = readdir($dh)) {
        if($file != '.' && $file != '..' ){
            $fullPath = $dir . '/' .$file;
            if(!is_dir($fullPath)){
                unlink($fullPath);
            }else{
                delDir($fullPath);
            }
        }
    }
    closedir($dh);

    if(rmdir($dir)){
        return true;
    }
    return false;
}
```