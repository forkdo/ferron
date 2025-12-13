---
title: 通过 Docker 安装
---

## 先决条件

在继续安装之前，请确保您具备以下条件：

- 安装了 Docker 的系统。如果未安装 Docker，请按照官方 [Docker 安装指南](https://docs.docker.com/get-started/get-docker/) 进行操作。
- 用于拉取 Ferron Docker 镜像的互联网连接。

## 安装步骤

### 1. 拉取 Ferron Docker 镜像

要从 Docker Hub 下载最新的 Ferron 镜像，请运行以下命令：

```sh
docker pull ferronserver/ferron:2
```

### 2. 运行 Ferron 容器

镜像下载完成后，使用以下命令启动 Ferron 容器：

```sh
docker run --name myferron -d -p 80:80 --restart=always ferronserver/ferron:2
```

此命令执行以下操作：

- `--name myferron`：为正在运行的容器分配一个名称 (`myferron`)。
- `-d`：以分离模式运行容器。
- `-p 80:80`：将容器的 80 端口映射到主机的 80 端口。
- `--restart=always`：确保容器在停止或系统重新启动时自动重新启动。

## 验证安装

要确认 Ferron 正在运行，请执行：

```sh
docker ps
```

这应该会显示一个名为 `myferron` 的正在运行的容器。

要测试 Web 服务器，请打开浏览器并导航到 `http://localhost`。您应该会看到默认的 Ferron 欢迎页面。

或者，使用 `curl`：

```sh
curl http://localhost
```

## 文件结构

Docker 上的 Ferron 具有以下文件结构：

- _/usr/sbin/ferron_ - Ferron Web 服务器
- _/usr/sbin/ferron-passwd_ - Ferron 用户密码生成工具
- _/usr/sbin/ferron-yaml2kdl_ - Ferron 配置转换工具
- _/var/cache/ferron-acme_ - Ferron 的 ACME 缓存目录（如果未在服务器配置中明确指定）
- _/var/log/ferron/access.log_ - 组合日志格式的 Ferron 访问日志（默认配置）
- _/var/log/ferron/error.log_ - Ferron 错误日志（默认配置）
- _/var/www/ferron_ - Ferron 的 Web 根目录
- _/etc/ferron.kdl_ - Ferron 配置

## 管理 Ferron 容器

### 停止容器

要停止 Ferron 容器，请运行：

```sh
docker stop myferron
```

### 重启容器

要重启容器：

```sh
docker start myferron
```

### 删除容器

如果您需要删除 Ferron 容器：

```sh
docker rm -f myferron
```

## 将 Ferron 与 Docker Compose 结合使用

如果您正在使用 Docker Compose，可以在 `docker-compose.yml` 文件中为 Ferron 定义一个服务：

```yaml
services:
  ferron:
    image: ferronserver/ferron:2
    ports:
      - "80:80"
    restart: always
```

然后，您可以使用以下命令启动 Ferron：

```sh
docker-compose up -d
```

### 示例：将 Ferron 与 Docker Compose 和自动 TLS 结合使用

如果将 Ferron 与 Docker Compose 和自动 TLS 结合使用，您可以使用以下 `docker-compose.yml` 文件内容：

```yaml
services:
  # Ferron 容器
  ferron:
    image: ferronserver/ferron:2
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - "./ferron.kdl:/etc/ferron.kdl" # Ferron 配置文件
      - "ferron-acme:/var/cache/ferron-acme" # 此卷需要用于持久性自动 TLS 缓存，否则 Web 服务器将在每次重新启动时获取新证书
    restart: always

volumes:
  ferron-acme:
```

您可能还需要像这样在“ferron.kdl”文件中配置 Ferron：

```kdl
// 将“example.com”替换为您的网站域名
example.com {
    root "/var/www/ferron"
}
```

然后，您可以使用以下命令启动 Ferron：

```sh
docker-compose up -d
```

## Ferron 镜像标签

Ferron 为 Ferron 镜像提供以下标签（适用于 Ferron 2.x）：

- `2` - 基于 Distroless，静态链接的二进制文件
- `2-alpine` - 基于 Alpine Linux，静态链接的二进制文件
- `2-debian` - 基于 Debian GNU/Linux，动态链接的二进制文件（需要 GNU libc）
