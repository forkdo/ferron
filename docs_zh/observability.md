---
title: 可观测性后端
---

Ferron 2.2.0 及更新版本支持模块化可观测性后端。这允许您监控 Ferron 服务器并深入了解其性能和行为。

以下可观测性后端支持已内置到 Ferron 中并默认启用：

- _logfile_ - 此可观测性后端将请求和错误记录到文件中。
- _otlp_ (Ferron 2.2.0 或更新版本) - 此可观测性后端将请求和错误发送到支持 OTLP 的服务（例如 OpenTelemetry 收集器）。

Ferron 还支持可在编译时启用的额外可观测性后端。

Ferron 提供的额外可观测性后端支持来自以下仓库：

- [ferron-observability-example](https://github.com/ferronweb/ferron-observability-example.git) - 将请求和错误记录到控制台中。

如果您想将 Ferron 与额外的可观测性后端一起使用，可以查看 [编译说明](https://github.com/ferronweb/ferron/blob/2.x/COMPILATION.md)。

## 指标说明

Ferron 中的指标使用 OpenTelemetry 风格的名称指定。以下是 Ferron 发送的指标：

- **`http.server.active_requests`** (单位: `{request}`)
  - 活跃 HTTP 服务器请求数量。
  - **属性**
    - `http.request.method` - HTTP 请求方法。
    - `url.scheme` - URL 方案（`"http"` 或 `"https"`）。
    - `network.protocol.name` - 始终为 `"http"`。
    - `network.protocol.version` - HTTP 版本。
    - `ferron.http.request.error_status_code` - 在具有错误配置的请求处理程序执行之前发生的 HTTP 错误状态码。
- **`http.server.request.duration`** (单位: `s`)
  - HTTP 服务器请求的持续时间。此指标还包括发生 HTTP 响应错误的请求。
  - **属性**
    - `http.request.method` - HTTP 请求方法。
    - `url.scheme` - URL 方案（`"http"` 或 `"https"`）。
    - `network.protocol.name` - 始终为 `"http"`。
    - `network.protocol.version` - HTTP 版本。
    - `ferron.http.request.error_status_code` - 在具有错误配置的请求处理程序执行之前发生的 HTTP 错误状态码。
- **`ferron.http.server.request_count`** (单位: `{request}`)
  - HTTP 服务器请求数量。此指标还包括发生 HTTP 响应错误的请求。
  - **属性**
    - `http.request.method` - HTTP 请求方法。
    - `url.scheme` - URL 方案（`"http"` 或 `"https"`）。
    - `network.protocol.name` - 始终为 `"http"`。
    - `network.protocol.version` - HTTP 版本。
    - `http.response.status_code` - HTTP 响应状态码。
    - `error.type` - 错误类型（如果状态码表示客户端或服务器错误）。
    - `ferron.http.request.error_status_code` - 在具有错误配置的请求处理程序执行之前发生的 HTTP 错误状态码。
- **`ferron.proxy.backends.selected`** (单位: `{backend}`; _rproxy_ 模块)
  - 后端服务器被选中的次数。
  - **属性**
    - `ferron.proxy.backend_url` - 后端服务器 URL。
    - `ferron.proxy.backend_unix_path` - 后端服务器 Unix 套接字路径。
- **`ferron.proxy.backends.unhealthy`** (单位: `{backend}`; _rproxy_ 模块)
  - 后端服务器健康检查失败次数。
  - **属性**
    - `ferron.proxy.backend_url` - 后端服务器 URL。
    - `ferron.proxy.backend_unix_path` - 后端服务器 Unix 套接字路径。
- **`ferron.proxy.requests`** (单位: `{request}`; _rproxy_ 模块; Ferron 2.3.0 或更新版本)
  - 反向代理请求数量。
  - **属性**
    - `ferron.proxy.connection_reused` - HTTP 客户端连接是否被重用。
- **`ferron.cache.lookups`** (单位: `{lookup}`; _cache_ 模块)
  - 缓存查找执行的次数。
  - **属性**
    - `ferron.cache.result` - 缓存查找结果（`"hit"` 或 `"miss"`）。
- **`ferron.cache.items`** (单位: `{item}`; _cache_ 模块)
  - 缓存中的项目数量。
- **`ferron.cache.evictions`** (单位: `{eviction}`; _cache_ 模块)
  - 缓存驱逐（项目移除）次数。
  - **属性**
    - `ferron.cache.eviction_reason` - 缓存驱逐原因（`"size"` 或 `"expired"`）。
- **`process.cpu.time`** (单位: `s`; Linux)
  - 按不同状态分解的总 CPU 秒数。
  - **属性**
    - `cpu.mode` - CPU 模式（`"user"` 或 `"system"`）
- **`process.cpu.utilization`** (单位: `1`; Linux)
  - 自上次测量以来 process.cpu.time 的差异，除以经过的时间和进程可用的 CPU 数量。
  - **属性**
    - `cpu.mode` - CPU 模式（`"user"` 或 `"system"`）
- **`process.memory.usage`** (单位: `By`; Linux)
  - 使用的物理内存量。
- **`process.memory.virtual`** (单位: `By`; Linux)
  - 已提交的虚拟内存量。

## 可观测性后端说明

### _otlp_ 可观测性后端

此可观测性后端支持 OTLP（OpenTelemetry 协议）日志、指标和追踪。

对于 OTLP 日志，访问日志具有 `access` OTLP 作用域，而错误日志具有 `error` 作用域。

对于 OTLP 指标，它们具有 `ferron` 作用域。

对于 OTLP 追踪，它们具有 `ferron` 作用域。