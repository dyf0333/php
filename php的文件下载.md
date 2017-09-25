# php的文件下载
  
>1.下载网络资源到本地

>2.下载自己服务器文件到本地

>3.下载网络资源到本服务器


## 1.下载网络文件到本地：

### 思路：

1.通过curl获取header信息

      http状态码（文件存在与否）
      文件类型（如果需求是强制下载为某个文件格式，则不用获取文件类型 ）
                并不是所有文件地址都暴露文件相关信息在URL里，比如微信的二维码图片文件
2.设置header的type为强制下载


代码：

```

$fileURL = 'http://www.XXX.com/......a.png'

$fileExtension = '';

//1.先通过curl的GET方式获取http的header头（若需求是强制下载为某个文件格式，则不用获取文件类型）
$ch = curl_init();
curl_setopt($ch, CURLOPT_URL, $fileURL);
curl_setopt($ch, CURLOPT_TIMEOUT, 200);
curl_setopt($ch, CURLOPT_HEADER, FALSE);
curl_setopt($ch, CURLOPT_NOBODY, FALSE);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, TRUE);
curl_setopt($ch, CURLOPT_FOLLOWLOCATION, FALSE);
curl_setopt($ch, CURLOPT_CUSTOMREQUEST, 'GET');
curl_exec($ch);

$httpHeaderInfo = curl_getinfo($ch); //获取http的header头信息

if ($httpHeaderInfo['http_code'] != 200) {
    //抛异常，文件不存在
}

if (!$fileExtension) {
    $fileExtension = substr($httpHeaderInfo['content_type'], strrpos($httpHeaderInfo['content_type'], '/') + 1);
}
```

### 设置当前链接为强制下载

```
header("Content-Type: application/force-download");
header('Content-Disposition: attachment; filename="' . rand(1000, 9999) . '.' . $fileExtension .'"');
readfile($fileURL);
```

## 2.下载服务器文件到本地：

```

$fileURL = '/.../a.png'

$fileExtension = '';

1.获取文件后缀（若需求是强制下载为某个文件格式，则不用获取文件类型）
if (!$fileExtension) {
    $fileExtension =  substr(strrchr($fileURL , '.'), 1);
}

//2.设置当前链接为强制下载
header("Content-Type: application/force-download");
header('Content-Disposition: attachment; filename="' . rand(1000, 9999) . '.' . $fileExtension .'"');
readfile($fileURL);
```

## 3.下载网络文件到服务器：

### 思路：

1.通过curl获取header信息
  
           http状态码（文件存在与否）
           文件类型（如果需求是强制下载为某个文件格式，则不用获取文件类型 ）
2.创建文件，保存curl到的内容到本地文件     



代码
```

$fileURL = "http://www.baidu.com/img/baidu_jgylogo3.gif";

$fileExtension = '';

//1.先通过curl的GET方式获取http的header头（若需求是强制下载为某个文件格式，则不用获取文件类型）
$ch = curl_init();
curl_setopt($ch, CURLOPT_URL, $fileURL);
curl_setopt($ch, CURLOPT_TIMEOUT, 200);
curl_setopt($ch, CURLOPT_HEADER, FALSE);
curl_setopt($ch, CURLOPT_NOBODY, FALSE);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, TRUE);
curl_setopt($ch, CURLOPT_FOLLOWLOCATION, FALSE);
curl_setopt($ch, CURLOPT_CUSTOMREQUEST, 'GET');
$content = curl_exec($ch);

$httpHeaderInfo = curl_getinfo($ch); //获取http的header头信息

if ($httpHeaderInfo['http_code'] != 200) {
    //抛异常，文件不存在
}

if (!$fileExtension) {
    $fileExtension = substr($httpHeaderInfo['content_type'], strrpos($httpHeaderInfo['content_type'], '/') + 1);
}

$filename = rand(1000, 999) . $fileExtension;
$save_dir = "down/";

if (trim($save_dir) == '') {
    $save_dir = './';
}
if (0 !== strrpos($save_dir, '/')) {
    $save_dir .= '/';
}
//创建保存目录
if (!file_exists($save_dir) && !mkdir($save_dir, 0777, true)) {
    return false;
}

$size = strlen($content);
//文件大小
$fp2 = @fopen($save_dir . $filename, 'a');
fwrite($fp2, $content);
fclose($fp2);
unset($content, $url);
return array(
    'file_name' => $filename,
    'save_path' => $save_dir . $filename,
    'file_size' => $size
);
```

## 补充：

因为有可能文件名是中文,需要下面函数吧文件名转化一下编码 iconv

echo iconv('GB2312', 'UTF-8', $str); //将字符串的编码从GB2312转到UTF-8 


