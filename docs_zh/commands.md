---
title: 命令
---

Ferron 附带了几个额外的命令行工具。

## 命令用法

### `ferron`

```
一个用 Rust 编写的快速、内存安全的 Web 服务器

用法：ferron [选项]

选项：
  -c, --config <config>
          服务器配置文件的路径 [默认: ./ferron.kdl]
      --config-adapter <config-adapter>
          要使用的配置适配器 [可选值: kdl, yaml-legacy]
      --module-config
          打印已使用的编译时模块配置（Ferron 源代码中的 `ferron-build.yaml` 或 `ferron-build-override.yaml`）并退出
  -V, --version
          打印版本和构建信息
  -h, --help
          打印帮助
```

### `ferron-passwd`

```
Ferron 的密码工具

用法：ferron-passwd

选项：
  -h, --help     打印帮助
  -V, --version  打印版本
```

### `ferron-precompress`

```
一个为 Ferron 预压缩静态文件的实用程序

用法：ferron-precompress [选项] <assets>...

参数：
  <assets>...  静态资产的路径（可以是目录或文件）

选项：
  -t, --threads <threads>  用于压缩的线程数 [默认: 64]
  -h, --help               打印帮助
  -V, --version            打印版本
```

### `ferron-yaml2kdl`

```
一个试图将 Ferron 1.x YAML 配置转换为 Ferron 2.x KDL 配置的实用程序

用法：ferron-yaml2kdl <input> <output>

参数：
  <input>   包含 Ferron 1.x YAML 配置的输入文件的名称
  <output>  包含 Ferron 2.x KDL 配置的输出文件的名称

选项：
  -h, --help     打印帮助
  -V, --version  打印版本
```
