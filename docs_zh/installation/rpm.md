---
title: 通过包管理器安装 (RHEL/Fedora)
description: "在 RHEL/Fedora 上使用官方 RPM 软件包安装 Ferron：添加 yum 存储库、安装 Ferron、启用 systemd 服务并验证。"
---

Ferron 为 Red Hat Enterprise Linux (RHEL)、Fedora 及其衍生版提供官方软件包。以下是通过包管理器在 RHEL 或 Fedora 上安装 Ferron 的说明。

## 安装步骤

### 1. 添加 Ferron 的存储库

要添加 Ferron 的存储库，请运行以下命令：

```bash
# 安装添加新存储库所需的软件包
sudo yum install yum-utils

# 添加新的 RPM 软件包存储库
sudo yum-config-manager --add-repo https://rpm.ferron.sh/ferron.repo
```

### 2. 安装 Ferron

要安装 Ferron Web 服务器，请运行以下命令：

```bash
sudo yum install ferron
```

### 3. 启用并启动服务

要启用并启动 Ferron 服务，请运行以下命令：

```bash
sudo systemctl enable ferron
sudo systemctl start ferron
```

### 4. 访问 Web 服务器

默认情况下，Ferron 从 `/var/www/ferron` 目录提供内容。打开 Web 浏览器并导航到 `http://localhost` 以检查服务器是否正在运行并提供默认的 `index.html` 文件。

如果您在页面上看到 "Ferron is installed successfully!"（Ferron 安装成功！）消息，说明 Web 服务器已成功安装并正在运行。

## 文件结构

通过 RHEL/Fedora 软件包安装的 Ferron 具有以下文件结构：

- _/usr/sbin/ferron_ - Ferron Web 服务器
- _/usr/sbin/ferron-passwd_ - Ferron 用户密码生成工具
- _/usr/sbin/ferron-precompress_ - Ferron 静态文件预压缩工具
- _/usr/sbin/ferron-yaml2kdl_ - Ferron 配置转换工具
- _/var/log/ferron/access.log_ - 组合日志格式的 Ferron 访问日志
- _/var/log/ferron/error.log_ - Ferron 错误日志
- _/var/www/ferron_ - Ferron 的 Web 根目录
- _/etc/ferron.kdl_ - Ferron 配置

## 管理 Ferron 服务

### 停止服务

要停止 Ferron 服务，请运行：

```sh
sudo systemctl stop ferron
```

### 重启服务

要重启服务：

```sh
sudo systemctl restart ferron
```

### 重新加载配置

要在不重启服务的情况下重新加载配置：

```sh
sudo systemctl reload ferron
```
