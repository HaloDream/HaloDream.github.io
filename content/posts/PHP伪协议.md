+++
date = '2026-04-21T15:29:15Z'
draft = false
title = 'PHP伪协议'
categories = ["基础知识"]
tags = ["PHP", "伪协议", "文件包含"]
+++

|伪协议|语法格式|核心功能（文件包含场景）|依赖 / 条件|
|--|--|--|--|
|file://     |	file://绝对路径 file://相对路径	                         | 读取本地文件（最基础、最常用）           |  无依赖，默认开启      |
|php://filter|	php://filter/read=convert.base64-encode/resource=文件名	| 读取文件源码（绕过 PHP 执行，直接看代码） |  无依赖，默认开启      |
|php://input |	php://input	                                            | POST 数据当作 PHP 代码执行（RCE 核心）  |  allow_url_include=On |
|data://     |	data://text/plain;base64,编码内容	                    | 直接执行 PHP 代码（无文件上传也能 RCE）   |  allow_url_include=On | 
|phar://	 |  phar://压缩包路径/内部文件                               | 读取压缩包内文件 / 代码执行（配合上传）   |	PHP >=5.3.0          |
|zip://	     |  zip://压缩包路径#内部文件                                | 读取 zip 压缩包内文件                   |  无特殊依赖            |
|expect://   |  expect://系统命令                                       | 直接执行系统命令（高危，极少开启）        |  需安装扩展            |