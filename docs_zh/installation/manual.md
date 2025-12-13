---
title: 手动安装
---

## 安装步骤

### 1. 下载 Ferron ZIP 压缩包

从 [Ferron 下载页面](/download) 获取适合您操作系统的最新 Ferron ZIP 压缩包。

### 2. 解压 ZIP 压缩包

- **类 Unix 系统**:

  打开终端并导航到包含下载的 ZIP 文件的目录。使用以下命令解压内容：

  ```bash
  unzip ferron.zip
  ```

- **Windows**:

  右键单击 ZIP 文件并选择“全部解压...”以解压内容。

### 3. 查看解压后的内容

解压后，您应该会看到以下文件和目录：

- `ferron` 或 `ferron.exe` - 主要的 Ferron Web 服务器可执行文件。
- `ferron-passwd` 或 `ferron-passwd.exe` - 用于为服务器配置生成哈希密码的工具。
- `ferron-yaml2kdl` 或 `ferron-yaml2kdl.exe` - 用于将 Ferron 1.x YAML 配置转换为 Ferron 2.x KDL 配置的工具。
- `ferron.kdl` - Ferron 的示例配置文件。
- `wwwroot/` - 包含默认 `index.html` 文件的 Web 根目录。

### 4. 配置 Ferron

修改 `ferron.yaml` 配置文件以满足您的服务器要求。此文件包括服务器端口、日志记录、模块等设置。详细的配置选项可在[服务器配置属性页面](/docs/configuration)中找到。

### 5. 运行 Ferron

- **类 Unix 系统**:

  在终端中，导航到包含 `ferron` 可执行文件的目录并运行：

  ```bash
  ./ferron
  ```

- **Windows**:

  打开命令提示符，导航到包含 `ferron.exe` 的目录，然后执行：

  ```cmd
  ferron.exe
  ```

### 6. 访问 Web 服务器

默认情况下，Ferron 从 `wwwroot` 目录提供内容。打开 Web 浏览器并导航到 `http://localhost` 以验证服务器是否正在运行并提供默认的 `index.html` 文件。

## 重新加载配置 (类 Unix 系统)

要在不重启服务的情况下重新加载配置，请向 `ferron` 进程发送 `SIGHUP` 信号：

```bash
kill -HUP $(pidof ferron)
```

## 附加工具

- **Ferron 密码工具**:

  `ferron-passwd` 工具有助于为安全配置生成带有哈希密码的用户条目。要使用它：
  - **类 Unix 系统**:
    ```bash
    ./ferron-passwd someuser
    ```
  - **Windows**:
    ```cmd
    ferron-passwd.exe someuser
    ```

  按照屏幕上的提示为您的配置文件生成必要的条目。
