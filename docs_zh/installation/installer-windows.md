---
title: 通过安装程序安装 (Windows Server)
description: "使用安装脚本在 Windows Server 上安装 Ferron。包含命令、默认路径，以及如何启动/停止服务。"
---

Ferron 可以使用安装脚本安装在 Windows Server 上。本指南将引导您完成安装过程。

## 安装步骤

### 1. 运行安装程序

要安装 Ferron Web 服务器，请运行以下命令：

```batch
powershell -c "irm https://get.ferron.sh/v2-win | iex"
```

系统将提示您选择安装类型。

### 2. 访问 Web 服务器

默认情况下，Ferron 从 `%SystemDrive%\ferron\wwwroot` 目录提供内容。打开 Web 浏览器并导航到 `http://localhost` 以检查服务器是否正在运行并提供默认的 `index.html` 文件。

如果您在页面上看到 "Ferron is installed successfully!"（Ferron 安装成功！）消息，说明 Web 服务器已成功安装并正在运行。

## 文件结构

通过 Windows Server 安装程序安装的 Ferron 具有以下文件结构：

- _%SystemDrive%\ferron\ferron.exe_ - Ferron Web 服务器
- _%SystemDrive%\ferron\ferron-passwd.exe_ - Ferron 用户密码生成工具
- _%SystemDrive%\ferron\ferron-precompress.exe_ - Ferron 静态文件预压缩工具
- _%SystemDrive%\ferron\ferron-yaml2kdl.exe_ - Ferron 配置转换工具
- _%SystemDrive%\ferron\log\access.log_ - 组合日志格式的 Ferron 访问日志
- _%SystemDrive%\ferron\log\error.log_ - Ferron 错误日志
- _%SystemDrive%\ferron\wwwroot_ - Ferron 的 Web 根目录
- _%SystemDrive%\ferron\ferron.kdl_ - Ferron 配置

## 管理 Ferron 服务

### 停止服务

要停止 Ferron 服务，请运行：

```batch
net stop ferron
```

### 重启服务

要重启服务：

```batch
net stop ferron
net start ferron
```
