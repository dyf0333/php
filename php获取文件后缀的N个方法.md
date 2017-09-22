# php获取文件后缀的N个方法

>补充获取文件后缀的N中方法：

第1种方法：

```
function get_extension($file)
{
substr(strrchr($file, '.'), 1);
}
```
第2种方法：

```
function get_extension($file)
{
return substr($file, strrpos($file, '.')+1);
}
```
第3种方法：

```
function get_extension($file)
{
return end(explode('.', $file));
}
```
第4种方法：
```
function get_extension($file)
{
$info = pathinfo($file);
return $info['extension'];
}
```
第5种方法：
```
function get_extension($file)
{
return pathinfo($file, PATHINFO_EXTENSION);
}
```