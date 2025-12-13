---
title: 重定向
---

如果要将整个网站重定向到另一个网站，可以使用此配置：

```kdl
// 将网站重定向到另一个网站的示例配置。将“example.org”替换为您的域名。
example.org {
    status 302 location="https://www.example.com" // 将“www.example.com”替换为您想要的域。此外，如果需要永久重定向，请将 302 替换为 301。
}
```

如果要将整个网站重定向到另一个网站并保留 URL，可以使用此配置：

```kdl
// 将网站重定向到另一个网站的示例配置。将“example.org”替换为您的域名。
example.org {
    status 302 location="https://www.example.com{path}" // 将“www.example.com”替换为您想要的域。此外，如果需要永久重定向，请将 302 替换为 301。
}
```

### 从不带“www.”的 URL 重定向到带“www.”的 URL

如果要将所有不带“www.”的 URL 请求重定向到带“www.”的 URL，可以使用此配置：

```kdl
// 从不带“www.”的 URL 重定向到带“www.”的 URL 的示例配置。将“example.com”替换为您的域名。
example.com {
    status 301 location="https://www.example.com{path}"
}

www.example.com {
    // 对于此示例，让我们提供静态文件
    root "/var/www/example"
}
```
