---
title: 静态文件服务
description: "使用 root、压缩、目录列表、SPA 重写、缓存和预压缩资源在 Ferron 中托管静态站点。"
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

## 注意事项与故障排查

- 如果对于本应存在的文件出现 `404 Not Found`，请确认 `root` 路径正确且运行 Ferron 的用户对其具有读取权限。
- 如果 SPA 路由（例如 `/dashboard/settings`）返回 `404 Not Found`，请添加 SPA 部分的重写规则，使未知路径回退到 `/`。
- 如果未提供预压缩资源，请检查匹配的文件是否存在（例如 `app.js.br` 或 `app.js.gz`），并在更改源资源后重新生成它们。
- 如果在使用 `cache` 时响应看起来过时，请缩短缓存生命周期（`file_cache_control`）或在调试时暂时禁用缓存。
- 如果您的站点同时提供静态文件和 API 流量，请使用 `location` 块拆分路由（例如 `/api` 用于代理，`/` 用于静态文件）。请参阅 [反向代理](/docs/use-cases/reverse-proxy)。
- 如果您在终止 HTTPS 的代理（例如 Cloudflare）之后为静态托管启用自动 TLS，请使用 HTTP-01 ACME 质询。请参阅 [自动 TLS](/docs/use-cases/automatic-tls#note-about-cloudflare-proxies-and-other-https-proxies)。
