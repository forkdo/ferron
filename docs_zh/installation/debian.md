---
title: 通过包管理器安装 (Debian/Ubuntu)
---

Ferron 为 Debian、Ubuntu 及其衍生版提供官方软件包。以下是通过包管理器在 Debian 或 Ubuntu 上安装 Ferron 的说明。

## 安装步骤

### 1. 添加 Ferron 的存储库

要添加 Ferron 的存储库，请运行以下命令（适用于 Debian 和 Ubuntu，如果您使用的是衍生版，请将 `$(lsb_release -cs)` 替换为最接近的 Debian 或 Ubuntu 版本代号）：

```bash
# 安装添加新存储库所需的软件包
sudo apt install curl gnupg2 ca-certificates lsb-release debian-archive-keyring

# 添加公共 PGP 密钥
curl https://deb.ferron.sh/signing.pgp | gpg --dearmor | sudo tee /usr/share/keyrings/ferron-keyring.gpg >/dev/null

# 添加新的 Debian 软件包存储库
echo "deb [signed-by=/usr/share/keyrings/ferron-keyring.gpg] https://deb.ferron.sh $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/ferron.list

# 获取软件包列表
sudo apt update
```

### 2. 安装 Ferron

要安装 Ferron Web 服务器，请运行以下命令：

```bash
sudo apt install ferron
```

### 3. 访问 Web 服务器

默认情况下，Ferron 从 `/var/www/ferron` 目录提供内容。打开 Web 浏览器并导航到 `http://localhost` 以验证服务器是否正在运行并提供默认的 `index.html` 文件。

## 文件结构

通过 GNU/Linux 安装程序安装的 Ferron 具有以下文件结构：

- _/usr/sbin/ferron_ - Ferron Web 服务器
- _/usr/sbin/ferron-passwd_ - Ferron 用户密码生成工具
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
