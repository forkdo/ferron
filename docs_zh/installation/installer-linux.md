---
title: 通过安装程序安装 (GNU/Linux)
---

## 安装步骤

### 1. 运行安装程序

要安装 Ferron Web 服务器，请运行以下命令：

```bash
sudo bash -c "$(curl -fsSL https://get.ferron.sh/v2)"
```

系统将提示您选择安装类型，并可能在需要时安装包含 `unzip` 和 `setcap` 的软件包。

### 2. 访问 Web 服务器

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

## 更新 Ferron

您可以使用 `ferron-updater` 命令将 Ferron 更新到最新版本。

## 从 Ferron 1.x 升级到 Ferron 2.x

要将 Ferron 从 1.x 升级到 2.x，请运行以下命令：

```bash
sudo bash -c "$(curl -fsSL https://get.ferron.sh/v1-to-v2)"
```

## 管理 Ferron 服务

### 停止服务

要停止 Ferron 服务，请运行：

```sh
sudo /etc/init.d/ferron stop # 对于非 systemd 系统
sudo systemctl stop ferron # 对于 systemd 系统
```

### 重启服务

要重启服务：

```sh
sudo /etc/init.d/ferron restart # 对于非 systemd 系统
sudo systemctl restart ferron # 对于 systemd 系统
```

### 重新加载配置

要在不重启服务的情况下重新加载配置：

```sh
sudo /etc/init.d/ferron reload # 对于非 systemd 系统
sudo systemctl reload ferron # 对于 systemd 系统
```
