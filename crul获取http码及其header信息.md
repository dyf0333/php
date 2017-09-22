# crul获取http码及其header信息

> 要用curl方法获取http的header相关数据，需要在 curl_exec() 方法后执行 curl_getinfo();方法

例子：

```

$fileURL = 'https://www.baidu.com'

$ch = curl_init ();
curl_setopt($ch, CURLOPT_URL, $fileURL);
curl_setopt($ch, CURLOPT_TIMEOUT, 200);
curl_setopt($ch, CURLOPT_HEADER, FALSE);
curl_setopt($ch, CURLOPT_NOBODY, FALSE);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, TRUE);
curl_setopt($ch, CURLOPT_FOLLOWLOCATION, FALSE);
curl_setopt($ch, CURLOPT_CUSTOMREQUEST, 'GET');
curl_exec($ch);
$httpCode = curl_getinfo($ch); //获取http的header头信息
//$httpCode = curl_getinfo($ch ,CURLINFO_HTTP_CODE);  //直接获取http响应码

dump($httpCode);

```


返回值：

```

array (size=26)
  'url' => string 'XXX' (length=147)
  'content_type' => string 'image/jpg' (length=9)
  'http_code' => int 200
  'header_size' => int 226
  'request_size' => int 177
  'filetime' => int -1
  'ssl_verify_result' => int 0
  'redirect_count' => int 0
  'total_time' => float 1.484
  'namelookup_time' => float 0.015
  'connect_time' => float 0.078
  'pretransfer_time' => float 0.172
  'size_upload' => float 0
  'size_download' => float 37889
  'speed_download' => float 25531
  'speed_upload' => float 0
  'download_content_length' => float 37889
  'upload_content_length' => float -1
  'starttransfer_time' => float 0.672
  'redirect_time' => float 0
  'redirect_url' => string '' (length=0)
  'primary_ip' => string '' (length=14)
  'certinfo' => 
    array (size=0)
      empty
  'primary_port' => int 443
  'local_ip' => string '' (length=13)
  'local_port' => int 60740

```




### curl_getinfo

>ch 
  
由 curl_init() 返回的 cURL 句柄。

>opt

这个参数可能是以下常量之一:

```
CURLINFO_EFFECTIVE_URL - 最后一个有效的URL地址
CURLINFO_HTTP_CODE - 最后一个收到的HTTP代码
CURLINFO_FILETIME - 远程获取文档的时间，如果无法获取，则返回值为“-1”
CURLINFO_TOTAL_TIME - 最后一次传输所消耗的时间
CURLINFO_NAMELOOKUP_TIME - 名称解析所消耗的时间
CURLINFO_CONNECT_TIME - 建立连接所消耗的时间
CURLINFO_PRETRANSFER_TIME - 从建立连接到准备传输所使用的时间
CURLINFO_STARTTRANSFER_TIME - 从建立连接到传输开始所使用的时间
CURLINFO_REDIRECT_TIME - 在事务传输开始前重定向所使用的时间
CURLINFO_SIZE_UPLOAD - 上传数据量的总值
CURLINFO_SIZE_DOWNLOAD - 下载数据量的总值
CURLINFO_SPEED_DOWNLOAD - 平均下载速度
CURLINFO_SPEED_UPLOAD - 平均上传速度
CURLINFO_HEADER_SIZE - header部分的大小
CURLINFO_HEADER_OUT - 发送请求的字符串
CURLINFO_REQUEST_SIZE - 在HTTP请求中有问题的请求的大小
CURLINFO_SSL_VERIFYRESULT - 通过设置CURLOPT_SSL_VERIFYPEER返回的SSL证书验证请求的结果
CURLINFO_CONTENT_LENGTH_DOWNLOAD - 从Content-Length: field中读取的下载内容长度
CURLINFO_CONTENT_LENGTH_UPLOAD - 上传内容大小的说明
CURLINFO_CONTENT_TYPE - 下载内容的Content-Type:值，NULL表示服务器没有发送有效的Content-Type: header
```
