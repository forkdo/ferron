---
title: 可观察性后端
---

Ferron 2.2.0 及更高版本支持模块化可观察性后端。这使您可以监控您的 Ferron 服务器并深入了解其性能和行为。

以下可观察性后端支持内置于 Ferron 中并默认启用：

- _logfile_ - 此可观察性后端将请求和错误记录到文件中。
- _otlp_ (Ferron 2.2.0 或更高版本) - 此可观察性后端将请求和错误发送到支持 OTLP 的服务（例如 OpenTelemetry 收集器）。

Ferron 还支持可在编译时启用的其他可观察性后端。

Ferron 提供的其他可观察性后端支持来自以下存储库：

- [ferron-observability-example](https://github.com/ferronweb/ferron-observability-example.git) - 将请求和错误记录到控制台。

如果您想将 Ferron 与其他可观察性后端一起使用，可以查看[编译说明](https://github.com/ferronweb/ferron/blob/2.x/COMPILATION.md)。

## 指标说明

Ferron 中的指标使用 OpenTelemetry 风格的名称指定。以下是 Ferron 发送的指标：

- **`http.server.active_requests`** (单位: `{request}`)
  - 活动的 HTTP 服务器请求数。
  - **属性**
    - `http.request.method` - HTTP 请求方法。
    - `url.scheme` - URL 方案 ( `"http"` 或 `"https"` )。
    - `network.protocol.name` - 始终为 `"http"`。
    - `network.protocol.version` - HTTP 版本。
    - `ferron.http.request.error_status_code` - 在执行具有错误配置的请求处理程序之前发生的 HTTP 错误状态代码。
- **`http.server.request.duration`** (单位: `s`)
  - HTTP 服务器请求的持续时间。此指标还包括发生较早的 HTTP 响应错误的请求。
  - **属性**
    - `http.request.method` - HTTP 请求方法。
    - `url.scheme` - URL 方案 ( `"http"` 或 `"https"` )。
    - `network.protocol.name` - 始终为 `"http"`。
    - `network.protocol.version` - HTTP 版本。
    - `ferron.http.request.error_status_code` - 在执行具有错误配置的请求处理程序之前发生的 HTTP 错误状态代码。
- **`ferron.http.server.request_count`** (单位: `{request}`)
  - HTTP 服务器请求数。此指标还包括发生较早的 HTTP 响应错误的请求。
  - **属性**
    - `http.request.method` - HTTP 请求方法。
    - `url.scheme` - URL 方案 ( `"http"` 或 `"https"` )。
    - `network.protocol.name` - 始终为 `"http"`。
    - `network.protocol.version` - HTTP 版本。
    - `http.response.status_code` - HTTP 响应状态代码。
    - `error.type` - 错误类型（如果状态代码指示客户端或服务器错误）。
    - `ferron.http.request.error_status_code` - 在执行具有错误配置的请求处理程序之前发生的 HTTP 错误状态代码。
- **`ferron.proxy.backends.selected`** (单位: `{backend}`; _rproxy_ 模块)
  - 选择后端服务器的次数。
  - **属性**
    - `ferron.proxy.backend_url` - 后端服务器 URL。
    - `ferron.proxy.backend_unix_path` - 后端服务器 Unix 套接字路径。
- **`ferron.proxy.backends.unhealthy`** (单位: `{backend}`; _rproxy_ 模块)
  - 后端服务器的健康检查失败次数。
  - **属性**
    - `ferron.proxy.backend_url` - 后端服务器 URL。
    - `ferron.proxy.backend_unix_path` - 后端服务器 Unix 套接字路径。
- **`ferron.cache.lookups`** (单位: `{lookup}`; _cache_ 模块)
  - 执行缓存查找的次数。
  - **属性**
    - `ferron.cache.result` - 缓存查找结果 ( `"hit"` 或 `"miss"` )。
- **`ferron.cache.items`** (单位: `{item}`; _cache_ 模块)
  - 缓存中的项目数。
- **`ferron.cache.evictions`** (单位: `{eviction}`; _cache_ 模块)
  - 缓存逐出（删除项目）的次数。
  - **属性**
    - `ferron.cache.eviction_reason` - 缓存逐出原因 ( `"size"` 或 `"expired"` )。
- **`process.cpu.time`** (单位: `s`; Linux)
  - 按不同状态细分的总 CPU 秒数。
  - **属性**
    - `cpu.mode` - CPU 的模式 ( `"user"` 或 `"system"` )
- **`process.cpu.utilization`** (单位: `1`; Linux)
  - 自上次测量以来 process.cpu.time 的差异，除以经过的时间和可用于进程的 CPU 数量。
  - **属性**
    - `cpu.mode` - CPU 的模式 ( `"user"` 或 `"system"` )
- **`process.memory.usage`** (单位: `By`; Linux)
  - 正在使用的物理内存量。
- **`process.memory.virtual`** (单位: `By`; Linux)
  - 已提交的虚拟内存量。

## 可观察性后端说明

### _otlp_ 可观察性后端

此可观察性后端支持 OTLP (OpenTelemetry Protocol) 日志、指标和跟踪。

对于 OTLP 日志，访问日志具有 `access` OTLP 范围，而错误日志具有 `error` 范围。

对于 OTLP 指标，它们具有 `ferron` 范围。

对于 OTLP 跟踪，它们具有 `ferron` 范围。
