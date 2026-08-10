---
title: 反向代理
description: "将 Ferron 配置为反向代理，支持 WebSocket、可选的静态/SPA 托管、多个位置以及预压缩资源。"
---

将 Ferron 配置为反向代理非常简单 - 您只需在 `proxy` 指令中指定后端服务器 URL。要将 Ferron 配置为反向代理，您可以使用以下配置：

```kdl
// 带反向代理的示例配置。将“example.com”替换为您的域名。
example.com {
    proxy "http://localhost:3000/" // 将“http://localhost:3000”替换为后端服务器 URL
}
```

在此配置示例中，WebSocket 协议开箱即用地受到支持——无需额外配置。

## 支持静态文件服务的反向代理

Ferron 支持同时提供静态文件和反向代理。您可以为此用例使用此配置：

```kdl
// 带反向代理和静态文件服务的示例配置。将“example.com”替换为您的域名。
example.com {
    // “/api”位置用于反向代理
    // 例如，“/api/login”端点被代理到“http://localhost:3000/api/login”
    location "/api" remove_base=#true {
        proxy "http://localhost:3000/api" // 将“http://localhost:3000/api”替换为后端 API URL
    }

    // “/”位置用于提供静态文件
    location "/" {
        root "/var/www/html" // 将“/var/www/html”替换为包含您的静态文件的目录
    }
}
```

## 带单页应用程序的反向代理

Ferron 支持同时提供单页应用程序和反向代理。您可以为此用例使用此配置：

```kdl
// 带反向代理和静态文件服务的示例配置。将“example.com”替换为您的域名。
example.com {
    // “/api”位置用于反向代理
    // 例如，“/api/login”端点被代理到“http://localhost:3000/api/login”
    location "/api" remove_base=#true {
        proxy "http://localhost:3000/api" // 将“http://localhost:3000/api”替换为后端 API URL
    }

    // “/”位置用于提供静态文件
    location "/" {
        root "/var/www/html" // 将“/var/www/html”替换为包含您的静态文件的目录
        rewrite "^/.*" "/" directory=#false file=#false last=#true
    }
}
```

## 负载均衡

Ferron 通过在 `proxy` 指令中指定多个后端服务器来支持负载均衡。要将 Ferron 配置为负载均衡器，您可以使用以下配置：

```kdl
// 带负载均衡的示例配置。将“example.com”替换为您的域名。
example.com {
    proxy "http://localhost:3000/" // 将“http://localhost:3000”替换为后端服务器 URL
    proxy "http://localhost:3001/" // 将“http://localhost:3001”替换为第二个后端服务器 URL
}
```

## 健康检查

Ferron 支持被动健康检查；您可以使用 `lb_health_check` 指令启用它。要将 Ferron 配置为具有被动健康检查的负载均衡器，您可以使用以下配置：

```kdl
// 带负载均衡和被动健康检查的示例配置。将“example.com”替换为您的域名。
example.com {
    proxy "http://localhost:3000/" // 将“http://localhost:3000”替换为后端服务器 URL
    proxy "http://localhost:3001/" // 将“http://localhost:3001”替换为第二个后端服务器 URL
    lb_health_check
}
```

## 缓存反向代理

Ferron 支持内存缓存以加速网站。要为反向代理启用内存缓存，您可以使用此配置：

```kdl
// 带缓存反向代理的示例配置。将“example.com”替换为您的域名。
example.com {
    proxy "http://localhost:3000/" // 将“http://localhost:3000”替换为后端服务器 URL
    cache
    // 可选：如果要缓存没有 Cache-Control 标头的后端服务器的响应，请设置 Cache-Control 标头
    header "Cache-Control" "max-age=3600"
}
```

## 使用完整“Host”标头的 HTTPS 反向代理

Ferron 默认情况下会在将请求发送到后端服务器之前重写“Host”标头（如果后端服务器使用 HTTPS 协议），并将原始“Host”标头值保留在“X-Forwarded-Host”标头中。

但是，有些 Web 应用程序可能无法使用此配置。这可能导致主机标头不匹配错误和其他问题。

在这种情况下，您可以将“Host”标头值设置为原始值：

