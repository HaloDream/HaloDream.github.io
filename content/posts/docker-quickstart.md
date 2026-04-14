---
title: "Docker 快速入门指南"
date: 2026-04-05T09:00:00+08:00
draft: false
description: "从零开始学习 Docker，掌握容器化技术的核心概念和常用命令"
tags: ["Docker", "容器", "DevOps"]
categories: ["技术"]
showToc: true
tocOpen: true
---

## 什么是 Docker？

Docker 是一个开源的容器化平台，可以将应用程序及其依赖项打包到一个轻量级、可移植的容器中。

**核心概念：**
- **镜像（Image）**：容器的模板，只读
- **容器（Container）**：镜像的运行实例
- **仓库（Registry）**：存储和分发镜像的地方

## 安装 Docker

### Ubuntu/Debian

```bash
# 更新包索引
sudo apt-get update

# 安装必要依赖
sudo apt-get install -y apt-transport-https ca-certificates curl gnupg

# 添加 Docker GPG 密钥
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# 安装 Docker Engine
sudo apt-get install -y docker-ce docker-ce-cli containerd.io

# 验证安装
docker --version
```

## 常用命令

### 镜像操作

```bash
# 拉取镜像
docker pull nginx:latest

# 查看本地镜像
docker images

# 删除镜像
docker rmi nginx:latest

# 构建镜像
docker build -t myapp:v1 .
```

### 容器操作

```bash
# 运行容器
docker run -d -p 80:80 --name mynginx nginx

# 查看运行中的容器
docker ps

# 查看所有容器
docker ps -a

# 停止容器
docker stop mynginx

# 删除容器
docker rm mynginx

# 进入容器
docker exec -it mynginx /bin/bash

# 查看容器日志
docker logs -f mynginx
```

## 编写 Dockerfile

```dockerfile
# 使用官方 Go 镜像作为构建环境
FROM golang:1.21-alpine AS builder

WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN go build -o main .

# 使用最小化镜像运行
FROM alpine:latest
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=builder /app/main .
EXPOSE 8080
CMD ["./main"]
```

## Docker Compose

使用 `docker-compose.yml` 管理多容器应用：

```yaml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "8080:8080"
    depends_on:
      - db
    environment:
      - DB_HOST=db
      - DB_PORT=5432

  db:
    image: postgres:15
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

```bash
# 启动所有服务
docker-compose up -d

# 查看服务状态
docker-compose ps

# 停止所有服务
docker-compose down
```

## 总结

Docker 大大简化了应用程序的部署流程：

| 传统部署 | Docker 部署 |
|---------|------------|
| 依赖环境配置复杂 | 一次构建，到处运行 |
| 环境差异导致问题 | 环境一致性保证 |
| 扩缩容困难 | 快速扩缩容 |
| 资源利用率低 | 资源隔离，利用率高 |

掌握 Docker，是现代开发者必备的技能之一！
