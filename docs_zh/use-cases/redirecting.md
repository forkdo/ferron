---
title: 重定向
description: "在 Ferron 中使用 status + Location 进行流量重定向，包括保留路径的重定向和规范化主机重定向。"
---

Ferron 可以使用带有 `location` 属性的 `status` 来发起重定向。这对于域名迁移、规范化主机名以及临时维护路由非常有用。

## 将整个站点重定向到另一个 URL

使用 `302` 进行临时重定向，或使用 `301` 进行永久重定向：

```kdl
// 将网站重定向到另一个网站的示例配置。将“example.org”替换为您的域名。
example.org {
    status 302 location="https://www.example.com" // 替换为您目标 URL；永久重定向请使用 301。
}
```

## 重定向并保留请求路径

如果您想保留原始请求路径：

```kdl
// 将网站重定向到另一个网站的示例配置。将“example.org”替换为您的域名。
example.org {
    status 302 location="https://www.example.com{path}" // 替换为您目标主机；永久重定向请使用 301。
}
```

例如，`example.org` 上的 `/docs/page` 会重定向到 `www.example.com` 上的 `/docs/page`。

## 规范化主机重定向（非 www -> www）

要将所有来自 `example.com` 的流量重定向到 `www.example.com`：

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

## 规范化主机重定向（www -> 非 www）

如果您偏好以非 www 作为规范化主机：

```kdl
// 从带“www.”的 URL 重定向到不带“www.”的 URL 的示例配置。将“example.com”替换为您的域名。
www.example.com {
    status 301 location="https://example.com{path}"
}

example.com {
    // 对于此示例，让我们提供静态文件
    root "/var/www/example"
}
```

## 注意事项与故障排查

- 如果在修改配置后浏览器仍使用旧的重定向，请清除浏览器缓存或使用隐私窗口测试；`301` 响应会被激进地缓存。
- 测试时请使用 `302`，只有在确认重定向为最终状态后才切换到 `301`。
- 如果发生重定向链，请确认每个请求只应用一条主机/协议规范化规则。
- 有关自动 `www` 重定向，请参阅 [配置：路由与 URL 处理](/docs/configuration/routing-url-processing) 中的 `wwwredirect`。
- 有关 URL 重写（内部路径转换），请参阅 [URL 重写](/docs/use-cases/url-rewriting)。
- 有关 `status` 指令及其属性的参考，请参阅 [配置：安全与 TLS](/docs/configuration/security-tls)。
