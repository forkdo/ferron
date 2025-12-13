---
title: 静态文件服务
---

将 Ferron 配置为静态文件服务器非常简单——您只需在 `root` 指令中指定包含静态文件的目录即可。要将 Ferron 配置为静态文件服务器，您可以使用以下配置：

```kdl
// 使用静态文件服务的示例配置。将“example.com”替换为您的域名。
example.com {
    root "/var/www/html" // 将“/var/www/html”替换为包含您的静态文件的目录
}
```

## 静态文件的 HTTP 压缩

静态文件的 HTTP 压缩默认启用。要禁用它，您可以使用此配置：

```kdl
// 禁用静态文件服务和 HTTP 压缩的示例配置。将“example.com”替换为您的域名。
example.com {
    root "/var/www/html" // 将“/var/www/html”替换为包含您的静态文件的目录
    compressed #false
}
```

## 目录列表

目录列表默认禁用。要启用它们，您可以使用此配置：

```kdl
// 启用静态文件服务和目录列表的示例配置。将“example.com”替换为您的域名。
example.com {
    root "/var/www/html" // 将“/var/www/html”替换为包含您的静态文件的目录
    directory_listing
}
```

## 单页应用程序

通过在静态文件服务配置之外添加 URL 重写规则（如果仅使用静态文件服务），Ferron 也支持单页应用程序 (SPA)。您可以使用此配置：

```kdl
// 静态文件服务和 SPA 的 URL 重写规则的示例配置。将“example.com”替换为您的域名。
example.com {
    root "/var/www/html" // 将“/var/www/html”替换为包含您的静态文件的目录
    rewrite "^/.*" "/" directory=#false file=#false last=#true
}
```

## 带内存缓存的静态文件服务

Ferron 支持内存缓存以加速网站。要为静态文件启用内存缓存，您可以使用此配置：

```kdl
// 启用静态文件服务和内存缓存的示例配置。将“example.com”替换为您的域名。
example.com {
    root "/var/www/html" // 将“/var/www/html”替换为包含您的静态文件的目录
    cache
    file_cache_control "max-age=3600"
}
```

## 提供预压缩的静态文件

Ferron 支持提供预压缩的静态文件。要启用此功能，您可以使用此配置：

```kdl
// 启用静态文件服务和预压缩文件的示例配置。将“example.com”替换为您的域名。
example.com {
    root "/var/www/html" // 将“/var/www/html”替换为包含您的静态文件的目录
    precompressed
}
```

在此配置中，如果存在静态文件的预压缩版本，Ferron 将提供它们。预压缩的静态文件将另外具有 `.gz`（gzip）、`.deflate`（Deflate）、`.br`（Brotli）或 `.zst`（Zstandard）扩展名。

要创建预压缩的静态文件，您可以使用 Ferron 附带的 `ferron-precompress` 工具：

```bash
# 将“/var/www/html”替换为包含您的静态文件的目录
ferron-precompress /var/www/html
```