```kdl
// 使用完整“Host”标头的 HTTPS 反向代理示例配置。将“example.com”替换为您的域名。
example.com {
    proxy "https://localhost:8443/" // 将“https://localhost:8443”替换为后端服务器 URL
    proxy_request_header_replace "Host" "{header:Host}"
}
```

## 反向代理到在 Unix 套接字上侦听的后端

Ferron 支持反向代理到在 Unix 套接字上侦听的后端。要为反向代理到在 Unix 套接字上侦听的后端配置 Ferron，您可以使用此配置：

```kdl
// 反向代理到在 Unix 套接字上侦听的后端的示例配置。将“example.com”替换为您的域名。
example.com {
    proxy "http://example.com" unix="/run/backend/web.sock" // 后端 URL 中的“example.com”可以替换为任意域名
}
```

## 反向代理到 gRPC 后端

Ferron 支持反向代理到接受通过 HTTPS 或具有先验知识的纯文本的 HTTP/2 请求的 gRPC 后端。要为反向代理到 gRPC 后端配置 Ferron，您可以使用此配置：

```kdl
// 反向代理到 gRPC 后端的示例配置。将“grpc.example.com”替换为您的域名。
grpc.example.com {
    proxy "http://localhost:3000/" // 将“http://localhost:3000”替换为后端服务器 URL
    proxy_http2_only // 启用仅 HTTP/2 代理以支持 gRPC 代理
}
```

## 反向代理到动态后端（通过 SRV 记录）

Ferron 支持通过 SRV 记录反向代理到动态后端。要为反向代理到动态后端配置 Ferron，您可以使用此配置：

```kdl
// 反向代理到动态后端的示例配置。将“example.com”替换为您的域名。
example.com {
    proxy_srv "http://_backend._tcp.example.com/" // 将“_backend._tcp.example.com”替换为您的后端服务器的实际 SRV 记录
}
```

## 示例：Ferron 多路复用到多个后端服务器

在此示例中，`example.com` 和 `bar.example.com` 域指向运行 Ferron 的服务器。

以下是此示例的假设：

- `https://example.com` 是“主站点”，而 `https://example.com/agenda` 托管日历服务。
- `https://foo.example.com` 被传递到 `https://saas.foo.net`
- `https://bar.example.com` 是内部后端的前端。

您可以像这样配置 Ferron：

```kdl
* {
    tls "/path/to/certificate.crt" "/path/to/private.key"
}

example.com {
    location "/agenda" {
        // 它将 /agenda/example 代理到 http://calender.example.net:5000/agenda/example
        proxy "http://calender.example.net:5000"
    }

    location "/" {
        // 捕获所有路径
        proxy "http://localhost:3000/"
    }
}

foo.example.com {
    proxy "https://saas.foo.net"
}

bar.example.com {
    proxy "http://backend.example.net:4000"
}
```

对于 `http://calender.example.net:5000/agenda/example`，您可能需要配置日历服务以删除“agenda/”或在 Ferron 中配置 URL 重写。

## 注意事项与故障排查

- 如果遇到 `502 Bad Gateway` 或 `504 Gateway Timeout`，请检查后端 URL/端口，确保后端进程正在运行，并确认 Ferron 到后端的网络/防火墙访问正常。
- 如果只有部分路径失败，请检查 `location` 匹配顺序和 `remove_base` 行为，以确保转发的路径与后端期望的一致。
- 如果您的后端应用程序报告主机不匹配错误或错误的绝对 URL，请使用 `proxy_request_header_replace "Host" "{header:Host}"`（参见完整 Host 标头部分）。
- 如果后端返回意外的 `404 Not Found`，请分别在有/无 `disable_url_sanitizer` 的情况下进行测试，并在禁用 URL 清理前确认后端路径处理方式。
- 对于静态 + API 混合设置，请将 API 路由放在类似 `/api` 的专用前缀中，并使用捕获所有 `/` 位置提供静态文件或 SPA 回退。
- 对于 gRPC 上游，请启用 `proxy_http2_only`；如果没有仅 HTTP/2 代理，许多 gRPC 后端会失败。
- 如果 Ferron 位于终止 HTTPS 的代理之后，且您还使用了自动 TLS，请使用 HTTP-01 质询而非 TLS-ALPN-01。请参阅 [自动 TLS](/docs/use-cases/automatic-tls#note-about-cloudflare-proxies-and-other-https-proxies)。
