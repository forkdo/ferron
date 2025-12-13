---
title: 服务器配置
---

Ferron 2.0.0 及更高版本可以在 [KDL 格式](https://kdl.dev/) 的配置文件（通常命名为 `ferron.kdl`）中进行配置。以下是此服务器配置属性的说明。

## 配置块

在服务器配置的顶层，指定了代表特定虚拟主机的配置块。以下是此类配置块的示例：

```kdl
globals {
  // 不涉及任何虚拟主机的全局配置
}

* {
  // 全局配置
}

*:80 {
  // 端口 80 的配置
}

example.com {
  // "example.com" 虚拟主机的配置
}

"192.168.1.1" {
  // "192.168.1.1" IP 虚拟主机的配置
}

example.com:8080 {
  // "example.com" 虚拟主机（端口 8080）的配置
}

"192.168.1.1:8080" {
  // "192.168.1.1" IP 虚拟主机（端口 8080）的配置
}

api.example.com {
  // 以下是路径以“/v1/”开头的位置配置。如果存在“remove_base=#true”，则会重写该位置的请求 URL 以删除基本 URL
  location "/v1" remove_base=#false {
    // ...

    // 以下是任何状态代码的错误处理程序配置。
    error_config {
      // ...
    }
  }

  // 位置和条件配置的顺序根据位置和条件的深度自动确定
  location "/" {
    // ...
  }

  // 以下是 404 Not Found 状态代码的错误处理程序配置。如果未包含“404”，则适用于所有错误。
  error_config 404 {
    // ...
  }
}

example.com,example.org {
  // example.com 和 example.org 的配置
  // 虚拟主机标识符（如 example.com 或 "192.168.1.1"）是逗号分隔的，但添加空格将不会被解释，
  // 例如 "example.com, example.org" 对 "example.org" 无效，但 "example.com,example.org" 会起作用。
}

with-conditions.example.com {
  condition "SOME_CONDITION" {
    // 这里定义了条件中的子条件。如果所有子条件也都通过，则该条件将通过
  }

  if "SOME_CONDITION" {
    // 条件配置
    // 条件可以嵌套
  }

  if_not "SOME_CONDITION" {
    // 条件不满足时的配置
  }
}

snippet "EXAMPLE" {
  // 示例代码段配置
  // 包含子条件的代码段也可以在 Ferron 2.1.0 或更高版本中的条件中使用
}

with-snippet.example.com {
  // 从代码段导入
  use "EXAMPLE"
}

with-snippet.example.org {
  // 代码段可重用
  use "EXAMPLE"
}

inheritance.example.com {
  // "proxy" 指令用作演示继承的示例。
  proxy "http://10.0.0.2:3000"
  proxy "http://10.0.0.3:3000"

  // 在这里，这些指令生效：
  //   proxy "http://10.0.0.2:3000"
  //   proxy "http://10.0.0.3:3000"

  location "/somelocation" {
    // 在这里，`some_directive` 指令继承自父块。
    // 这些指令生效：
    //   proxy "http://10.0.0.2:3000"
    //   proxy "http://10.0.0.3:3000"
  }

  location "/anotherlocation" {
    // 如果块中有其他同名指令，则不会继承父块的指令。
    // 在这里，这些指令生效：
    //   proxy "http://10.0.0.4:3000"
    proxy "http://10.0.0.4:3000"
  }
}
```

此外，还可以使用 `include <included_configuration_path: string>` 指令包含其他配置文件，如下所示：

```kdl
include "/etc/ferron.d/**/*.kdl"
```

## 指令类别概述

本配置参考按 **范围**（可在何处使用）和 **功能类别**（它们的作用）组织指令。这使您可以根据您的特定要求更轻松地找到所需的指令。

### 范围

- **仅限全局** - 只能在全局配置范围内使用
- **全局和虚拟主机** - 可在全局和虚拟主机范围内使用
- **通用指令** - 可在包括虚拟主机和位置块在内的各种范围内使用

### 功能类别

- **TLS/SSL 与安全** - 证书管理、加密设置和安全策略
- **HTTP 协议与性能** - 协议设置、超时和性能调整
- **网络与系统** - 网络配置和系统级设置
- **缓存** - HTTP 缓存配置和缓存管理
- **负载均衡** - 健康检查和负载均衡器设置
- **静态文件服务** - 文件服务、压缩和目录列表
- **URL 处理与路由** - URL 重写、重定向和路由规则
- **标头与响应自定义** - 自定义标头和响应修改
- **安全与访问控制** - 身份验证、授权和访问限制
- **反向代理与负载均衡** - 代理配置和后端管理
- **正向代理** - 正向代理功能
- **身份验证转发** - 外部身份验证集成
- **CGI 与应用服务器** - CGI、FastCGI、SCGI 和其他网关接口配置
- **内容处理** - 响应正文修改和过滤
- **速率限制** - 请求速率限制和节流
- **日志记录** - 访问和错误日志记录配置

## 仅限全局的指令

### TLS/SSL 与安全

- `tls_cipher_suite <tls_cipher_suite: string> [<tls_cipher_suite_2: string> ...]`
  - 此指令指定支持的 TLS 密码套件。如果使用 HTTP/3 协议（在 Ferron 中是实验性的），则需要启用 `TLS_AES_128_GCM_SHA256` 密码套件（默认启用），否则 HTTP/3 服务器将根本无法启动。此指令可以多次指定。默认值：Rustls 的默认 TLS 密码套件
- `tls_ecdh_curves <ecdh_curve: string> [<ecdh_curve: string> ...]`
  - 此指令指定支持的 TLS ECDH 曲线。此指令可以多次指定。默认值：Rustls 的默认 ECDH 曲线
- `tls_client_certificate [tls_client_certificate: bool|string]`
  - 此指令指定是否启用 TLS 客户端证书验证。如果设置为 `#true`，将根据系统证书存储区验证客户端证书。如果设置为字符串，将根据指定路径中的证书颁发机构验证客户端证书。默认值：`tls_client_certificate #false`
- `tls_min_version <tls_min_version: string>`
  - 此指令指定服务器将接受的最低 TLS 版本（TLSv1.2 或 TLSv1.3）。默认值：`tls_min_version "TLSv1.2"`
- `tls_max_version <tls_max_version: string>`
  - 此指令指定服务器将接受的最高 TLS 版本（TLSv1.2 或 TLSv1.3）。默认值：`tls_max_version "TLSv1.3"`
- `ocsp_stapling [enable_ocsp_stapling: bool]`
  - 此指令指定是否启用 OCSP 装订。默认值：`ocsp_stapling #true`
- `auto_tls_on_demand_ask <auto_tls_on_demand_ask_url: string|null>`
  - 此指令指定用于询问是否允许按需自动 TLS 的主机名的 URL。服务器会将带有要颁发证书的域名的 `domain` 查询参数附加到该 URL。建议在使用按需自动 TLS 时配置此选项以防止滥用。默认值：`auto_tls_on_demand_ask #null`
- `auto_tls_on_demand_ask_no_verification [auto_tls_on_demand_ask_no_verification: bool]`
  - 此指令指定服务器是否不应验证按需自动 TLS 请求端点的 TLS 证书。默认值：`auto_tls_on_demand_ask_no_verification #false`

**配置示例：**

```kdl
* {
    tls_cipher_suite "TLS_AES_256_GCM_SHA384" "TLS_AES_128_GCM_SHA256"
    tls_ecdh_curves "secp256r1" "secp384r1"
    tls_client_certificate #false
    ocsp_stapling
    auto_tls_on_demand_ask "https://auth.example.com/check"
    auto_tls_on_demand_ask_no_verification #false
}
```

### HTTP 协议与性能

- `default_http_port <default_http_port: integer|null>`
  - 此指令指定 HTTP 连接的默认端口。如果设置为 `default_http_port #null`，则禁用隐式默认 HTTP 端口。默认值：`default_http_port 80`
- `default_https_port <default_https_port: integer|null>`
  - 此指令指定 HTTPS 连接的默认端口。如果设置为 `default_https_port #null`，则禁用隐式默认 HTTPS 端口。默认值：`default_https_port 443`
- `protocols <protocol: string> [<protocol: string> ...]`
  - 此指令指定 Web 服务器启用的协议。支持的协议是 `"h1"` (HTTP/1.x)、`"h2"` (HTTP/2) 和 `"h3"` (HTTP/3；实验性)。默认值：`protocols "h1" "h2"`
- `timeout <timeout: integer|null>`
  - 此指令指定服务器处理请求的最长时间（以毫秒为单位），之后服务器将重置连接。如果设置为 `timeout #null`，则禁用超时。不建议禁用超时，因为这可能会使服务器容易受到慢速 HTTP 攻击。默认值：`timeout 300000`
- `h2_initial_window_size <h2_initial_window_size: integer>`
  - 此指令指定 HTTP/2 初始窗口大小。默认值：Hyper 默认值
- `h2_max_frame_size <h2_max_frame_size: integer>`
  - 此指令指定最大 HTTP/2 帧大小。默认值：Hyper 默认值
- `h2_max_concurrent_streams <h2_max_concurrent_streams: integer>`
  - 此指令指定并发 HTTP/2 流的最大数量。默认值：Hyper 默认值
- `h2_max_header_list_size <h2_max_header_list_size: integer>`
  - 此指令指定最大 HTTP/2 帧大小。默认值：Hyper 默认值
- `h2_enable_connect_protocol [h2_enable_connect_protocol: bool]`
  - 此指令指定是否启用 HTTP/2 中的 CONNECT 协议。默认值：Hyper 默认值
- `protocol_proxy [enable_proxy_protocol: bool]`
  - 此指令指定是否启用 PROXY 协议接受。如果启用，服务器将在每个连接开始时期望 PROXY 协议头。默认值：`protocol_proxy #false`
- `buffer_request <request_buffer_size: integer|null>`
  - 此指令指定传入请求的缓冲区大小（以字节为单位）。如果设置为 `buffer_request #null`，则禁用请求缓冲区。请求缓冲区可以作为针对 Slowloris 式攻击的底层后端服务器的附加保护。默认值：`buffer_request #null`
- `buffer_response <response_buffer_size: integer|null>`
  - 此指令指定传出响应的缓冲区大小（以字节为单位）。如果设置为 `buffer_response #null`，则禁用响应缓冲区。默认值：`buffer_response #null`

**配置示例：**

```kdl
* {
    default_http_port 80
    default_https_port 443
    protocols "h1" "h2" "h3"
    timeout 300000
    h2_initial_window_size 65536
    h2_max_frame_size 16384
    h2_max_concurrent_streams 100
    h2_max_header_list_size 8192
    h2_enable_connect_protocol
    protocol_proxy #false
    buffer_request #null
    buffer_response #null
}
```

### 缓存

- `cache_max_entries <cache_max_entries: integer|null>` (_cache_ 模块)
  - 此指令指定可在 HTTP 缓存中存储的最大条目数。如果设置为 `cache_max_entries #null`，则缓存理论上可以存储无限数量的条目。条目的缓存键取决于请求方法、重写的请求 URL、“Host”标头值和变化的请求标头。默认值：`cache_max_entries 1024`

**配置示例：**

```kdl
* {
    cache_max_entries 2048
}
```

### 网络与系统

- `listen_ip <listen_ip: string>`
  - 此指令指定要侦听的 IP 地址。默认值：`listen_ip "::1"`
- `io_uring [enable_io_uring: bool]`
  - 此指令指定是否启用 `io_uring`。此指令对不支持 `io_uring` 的系统和使用 Tokio 而不是 Monoio 的 Web 服务器构建没有影响。默认值：`io_uring #true`
- `tcp_send_buffer <tcp_send_buffer: integer>`
  - 此指令指定 TCP 侦听器的发送缓冲区大小（以字节为单位）。默认值：无
- `tcp_recv_buffer <tcp_recv_buffer: integer>`
  - 此指令指定 TCP 侦听器的接收缓冲区大小（以字节为单位）。默认值：无

**配置示例：**

```kdl
* {
    listen_ip "0.0.0.0"
    io_uring
    tcp_send_buffer 65536
    tcp_recv_buffer 65536
}
```

## 全局和虚拟主机指令

### TLS/SSL 与安全

- `tls <certificate_path: string> <private_key_path: string>`
  - 此指令指定 TLS 证书和私钥的路径。默认值：无
- `auto_tls [enable_automatic_tls: bool]`
  - 此指令指定是否启用自动 TLS。默认值：当未明确指定端口且主机名看起来不像本地地址（`127.0.0.1`、`::1`、`localhost`）时为 `auto_tls #true`，否则为 `auto_tls #false`
- `auto_tls_contact <auto_tls_contact: string|null>`
  - 此指令指定用于为自动 TLS 注册 ACME 帐户的电子邮件地址。默认值：`auto_tls_contact #null`
- `auto_tls_cache <auto_tls_cache: string|null>`
  - 此指令指定用于存储缓存的 ACME 数据（例如缓存的帐户数据和证书）的目录。默认值：特定于操作系统的目录，例如在 GNU/Linux 上对于“user”用户可以是 `/home/user/.local/share/ferron-acme`，在 macOS 上对于“user”用户可以是 `/Users/user/Library/Application Support/ferron-acme`，在 Windows 上对于“user”用户可以是 `C:\Users\user\AppData\Local\ferron-acme`。在 Docker 上，它将是 `/var/lib/ferron-acme`。
- `auto_tls_letsencrypt_production [enable_auto_tls_letsencrypt_production: bool]`
  - 此指令指定是否使用生产 Let's Encrypt ACME 端点。如果设置为 `auto_tls_letsencrypt_production #false`，则使用暂存 Let's Encrypt ACME 端点。默认值：`auto_tls_letsencrypt_production #true`
- `auto_tls_challenge <acme_challenge_type: string> [provider=<acme_challenge_provider: string>] [...]`
  - 此指令指定使用的 ACME 质询类型。支持的类型有 `"http-01"`（HTTP-01 ACME 质询）、`"tls-alpn-01"`（TLS-ALPN-01 ACME 质询）和 `"dns-01"`（DNS-01 ACME 质询）。`provider` 属性定义了用于 DNS-01 质询的 DNS 提供程序。可以将其他属性作为 DNS 提供程序的参数传递，请参阅自动 TLS 文档。默认值：`auto_tls_challenge "tls-alpn-01"`
- `auto_tls_directory <auto_tls_directory: string>`
  - 此指令指定从中获取证书的 ACME 目录 URL。覆盖 `auto_tls_letsencrypt_production` 指令。默认值：无
- `auto_tls_no_verification [auto_tls_no_verification: bool]`
  - 此指令指定是否禁用 ACME 服务器的证书验证。默认值：`auto_tls_no_verification #false`
- `auto_tls_profile <auto_tls_profile: string|null>`
  - 此指令指定用于证书的 ACME 配置文件。默认值：`auto_tls_profile #null`
- `auto_tls_on_demand <auto_tls_on_demand: bool>`
  - 此指令指定是否启用按需自动 TLS。该功能在网站首次访问时自动获取 TLS 证书。建议使用 HTTP-01 或 TLS-ALPN-01 ACME 质询，因为 DNS-01 ACME 质询可能由于 DNS 传播延迟而较慢。还建议将此指令与 `auto_tls_on_demand_ask` 指令一起配置。默认值：`auto_tls_on_demand #false`
- `auto_tls_eab (<auto_tls_eab_key_id: string> <auto_tls_eab_key_hmac: string>)|<auto_tls_eab_disabled: null>`
  - 此指令指定 ACME 外部帐户绑定的 EAB 密钥 ID 和 HMAC。HMAC 密钥值以 URL 安全的 Base64 编码。如果设置为 `auto_tls_eab_disabled #null`，则禁用 EAB。默认值：`auto_tls_eab_disabled #null`

**配置示例：**

```kdl
example.com {
    auto_tls
    auto_tls_contact "admin@example.com"
    auto_tls_cache "/var/cache/ferron-acme"
    auto_tls_letsencrypt_production
    auto_tls_challenge "tls-alpn-01"
    auto_tls_profile "default"
    auto_tls_on_demand #false
    auto_tls_eab #null
}

manual-tls.example.com {
    tls "/etc/ssl/certs/example.com.crt" "/etc/ssl/private/example.com.key"
}
```

## 指令

### 标头与响应自定义

- `header <header_name: string> <header_value: string>`
  - 此指令指定要添加到 HTTP 响应中的标头。标头值支持占位符，如 `{path}`，它将被替换为请求路径。此指令可以多次指定。默认值：无
- `server_administrator_email <server_administrator_email: string>`
  - 此指令指定要在默认的 500 内部服务器错误页面中使用的服务器管理员的电子邮件地址。默认值：无
- `error_page <status_code: integer> <path: string>`
  - 此指令指定要由 Web 服务器提供的自定义错误页面。默认值：无
- `header_remove <header_name: string>`
  - 此指令指定要从 HTTP 响应中删除的标头。此指令可以多次指定。默认值：无
- `header_replace <header_name: string> <header_value: string>`
  - 此指令指定要添加到 HTTP 响应中的标头，可能会替换现有标头。标头值支持占位符，如 `{path}`，它将被替换为请求路径。此指令可以多次指定。默认值：无

**配置示例：**

```kdl
example.com {
    header "X-Frame-Options" "DENY"
    header "X-Content-Type-Options" "nosniff"
    header "X-XSS-Protection" "1; mode=block"
    header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"
    header "X-Custom-Header" "Custom value with {path} placeholder"

    header_remove "X-Header-To-Remove"
    header_replace "X-Powered-By" "Ferron"

    server_administrator_email "admin@example.com"
    error_page 404 "/var/www/errors/404.html"
    error_page 500 "/var/www/errors/500.html"
}
```

### 安全与访问控制

- `trust_x_forwarded_for [trust_x_forwarded_for: bool]`
  - 此指令指定是否信任 `X-Forwarded-For` 标头的值。如果位于反向代理之后，建议配置此指令。默认值：`trust_x_forwarded_for #false`
- `status <status_code: integer> [url=<url: string>|regex=<regex: string>] [location=<location: string>] [realm=<realm: string>] [brute_protection=<enable_brute_protection: bool>] [users=<users: string>] [allowed=<allowed: string>] [not_allowed=<not_allowed: string>] [body=<response_body: string>]`
  - 此指令指定自定义状态代码。此指令可以多次指定。`url` 属性指定此状态代码的请求路径。`regex` 属性指定自定义状态代码的正则表达式（如 `^/ferron(?:$|[/#?])`）。`location` 属性指定重定向的目标；它支持占位符，如 `{path}`，它将被替换为请求路径。`realm` 属性指定 HTTP 基本身份验证领域。`brute_protection` 属性指定是否启用暴力破解保护。`users` 属性是用于 HTTP 身份验证的允许用户的逗号分隔列表。`allowed` 属性是适用于状态代码的 IP 地址的逗号分隔列表。`not_allowed` 属性是不适用于状态代码的 IP 地址的逗号分隔列表。`body` 属性指定要发送的响应正文。默认值：无
- `user [username: string] [password_hash: string]`
  - 此指令指定一个用户及其密码哈希，用于 HTTP 基本身份验证（可以是 Argon2、PBKDF2 或 `scrypt` 之一）。建议使用 `ferron-passwd` 工具生成密码哈希。此指令可以多次指定。默认值：无
- `block (<blocked_ip: string> [<blocked_ip: string> ...])|<not_specified: null>`
  - 此指令指定要阻止的 IP 地址和 CIDR 范围。如果设置为 `block #null`，则忽略此指令。在 Ferron 2.1.0 之前，此指令仅为全局指令。此指令可以多次指定。默认值：无
- `allow (<allowed_ip: string> [<allowed_ip: string> ...])|<not_specified: null>`
  - 此指令指定要允许的 IP 地址和 CIDR 范围。如果设置为 `allow #null`，则忽略此指令。在 Ferron 2.1.0 之前，此指令仅为全局指令。此指令可以多次指定。默认值：无

**配置示例：**

```kdl
example.com {
    trust_x_forwarded_for

    // 带有自定义状态代码的基本身份验证
    status 401 url="/admin" realm="Admin Area" users="admin,moderator"
    status 403 url="/restricted" allowed="192.168.1.0/24" body="Access denied"
    status 301 url="/old-page" location="/new-page"

    // 身份验证的用户定义（使用 `ferron-passwd` 生成密码哈希）
    user "admin" "$2b$10$hashedpassword12345"
    user "moderator" "$2b$10$anotherhashedpassword"

    // 限制谁可以访问该站点
    block "192.168.1.100" "10.0.0.5"
    allow "192.168.1.0/24" "10.0.0.0/8"
}
```

### URL 处理与路由

- `allow_double_slashes [allow_double_slashes: bool]`
  - 此指令指定是否允许在 URL 中使用双斜杠。默认值：`allow_double_slashes #false`
- `no_redirect_to_https [no_redirect_to_https: bool]`
  - 此指令指定是否不从 HTTP URL 重定向到 HTTPS URL。当在配置中明确指定服务器端口时，此指令始终有效设置为 `no_redirect_to_https`。默认值：`no_redirect_to_https #false`
- `wwwredirect [enable_wwwredirect: bool]`
  - 此指令指定是否从不带“www.”的 URL 重定向到带“www.”的 URL。默认值：`wwwredirect #false`
- `rewrite <regex: string> <replacement: string> [directory=<directory: bool>] [file=<file: bool>] [last=<last: bool>] [allow_double_slashes=<allow_double_slashes: bool>]`
  - 此指令指定 URL 重写规则。此指令可以多次指定。第一个值是正则表达式（如 `^/ferron(?:$|[/#?])`）。`directory` 属性指定当路径对应于目录时是否应用重写规则（如果为 `#false`，则不应用）。`file` 属性指定当路径对应于文件时是否应用重写规则（如果为 `#false`，则不应用）。`last` 属性指定重写规则是否为应用的最后一条规则。`allow_double_slashes` 属性指定重写规则是否允许请求 URL 中有双斜杠。默认值：无
- `rewrite_log [rewrite_log: bool]`
  - 此指令指定是否将 URL 重写操作记录到错误日志中。默认值：`rewrite_log #false`
- `no_trailing_redirect [no_trailing_redirect: bool]`
  - 此指令指定当 URL 指向目录时，是否不将不带尾部斜杠的 URL 重定向到带尾部斜杠的 URL。默认值：`no_trailing_redirect #false`

**配置示例：**

```kdl
example.com {
    allow_double_slashes #false
    no_redirect_to_https #false
    wwwredirect #false

    // URL 重写示例
    rewrite "^/old-path/(.*)" "/new-path/$1" last=#true
    rewrite "^/api/v1/(.*)" "/api/v2/$1" file=#false directory=#false
    rewrite "^/blog/([^/]+)/?(?:$|[?#])" "/blog.php?slug=$1" last=#true

    rewrite_log
    no_trailing_redirect #false
}
```

### 静态文件服务

- `root <webroot: string|null>`
  - 此指令指定提供静态文件的 Web 根目录。如果设置为 `root #null`，则禁用静态文件服务功能。默认值：无
- `etag [enable_etag: bool]` (_static_ 模块)
  - 此指令指定是否启用 ETag 标头。默认值：`etag #true`
- `compressed [enable_compression: bool]` (_static_ 模块)
  - 此指令指定是否为静态文件启用 HTTP 压缩。默认值：`compressed #true`
- `directory_listing [enable_directory_listing: bool]` (_static_ 模块)
  - 此指令指定是否启用目录列表。默认值：`directory_listing #false`
- `precompressed [enable_precompression: bool]` (_static_ 模块)
  - 此指令指定是否启用提供预压缩的静态文件。预压缩的静态文件将另外具有 `.gz`（gzip）、`.deflate`（Deflate）、`.br`（Brotli）或 `.zst`（Zstandard）扩展名。默认值：`precompressed #false`
- `mime_type <file_extension: string> <mime_type: string>` (_static_ 模块；Ferron 2.1.0 或更高版本)
  - 此指令为静态文件指定与文件扩展名（如 `.html`）对应的附加 MIME 类型。默认值：无
- `index <index_file: string> [<another_index_file: string> ...]` (_static_ 模块；Ferron 2.1.0 或更高版本)
  - 此指令指定当请求目录时要使用的索引文件。默认值：`index "index.html" "index.htm" "index.html"`（静态文件服务），`index "index.php" "index.cgi" "index.html" "index.htm" "index.html"`（CGI、FastCGI）
- `dynamic_compressed [enable_dynamic_content_compression: bool]` (_dcompress_ 模块；Ferron 2.1.0 或更高版本)
  - 此指令指定是否为动态内容启用 HTTP 压缩。默认值：`dynamic_compressed #false`

**配置示例：**

```kdl
example.com {
    root "/var/www/example.com"
    etag
    compressed
    directory_listing #false

    // 为静态文件设置“Cache-Control”标头
    file_cache_control "public, max-age=3600"
}
```

### 缓存

- `cache [enable_cache: bool]` (_cache_ 模块)
  - 此指令指定是否启用 HTTP 缓存。默认值：`cache #false`
- `cache_max_response_size <cache_max_response_size: integer|null>` (_cache_ 模块)
  - 此指令指定可存储在 HTTP 缓存中的响应的最大大小（以字节为单位）。如果设置为 `cache_max_response_size #null`，则缓存理论上可以存储任何大小的响应。默认值：`cache_max_response_size 2097152`
- `cache_vary <varying_request_header: string> [<varying_request_header: string> ...]` (_cache_ 模块)
  - 此指令指定用于区分缓存条目的请求标头。此指令可以多次指定。默认值：无
- `cache_ignore <ignored_response_header: string> [<ignored_response_header: string> ...]` (_cache_ 模块)
  - 此指令指定在缓存响应时忽略的响应标头。此指令可以多次指定。默认值：无
- `file_cache_control <cache_control: string|null>` (_static_ 模块)
  - 此指令指定静态文件的 Cache-Control 标头值。如果设置为 `file_cache_control #null`，则不设置 Cache-Control 标头。默认值：`file_cache_control #null`

**配置示例：**

```kdl
example.com {
    cache
    cache_max_response_size 2097152
    cache_vary "Accept-Encoding" "Accept-Language"
    cache_ignore "Set-Cookie" "Cache-Control"
}
```

### 反向代理与负载均衡

- `proxy <proxy_to: string|null> [unix=<unix_socket_path: string>]` (_rproxy_ 模块)
  - 此指令指定反向代理应将请求转发到的 URL。支持 HTTP（例如 `http://localhost:3000/`）和 HTTPS URL（例如 `https://localhost:3000/`）。还通过将 `unix` 属性设置为套接字路径来支持 Unix 套接字（主值设置为网站的 URL），仅在 Unix 和类 Unix 系统上受支持。此指令可以多次指定。默认值：无
- `lb_health_check [enable_lb_health_check: bool]` (_rproxy_ 模块)
  - 此指令指定是否启用负载均衡器被动健康检查。默认值：`lb_health_check #false`
- `lb_health_check_max_fails <max_fails: integer>` (_rproxy_ 模块)
  - 此指令指定负载均衡器将后端标记为不健康之前的最大连续失败次数。默认值：`lb_health_check_max_fails 3`
- `proxy_no_verification [proxy_no_verification: bool]` (_rproxy_ 模块)
  - 此指令指定反向代理是否不应验证后端的 TLS 证书。默认值：`proxy_no_verification #false`
- `proxy_intercept_errors [proxy_intercept_errors: bool]` (_rproxy_ 模块)
  - 此指令指定反向代理是否应拦截来自后端的错误。默认值：`proxy_intercept_errors #false`
- `proxy_request_header <header_name: string> <header_value: string>` (_rproxy_ 模块)
  - 此指令指定要添加到由反向代理发送的 HTTP 请求中的标头。标头值支持占位符，如 `{path}`，它将被替换为请求路径。此指令可以多次指定。默认值：无
- `proxy_request_header_remove <header_name: string>` (_rproxy_ 模块)
  - 此指令指定要从由反向代理发送的 HTTP 请求中删除的标头。此指令可以多次指定。默认值：无
- `proxy_keepalive [proxy_keepalive: bool]` (_rproxy_ 模块)
  - 此指令指定反向代理是否应保持与后端的连接活动。默认值：`proxy_keepalive #true`
- `proxy_request_header_replace <header_name: string> <header_value: string>` (_rproxy_ 模块)
  - 此指令指定要添加到由反向代理发送的 HTTP 请求中的标头，可能会替换现有标头。标头值支持占位符，如 `{path}`，它将被替换为请求路径。此指令可以多次指定。默认值：无
- `proxy_http2 [enable_proxy_http2: bool]` (_rproxy_ 模块)
  - 此指令指定反向代理在连接到后端服务器时是否可以使用 HTTP/2 协议。此指令仅在后端服务器支持 HTTP/2 并通过 HTTPS 连接时才有效。默认值：`proxy_http2 #false`
- `lb_retry_connection [enable_lb_retry_connection: bool]` (_rproxy_ 模块)
  - 此指令指定在 TCP 连接或 TLS 握手失败的情况下，负载均衡器是否应重试连接到另一个后端服务器。默认值：`lb_retry_connection #true`
- `lb_algorithm <lb_algorithm: string>` (_rproxy_ 模块)
  - 此指令指定要使用的负载均衡算法。支持的算法有 `random`（随机选择）、`round-robin`（轮询）、`least_conn`（最少连接，“连接”此处指并发请求）和 `two_random`（两次随机选择；两次随机选择后，选择并发请求最少的后端服务器）。默认值：`lb_algorithm "two_random"`
- `lb_health_check_window <lb_health_check_window: integer>` (_rproxy_ 模块)
  - 此指令指定负载均衡器健康检查的窗口大小（以毫秒为单位）。默认值：`lb_health_check_window 5000`
- `proxy_keepalive_idle_conns <proxy_keepalive_idle_conns: integer>` (_rproxy_ 模块)
  - 此指令指定要保持活动的到后端服务器的最大空闲连接数。默认值：`proxy_keepalive_idle_conns 48`
- `proxy_http2_only [enable_proxy_http2_only: bool]` (_rproxy_ 模块；Ferron 2.1.0 或更高版本)
  - 此指令指定反向代理在连接到后端服务器时是否使用 HTTP/2 协议（无 HTTP/1.1 回退）。当后端服务器通过 HTTPS 连接时，反向代理在 TLS 握手期间协商 HTTP/2。当后端服务器通过 HTTP 连接时，反向代理使用具有先验知识的 HTTP/2。此指令可用于代理 gRPC 请求。默认值：`proxy_http2_only #false`
- `proxy_proxy_header <proxy_version_version: string|null>` (_rproxy_ 模块；Ferron 2.1.0 或更高版本)
  - 此指令指定在充当反向代理时要发送到后端服务器的 PROXY 协议头的版本。支持的版本是 `"v1"`（PROXY 协议版本 1）和 `"v2"`（PROXY 协议版本 2）。如果指定为 `#null` 值，则不发送 PROXY 协议头。默认值：`proxy_proxy_header #null`

**配置示例：**

```kdl
api.example.com {
    // 用于负载均衡的后端
    // （您也可以通过仅指定一个 `proxy` 指令来使用单个后端）
    proxy "http://backend1:8080"
    proxy "http://backend2:8080"
    proxy "http://backend3:8080"

    // 健康检查配置
    lb_health_check
    lb_health_check_max_fails 3
    lb_health_check_window 5000

    // 代理设置
    proxy_no_verification #false
    proxy_intercept_errors #false
    proxy_keepalive
    proxy_http2 #false

    // 代理标头
    proxy_request_header "X-Custom-Header" "CustomValue"

    proxy_request_header_remove "X-Internal-Token"
    proxy_request_header_replace "X-Real-IP" "{client_ip}"
}
```

### 正向代理

- `forward_proxy [enable_forward_proxy: bool]` (_fproxy_ 模块)
  - 此指令指定是否启用正向代理功能。默认值：`forward_proxy #false`

**配置示例：**

```kdl
* {
    forward_proxy
}
```

### 身份验证转发

- `auth_to <auth_to: string|null>` (_fauth_ 模块)
  - 此指令指定 Web 服务器应将请求发送到哪个 URL 进行转发身份验证。默认值：无
- `auth_to_no_verification [auth_to_no_verification: bool]` (_fauth_ 模块)
  - 此指令指定服务器是否不应验证后端身份验证服务器的 TLS 证书。默认值：`auth_to_no_verification #false`
- `auth_to_copy <request_header_to_copy: string> [<request_header_to_copy: string> ...]` (_fauth_ 模块)
  - 此指令指定将复制并发送到转发身份验证后端服务器的请求标头。此指令可以多次指定。默认值：无

**配置示例：**

```kdl
app.example.com {
    // 将身份验证转发到外部服务
    auth_to "https://auth.example.com/validate"
    auth_to_no_verification #false
    auth_to_copy "Authorization" "X-User-Token" "X-Session-ID"
}
```

### CGI 与应用服务器

- `cgi [enable_cgi: bool]` (_cgi_ 模块)
  - 此指令指定是否启用 CGI 处理程序。默认值：`cgi #false`
- `cgi_extension <cgi_extension: string|null>` (_cgi_ 模块)
  - 此指令指定 CGI 脚本扩展名，将通过 CGI 处理程序在 `cgi-bin` 目录之外处理。此指令可以多次指定。默认值：无
- `cgi_interpreter <cgi_extension: string> <cgi_interpreter: string|null> [<cgi_interpreter_argument: string> ...]` (_cgi_ 模块)
  - 此指令指定 CGI 处理程序使用的 CGI 脚本解释器。如果 CGI 解释器设置为 `#null`，则将禁用默认解释器设置。此指令可以多次指定。默认值：为 `.pl`、`.py`、`.sh`、`.ksh`、`.csh`、`.rb` 和 `.php` 扩展名指定，对于 Windows，还为 `.exe`、`.bat` 和 `.vbs` 扩展名指定
- `cgi_environment <environment_variable_name: string> <environment_variable_value: string>` (_cgi_ 模块)
  - 此指令指定传递到 CGI 应用程序的环境变量。此指令可以多次指定。默认值：无
- `scgi <scgi_to: string|null>` (_scgi_ 模块)
  - 此指令指定是否启用 SCGI 以及 SCGI 客户端将向其发送请求的基本 URL。支持 TCP（例如 `tcp://localhost:4000/`）和 Unix 套接字 URL（仅在 Unix 系统上；例如 `unix:///run/scgi.sock`）。默认值：`scgi #null`
- `scgi_environment <environment_variable_name: string> <environment_variable_value: string>` (_scgi_ 模块)
  - 此指令指定传递到 SCGI 服务器的环境变量。此指令可以多次指定。默认值：无
- `fcgi <fcgi_to: string|null> [pass=<fcgi_pass: bool>]` (_fcgi_ 模块)
  - 此指令指定是否启用 FastCGI 以及 FastCGI 客户端将为“.php”文件发送请求的基本 URL。`pass` 属性指定是否将所有请求传递给 FastCGI 请求处理程序。支持 TCP（例如 `tcp://localhost:4000/`）和 Unix 套接字 URL（仅在 Unix 系统上；例如 `unix:///run/scgi.sock`）。默认值：`fcgi #null pass=#true`
- `fcgi_php <fcgi_php_to: string|null>` (_fcgi_ 模块)
  - 此指令指定是否通过 FastCGI 启用 PHP 以及 FastCGI 客户端将为“.php”文件发送请求的基本 URL。支持 TCP（例如 `tcp://localhost:4000/`）和 Unix 套接字 URL（仅在 Unix 系统上；例如 `unix:///run/scgi.sock`）。默认值：`fcgi_php #null`
- `fcgi_extension <fcgi_extension: string|null>` (_fcgi_ 模块)
  - 此指令指定文件扩展名，将通过 FastCGI 处理程序处理。此指令可以多次指定。默认值：无
- `fcgi_environment <environment_variable_name: string> <environment_variable_value: string>` (_fcgi_ 模块)
  - 此指令指定传递到 FastCGI 服务器的环境变量。此指令可以多次指定。默认值：无

**配置示例：**

```kdl
cgi.example.com {
    // CGI 配置
    cgi
    cgi_extension ".cgi" ".pl" ".py"
    cgi_interpreter ".py" "/usr/bin/python3"
    cgi_interpreter ".pl" "/usr/bin/perl"
    cgi_environment "PATH" "/usr/bin:/bin"
    cgi_environment "SCRIPT_ROOT" "/var/www/cgi-bin"
}

scgi.example.com {
    // SCGI 配置
    scgi "tcp://localhost:4000/"
    scgi_environment "SCRIPT_NAME" "/app"
    scgi_environment "SERVER_NAME" "example.com"
}

fastcgi.example.com {
    // FastCGI 配置
    fcgi "tcp://localhost:9000/" pass=#true
    fcgi_php "tcp://localhost:9000/"
    fcgi_extension ".php" ".php5"
    fcgi_environment "DOCUMENT_ROOT" "/var/www/example.com"
}
```

### 内容处理

字符串替换需要禁用 HTTP 压缩。

- `replace <searched_string: string> <replaced_string: string> [once=<replace_once: bool>]` (_replace_ 模块)
  - 此指令指定要在响应正文中替换的字符串和替换字符串。`once` 属性指定字符串是否只替换一次，默认情况下此属性设置为 `#true`。此指令可以多次指定。默认值：无
- `replace_last_modified [preserve_last_modified: bool]` (_replace_ 模块)
  - 此指令指定是否在响应中保留“Last-Modified”标头。默认值：`replace_last_modified #false`
- `replace_filter_types <filter_type: string> [<filter_type: string> ...]` (_replace_ 模块)
  - 此指令指定响应 MIME 类型过滤器。过滤器可以是特定的 MIME 类型（如 `text/html`）或通配符 (`*`)，指定处理所有 MIME 类型的响应以进行替换。此指令可以多次指定。默认值：`replace_filter_types "text/html"`

**配置示例：**

```kdl
example.com {
    // 字符串替换需要禁用 HTTP 压缩
    compressed #false

    // 响应正文中的字符串替换（在禁用 HTTP 压缩的情况下工作）
    replace "old-company-name" "new-company-name" once=#false
    replace "http://old-domain.com" "https://new-domain.com" once=#true

    replace_last_modified
    replace_filter_types "text/html" "text/css" "application/javascript"
}
```

### 速率限制

- `limit [enable_limit: bool] [rate=<rate: integer|float>] [burst=<rate: integer|float>]` (_limit_ 模块)
  - 此指令指定是否启用速率限制。`rate` 属性指定每秒最大平均请求数，默认为每秒 25 个请求。`burst` 属性指定每秒最大峰值请求数，默认为最大平均请求数的 4 倍。默认值：`limit #false`

**配置示例：**

```kdl
example.com {
    // 全局速率限制
    limit rate=100 burst=200

    // 不同路径的不同速率限制
    location "/api" {
        limit rate=10 burst=20
    }

    location "/login" {
        limit rate=5 burst=10
    }
}
```

### 日志记录

- `log_date_format <log_date_format: string>`
  - 此指令指定访问日志文件的日期格式（根据 POSIX）。默认值：`"%d/%b/%Y:%H:%M:%S %z"`
- `log_format <log_format: string>`
  - 此指令指定访问日志文件的条目格式。可以在指定部分的参考中找到占位符。默认值：`"{client_ip} - {auth_user} [{timestamp}] \"{method} {path_and_query} {version}\" {status_code} {content_length} \"{header:Referer}\" \"{header:User-Agent}\""`（组合日志格式）
- `log <log_file_path: string>` (_logfile_ 可观察性后端)
  - 此指令指定访问日志文件的路径，该文件包含组合日志格式的 HTTP 响应日志。在 Ferron 2.2.0 之前，此指令是全局和虚拟主机指令。默认值：无
- `error_log <error_log_file_path: string>` (_logfile_ 可观察性后端)
  - 此指令指定错误日志文件的路径。在 Ferron 2.2.0 之前，此指令是全局和虚拟主机指令。默认值：无
- `otlp_no_verification [otlp_no_verification: bool]` (_otlp_ 可观察性后端；Ferron 2.2.0 或更高版本)
  - 此指令指定服务器是否不应验证 OTLP (OpenTelemetry Protocol) 端点的 TLS 证书。默认值：`otlp_no_verification #false`
- `otlp_service_name <otlp_service_name: string>` (_otlp_ 可观察性后端；Ferron 2.2.0 或更高版本)
  - 此指令指定要在 OTLP (OpenTelemetry Protocol) 端点中使用的服务名称。默认值：`otlp_service_name "ferron"`
- `otlp_logs <otlp_logs_endpoint: string|null> [authorization=<otlp_logs_authorization: string>] [protocol=<otlp_logs_protocol: string>]` (_otlp_ 可观察性后端；Ferron 2.2.0 或更高版本)
  - 此指令指定用于将日志记录到 OTLP (OpenTelemetry Protocol) 端点的端点 URL。如果使用 HTTP 协议，`authorization` 属性是 `Authorization` HTTP 标头的值。`protocol` 属性指定要使用的协议（`grpc` 表示 gRPC，`http/protobuf` 表示使用 protobuf 数据的 HTTP，`http/json` 表示使用 JSON 数据的 HTTP）。支持 HTTP 和 HTTPS（仅限基于 HTTP 的协议）URL。默认值：`otlp_logs #null protocol="grpc"`
- `otlp_metrics <otlp_metrics_endpoint: string|null> [authorization=<otlp_metrics_authorization: string>] [protocol=<otlp_metrics_protocol: string>]` (_otlp_ 可观察性后端；Ferron 2.2.0 或更高版本)
  - 此指令指定用于将指标记录到 OTLP (OpenTelemetry Protocol) 端点的端点 URL。如果使用 HTTP 协议，`authorization` 属性是 `Authorization` HTTP 标头的值。`protocol` 属性指定要使用的协议（`grpc` 表示 gRPC，`http/protobuf` 表示使用 protobuf 数据的 HTTP，`http/json` 表示使用 JSON 数据的 HTTP）。支持 HTTP 和 HTTPS（仅限基于 HTTP 的协议）URL。默认值：`otlp_metrics #null protocol="grpc"`
- `otlp_traces <otlp_traces_endpoint: string|null> [authorization=<otlp_traces_authorization: string>] [protocol=<otlp_traces_protocol: string>]` (_otlp_ 可观察性后端；Ferron 2.2.0 或更高版本)
  - 此指令指定用于将跟踪记录到 OTLP (OpenTelemetry Protocol) 端点的端点 URL。如果使用 HTTP 协议，`authorization` 属性是 `Authorization` HTTP 标头的值。`protocol` 属性指定要使用的协议（`grpc` 表示 gRPC，`http/protobuf` 表示使用 protobuf 数据的 HTTP，`http/json` 表示使用 JSON 数据的 HTTP）。支持 HTTP 和 HTTPS（仅限基于 HTTP 的协议）URL。默认值：`otlp_traces #null protocol="grpc"`

**配置示例：**

```kdl
* {
    log_date_format "%d/%b/%Y:%H:%M:%S %z"
    log_format "{client_ip} - {auth_user} [{timestamp}] \"{method} {path_and_query} {version}\" {status_code} {content_length} \"{header:Referer}\" \"{header:User-Agent}\""
}

example.com {
    log "/var/log/ferron/example.com.access.log"
    error_log "/var/log/ferron/example.com.error.log"
}
```

## 子条件

Ferron 2.0.0 及更高版本支持基于条件的条件配置。这允许您根据请求方法、路径或其他条件配置不同的设置。

以下是支持的子条件列表：

- `is_remote_ip <remote_ip: string> [<remote_ip: string> ...]`
  - 此子条件检查请求是否来自特定的远程 IP 地址或 IP 地址列表。
- `is_forwarded_for <remote_ip: string> [<remote_ip: string> ...]`
  - 此子条件检查请求（关于 `X-Forwarded-For` 标头）是否来自特定的转发 IP 地址或 IP 地址列表。
- `is_not_remote_ip <remote_ip: string> [<remote_ip: string> ...]`
  - 此子条件检查请求是否不是来自特定的远程 IP 地址或 IP 地址列表。
- `is_not_forwarded_for <remote_ip: string> [<remote_ip: string> ...]`
  - 此子条件检查请求（关于 `X-Forwarded-For` 标头）是否不是来自特定的转发 IP 地址或 IP 地址列表。
- `is_equal <left_side: string> <right_side: string>`
  - 此子条件检查左侧是否等于右侧。
- `is_not_equal <left_side: string> <right_side: string>`
  - 此子条件检查左侧是否不等于右侧。
- `is_regex <value: string> <regex: string> [case_insensitive=<case_insensitive: bool>]`
  - 此子条件检查值是否与正则表达式匹配。`case_insensitive` 属性指定正则表达式是否应不区分大小写（默认为 `#false`）。
- `is_not_regex <value: string> <regex: string> [case_insensitive=<case_insensitive: bool>]`
  - 此子条件检查值是否不与正则表达式匹配。`case_insensitive` 属性指定正则表达式是否应不区分大小写（默认为 `#false`）。
- `is_rego <rego_policy: string>`
  - 此子条件评估 Rego 策略。
- `set_constant <key: string> <value: string>` (Ferron 2.1.0 或更高版本)
  - 此子条件设置一个常量值。
- `is_language <language: string>` (Ferron 2.1.0 或更高版本)
  - 此子条件检查语言是否是 `Accept-Language` 标头中指定的首选语言。此子条件使用 `LANGUAGES` 常量，这是一个以逗号分隔的首选语言代码列表（例如 `en-US` 或 `fr-FR`）。

## Rego 子条件

Ferron 2.0.0 及更高版本支持使用嵌入在 Ferron 配置中的 Rego 策略实现更高级的子条件。

在为 Ferron 子条件编写 Rego 策略时，您需要将包名称设置为 `ferron`（`package ferron` 行）。Ferron 检查策略的 `pass` 结果。

基于 Rego 的子条件（`input`）的输入如下：

- `input.method` (string) - 请求的 HTTP 方法（`GET`、`POST` 等）
- `input.uri` (string) - 请求的 URI（例如 `/index.php?page=1`）
- `input.headers` (array<string, array<string>>) - 请求的标头。标头名称为小写。
- `input.socket_data.client_ip` (string) - 客户端的 IP 地址。
- `input.socket_data.client_port` (number) - 客户端的端口。
- `input.socket_data.server_ip` (string) - 服务器的 IP 地址。
- `input.socket_data.server_port` (number) - 服务器的端口。
- `input.socket_data.encrypted` (boolean) - 连接是否已加密。
- `input.constants` (array<string, string>; Ferron 2.1.0 或更高版本) - 由 `set_constant` 子条件设置的常量。

您可以在 [Open Policy Agent 文档](https://www.openpolicyagent.org/docs/policy-language) 中阅读有关 Rego 的更多信息。

**利用 Rego 子条件的配置示例（拒绝 `curl` 请求）：**

```kdl
// 将 "example.com" 替换为您的域名。
example.com {
  condition "DENY_CURL" {
    is_rego """
      package ferron

      default pass := false

      pass := true if {
        input.headers["user-agent"][0] == "curl"
      }

      pass := true if {
        startswith(input.headers["user-agent"], "curl/")
      }
      """
  }

  if "DENY_CURL" {
    status 403
  }

  // 提供静态文件
  root "/var/www/html"
}
```

## 占位符

Ferron 支持以下用于标头值、子条件、反向代理和重定向目标的占位符：

- `{path}` - 带路径的请求 URI（例如 `/index.html`）
- `{path_and_query}` - 带路径和查询字符串的请求 URI（例如 `/index.html?param=value`）
- `{method}` - 请求方法
- `{version}` - 请求的 HTTP 版本
- `{header:<header_name>}` - 请求 URI 的标头值
- `{scheme}` - 请求 URI 的方案（`http` 或 `https`），仅适用于子条件、反向代理和重定向目标。
- `{client_ip}` - 客户端 IP 地址，仅适用于子条件、反向代理和重定向目标。
- `{client_port}` - 客户端端口号，仅适用于子条件、反向代理和重定向目标。
- `{server_ip}` - 服务器 IP 地址，仅适用于子条件、反向代理和重定向目标。
- `{server_port}` - 服务器端口号，仅适用于子条件、反向代理和重定向目标。

## 日志占位符

Ferron 2.0.0 及更高版本支持以下用于访问日志的占位符：

- `{path}` - 带路径的请求 URI（例如 `/index.html`）
- `{path_and_query}` - 带路径和查询字符串的请求 URI（例如 `/index.html?param=value`）
- `{method}` - 请求方法
- `{version}` - 请求的 HTTP 版本
- `{header:<header_name>}` - 请求 URI 的标头值（如果标头丢失，则为 `-`）
- `{scheme}` - 请求 URI 的方案（`http` 或 `https`）。
- `{client_ip}` - 客户端 IP 地址。
- `{client_port}` - 客户端端口号。
- `{server_ip}` - 服务器 IP 地址。
- `{server_port}` - 服务器端口号。
- `{auth_user}` - 经过身份验证的用户的用户名（如果未经过身份验证，则为 `-`）
- `{timestamp}` - 条目的格式化时间戳
- `{status_code}` - 响应的 HTTP 状态代码
- `{content_length}` - 响应的内容长度（如果不可用，则为 `-`）

## 特定语言内容配置示例

以下是涉及条件以提供特定语言内容的 Ferron 配置示例：

```kdl
// 通用语言设置的代码段
snippet "LANG_COMMON" {
    set_constant "LANGUAGES" "en,de,pl"
}

example.com {
    // 通用响应
    status 200 body="lang: Unknown"

    condition "LANG_PL" {
        use "LANG_COMMON"
        is_language "pl"
    }

    condition "LANG_DE" {
        use "LANG_COMMON"
        is_language "de"
    }

    condition "LANG_EN" {
        use "LANG_COMMON"
        is_language "en"
    }

    if "LANG_PL" {
        // 波兰语
        status 200 body="lang: Polski"
    }

    if "LANG_DE" {
        // 德语
        status 200 body="lang: Deutsch"
    }

    if "LANG_EN" {
        // 英语
        status 200 body="lang: English"
    }
}
```

## 位置块示例

以下是涉及位置块的 Ferron 配置示例：

```kdl
example.com {
    root "/var/www/example.com"

    // 具有不同设置的静态资产
    location "/static" remove_base=#true {
        root "/var/www/static"
        compressed
        file_cache_control "public, max-age=31536000"
    }

    // 带代理的 API 端点
    location "/api" {
        proxy "http://backend:8080"
        proxy_request_header "X-Forwarded-For" "{client_ip}"
        limit rate=50 burst=100
    }

    // 带有身份验证的管理区域
    location "/admin" {
        status 401 realm="Admin Area" users="admin,superuser"
        root "/var/www/admin"
    }

    // PHP 文件
    fcgi_php "tcp://localhost:9000/"
}
```

## 组合多个部分的完整示例

以下是组合多个部分的 Ferron 配置的完整示例：

```kdl
// 全局配置
* {
    // 协议和性能设置
    protocols "h1" "h2"
    h2_initial_window_size 65536
    h2_max_concurrent_streams 100
    timeout 300000

    // 网络设置
    listen_ip "0.0.0.0"
    default_http_port 80
    default_https_port 443

    // 安全默认值
    tls_cipher_suite "TLS_AES_256_GCM_SHA384" "TLS_AES_128_GCM_SHA256"
    ocsp_stapling
    block "192.168.1.100"

    // 全局缓存
    cache_max_entries 1024

    // 负载均衡设置
    lb_health_check_window 5000

    // 日志记录
    log "/var/log/ferron/access.log"
    error_log "/var/log/ferron/error.log"

    // 静态文件默认值
    compressed
    etag
}

// 定义可重用的代码段
snippet "security_headers" {
    // 通用安全标头
    header "X-Frame-Options" "DENY"
    header "X-Content-Type-Options" "nosniff"
    header "X-XSS-Protection" "1; mode=block"
    header "Referrer-Policy" "strict-origin-when-cross-origin"
}

snippet "cors_headers" {
    // API 端点的 CORS 标头
    header "Access-Control-Allow-Origin" "*"
    header "Access-Control-Allow-Methods" "GET, POST, PUT, DELETE, OPTIONS"
    header "Access-Control-Allow-Headers" "Authorization, Content-Type, X-Requested-With"
    header "Access-Control-Max-Age" "86400"
}

snippet "static_caching" {
    // 静态资产的积极缓存
    file_cache_control "public, max-age=31536000, immutable"
    compressed
    etag
}

snippet "admin_protection" {
    // 管理区域保护
    status 401 realm="Admin Area" users="admin,superuser"
    user "admin" "$2b$10$hashedpassword12345"
    user "superuser" "$2b$10$anotherhashpassword67890"
    limit rate=10 burst=20
}

snippet "mobile_condition" {
    condition "is_mobile" {
        is_regex "{header:User-Agent}" "(Mobile|Android|iPhone|iPad)" case_insensitive=#true
    }
}

snippet "admin_ip_condition" {
    condition "is_admin_ip" {
        is_remote_ip "192.168.1.10" "10.0.0.5"
    }
}

snippet "api_request_condition" {
    condition "is_api_request" {
        is_regex "{path}" "^/api/"
    }
}

snippet "static_asset_condition" {
    condition "is_static_asset" {
        is_regex "{path}" "\\.(css|js|png|jpg|jpeg|gif|svg|woff|woff2|ttf|eot|ico)(?:$|[?#])" case_insensitive=#true
    }
}

snippet "development_condition" {
    condition "is_development" {
        is_equal "{header:X-Environment}" "development"
    }
}

// 带有条件配置的主网站
example.com {
    // TLS 配置
    tls "/etc/ssl/certs/example.com.crt" "/etc/ssl/private/example.com.key"
    auto_tls_contact "admin@example.com"

    // 基本设置
    root "/var/www/example.com"
    server_administrator_email "admin@example.com"

    // 使用安全标头代码段
    use "security_headers"

    // 导入条件代码段
    use "mobile_condition"
    use "admin_ip_condition"
    use "api_request_condition"
    use "static_asset_condition"
    use "development_condition"

    // 附加安全标头
    header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

    // 基于移动设备检测的条件配置
    if "is_mobile" {
        // 添加未继承的标头
        use "security_headers"
        header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

        // 移动设备特定设置
        header "X-Mobile-Detected" "true"
        root "/var/www/example.com/mobile"

        // 移动设备的较轻速率限制
        limit rate=50 burst=100
    }

    if_not "is_mobile" {
        // 添加未继承的标头
        use "security_headers"
        header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

        // 桌面设置
        header "X-Mobile-Detected" "false"
        limit rate=100 burst=200
    }

    // 管理员 IP 获得特殊处理
    if "is_admin_ip" {
        // 添加未继承的标头
        use "security_headers"
        header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

        // 对管理员 IP 不进行速率限制
        limit #false

        // 附加调试标头
        header "X-Admin-Access" "true"
        header "X-Client-IP" "{client_ip}"

        // 增强管理员访问日志记录
        log "/var/log/ferron/admin-access.log"
    }

    // 开发环境条件设置
    if "is_development" {
        // 添加未继承的标头
        use "security_headers"
        header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

        // 开发特定标头
        header "X-Environment" "development"
        header "X-Debug-Mode" "enabled"

        // 在开发中禁用缓存
        header "Cache-Control" "no-cache, no-store, must-revalidate"
        header "Pragma" "no-cache"
        header "Expires" "0"
    }

    if_not "is_development" {
        // 添加未继承的标头
        use "security_headers"
        header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

        // 生产缓存
        cache
        cache_vary "Accept-Encoding" "Accept-Language"
        file_cache_control "public, max-age=3600"
    }

    // URL 重写
    rewrite "^/old-section/(.*)" "/new-section/$1" last=#true

    // 错误页面
    error_page 404 "/var/www/errors/404.html"
    error_page 500 "/var/www/errors/500.html"

    // 带有条件缓存的静态资产位置
    location "/assets" remove_base=#true {
        root "/var/www/assets"

        if "is_static_asset" {
            // 添加未继承的标头
            use "security_headers"
            header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

            use "static_caching"
        }

        // Web 字体的 CORS
        if_not "is_static_asset" {
            // 添加未继承的标头
            use "security_headers"
            header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

            header "Access-Control-Allow-Origin" "*"
        }
    }

    // API 端点
    location "/api" {
        if "is_api_request" {
            // 添加未继承的标头
            use "security_headers"
            header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

            use "cors_headers"

            // API 特定速率限制
            limit rate=1000 burst=2000

            // 代理到 API 后端
            proxy "http://api-backend:8080"
            proxy_request_header_replace "X-Real-IP" "{client_ip}"
            proxy_request_header "X-Forwarded-Proto" "{scheme}"
        }
    }

    // 带有条件访问的管理区域
    location "/admin" {
        if "is_admin_ip" {
            // 添加未继承的标头
            use "security_headers"
            header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

            // 管理员 IP 直接访问
            root "/var/www/admin"

            // 特殊管理员标头
            header "X-Admin-Direct-Access" "true"
        }

        if_not "is_admin_ip" {
            // 添加未继承的标头
            use "security_headers"
            header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

            // 非管理员 IP 需要身份验证
            use "admin_protection"
        }
    }

    // PHP 应用程序
    fcgi_php "tcp://localhost:9000/"
}

// 具有条件配置的主网站
example.com {
    // TLS 配置
    tls "/etc/ssl/certs/example.com.crt" "/etc/ssl/private/example.com.key"
    auto_tls_contact "admin@example.com"

    // 基本设置
    root "/var/www/example.com"
    server_administrator_email "admin@example.com"

    // 使用安全标头代码段
    use "security_headers"

    // 导入条件代码段
    use "mobile_condition"
    use "admin_ip_condition"
    use "api_request_condition"
    use "static_asset_condition"
    use "development_condition"

    // 附加安全标头
    header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

    // 基于移动设备检测的条件配置
    if "is_mobile" {
        // 添加未继承的标头
        use "security_headers"
        header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

        // 移动设备特定设置
        header "X-Mobile-Detected" "true"
        root "/var/www/example.com/mobile"

        // 移动设备的较轻速率限制
        limit rate=50 burst=100
    }

    if_not "is_mobile" {
        // 添加未继承的标头
        use "security_headers"
        header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

        // 桌面设置
        header "X-Mobile-Detected" "false"
        limit rate=100 burst=200
    }

    // 管理员 IP 获得特殊处理
    if "is_admin_ip" {
        // 添加未继承的标头
        use "security_headers"
        header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

        // 对管理员 IP 不进行速率限制
        limit #false

        // 附加调试标头
        header "X-Admin-Access" "true"
        header "X-Client-IP" "{client_ip}"

        // 增强管理员访问日志记录
        log "/var/log/ferron/admin-access.log"
    }

    // 开发环境条件设置
    if "is_development" {
        // 添加未继承的标头
        use "security_headers"
        header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

        // 开发特定标头
        header "X-Environment" "development"
        header "X-Debug-Mode" "enabled"

        // 在开发中禁用缓存
        header "Cache-Control" "no-cache, no-store, must-revalidate"
        header "Pragma" "no-cache"
        header "Expires" "0"
    }

    if_not "is_development" {
        // 添加未继承的标头
        use "security_headers"
        header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

        // 生产缓存
        cache
        cache_vary "Accept-Encoding" "Accept-Language"
        file_cache_control "public, max-age=3600"
    }

    // URL 重写
    rewrite "^/old-section/(.*)" "/new-section/$1" last=#true

    // 错误页面
    error_page 404 "/var/www/errors/404.html"
    error_page 500 "/var/www/errors/500.html"

    // 带有条件缓存的静态资产位置
    location "/assets" remove_base=#true {
        root "/var/www/assets"

        if "is_static_asset" {
            // 添加未继承的标头
            use "security_headers"
            header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

            use "static_caching"
        }

        // Web 字体的 CORS
        if_not "is_static_asset" {
            // 添加未继承的标头
            use "security_headers"
            header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

            header "Access-Control-Allow-Origin" "*"
        }
    }

    // API 端点
    location "/api" {
        if "is_api_request" {
            // 添加未继承的标头
            use "security_headers"
            header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

            use "cors_headers"

            // API 特定速率限制
            limit rate=1000 burst=2000

            // 代理到 API 后端
            proxy "http://api-backend:8080"
            proxy_request_header_replace "X-Real-IP" "{client_ip}"
            proxy_request_header "X-Forwarded-Proto" "{scheme}"
        }
    }

    // 带有条件访问的管理区域
    location "/admin" {
        if "is_admin_ip" {
            // 添加未继承的标头
            use "security_headers"
            header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

            // 管理员 IP 直接访问
            root "/var/www/admin"

            // 特殊管理员标头
            header "X-Admin-Direct-Access" "true"
        }

        if_not "is_admin_ip" {
            // 添加未继承的标头
            use "security_headers"
            header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

            // 非管理员 IP 需要身份验证
            use "admin_protection"
        }
    }

    // PHP 应用程序
    fcgi_php "tcp://localhost:9000/"
}

// 具有条件配置的主网站
example.com {
    // TLS 配置
    tls "/etc/ssl/certs/example.com.crt" "/etc/ssl/private/example.com.key"
    auto_tls_contact "admin@example.com"

    // 基本设置
    root "/var/www/example.com"
    server_administrator_email "admin@example.com"

    // 使用安全标头代码段
    use "security_headers"

    // 导入条件代码段
    use "mobile_condition"
    use "admin_ip_condition"
    use "api_request_condition"
    use "static_asset_condition"
    use "development_condition"

    // 附加安全标头
    header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

    // 基于移动设备检测的条件配置
    if "is_mobile" {
        // 添加未继承的标头
        use "security_headers"
        header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

        // 移动设备特定设置
        header "X-Mobile-Detected" "true"
        root "/var/www/example.com/mobile"

        // 移动设备的较轻速率限制
        limit rate=50 burst=100
    }

    if_not "is_mobile" {
        // 添加未继承的标头
        use "security_headers"
        header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

        // 桌面设置
        header "X-Mobile-Detected" "false"
        limit rate=100 burst=200
    }

    // 管理员 IP 获得特殊处理
    if "is_admin_ip" {
        // 添加未继承的标头
        use "security_headers"
        header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

        // 对管理员 IP 不进行速率限制
        limit #false

        // 附加调试标头
        header "X-Admin-Access" "true"
        header "X-Client-IP" "{client_ip}"

        // 增强管理员访问日志记录
        log "/var/log/ferron/admin-access.log"
    }

    // 开发环境条件设置
    if "is_development" {
        // 添加未继承的标头
        use "security_headers"
        header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

        // 开发特定标头
        header "X-Environment" "development"
        header "X-Debug-Mode" "enabled"

        // 在开发中禁用缓存
        header "Cache-Control" "no-cache, no-store, must-revalidate"
        header "Pragma" "no-cache"
        header "Expires" "0"
    }

    if_not "is_development" {
        // 添加未继承的标头
        use "security_headers"
        header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

        // 生产缓存
        cache
        cache_vary "Accept-Encoding" "Accept-Language"
        file_cache_control "public, max-age=3600"
    }

    // URL 重写
    rewrite "^/old-section/(.*)" "/new-section/$1" last=#true

    // 错误页面
    error_page 404 "/var/www/errors/404.html"
    error_page 500 "/var/www/errors/500.html"

    // 带有条件缓存的静态资产位置
    location "/assets" remove_base=#true {
        root "/var/www/assets"

        if "is_static_asset" {
            // 添加未继承的标头
            use "security_headers"
            header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

            use "static_caching"
        }

        // Web 字体的 CORS
        if_not "is_static_asset" {
            // 添加未继承的标头
            use "security_headers"
            header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

            header "Access-Control-Allow-Origin" "*"
        }
    }

    // API 端点
    location "/api" {
        if "is_api_request" {
            // 添加未继承的标头
            use "security_headers"
            header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

            use "cors_headers"

            // API 特定速率限制
            limit rate=1000 burst=2000

            // 代理到 API 后端
            proxy "http://api-backend:8080"
            proxy_request_header_replace "X-Real-IP" "{client_ip}"
            proxy_request_header "X-Forwarded-Proto" "{scheme}"
        }
    }

    // 带有条件访问的管理区域
    location "/admin" {
        if "is_admin_ip" {
            // 添加未继承的标头
            use "security_headers"
            header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

            // 管理员 IP 直接访问
            root "/var/www/admin"

            // 特殊管理员标头
            header "X-Admin-Direct-Access" "true"
        }

        if_not "is_admin_ip" {
            // 添加未继承的标头
            use "security_headers"
            header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

            // 非管理员 IP 需要身份验证
            use "admin_protection"
        }
    }

    // PHP 应用程序
    fcgi_php "tcp://localhost:9000/"
}

// 具有条件配置的主网站
example.com {
    // TLS 配置
    tls "/etc/ssl/certs/example.com.crt" "/etc/ssl/private/example.com.key"
    auto_tls_contact "admin@example.com"

    // 基本设置
    root "/var/www/example.com"
    server_administrator_email "admin@example.com"

    // 使用安全标头代码段
    use "security_headers"

    // 导入条件代码段
    use "mobile_condition"
    use "admin_ip_condition"
    use "api_request_condition"
    use "static_asset_condition"
    use "development_condition"

    // 附加安全标头
    header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

    // 基于移动设备检测的条件配置
    if "is_mobile" {
        // 添加未继承的标头
        use "security_headers"
        header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

        // 移动设备特定设置
        header "X-Mobile-Detected" "true"
        root "/var/www/example.com/mobile"

        // 移动设备的较轻速率限制
        limit rate=50 burst=100
    }

    if_not "is_mobile" {
        // 添加未继承的标头
        use "security_headers"
        header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

        // 桌面设置
        header "X-Mobile-Detected" "false"
        limit rate=100 burst=200
    }

    // 管理员 IP 获得特殊处理
    if "is_admin_ip" {
        // 添加未继承的标头
        use "security_headers"
        header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

        // 对管理员 IP 不进行速率限制
        limit #false

        // 附加调试标头
        header "X-Admin-Access" "true"
        header "X-Client-IP" "{client_ip}"

        // 增强管理员访问日志记录
        log "/var/log/ferron/admin-access.log"
    }

    // 开发环境条件设置
    if "is_development" {
        // 添加未继承的标头
        use "security_headers"
        header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

        // 开发特定标头
        header "X-Environment" "development"
        header "X-Debug-Mode" "enabled"

        // 在开发中禁用缓存
        header "Cache-Control" "no-cache, no-store, must-revalidate"
        header "Pragma" "no-cache"
        header "Expires" "0"
    }

    if_not "is_development" {
        // 添加未继承的标头
        use "security_headers"
        header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

        // 生产缓存
        cache
        cache_vary "Accept-Encoding" "Accept-Language"
        file_cache_control "public, max-age=3600"
    }

    // URL 重写
    rewrite "^/old-section/(.*)" "/new-section/$1" last=#true

    // 错误页面
    error_page 404 "/var/www/errors/404.html"
    error_page 500 "/var/www/errors/500.html"

    // 带有条件缓存的静态资产位置
    location "/assets" remove_base=#true {
        root "/var/www/assets"

        if "is_static_asset" {
            // 添加未继承的标头
            use "security_headers"
            header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

            use "static_caching"
        }

        // Web 字体的 CORS
        if_not "is_static_asset" {
            // 添加未继承的标头
            use "security_headers"
            header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

            header "Access-Control-Allow-Origin" "*"
        }
    }

    // API 端点
    location "/api" {
        if "is_api_request" {
            // 添加未继承的标头
            use "security_headers"
            header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

            use "cors_headers"

            // API 特定速率限制
            limit rate=1000 burst=2000

            // 代理到 API 后端
            proxy "http://api-backend:8080"
            proxy_request_header_replace "X-Real-IP" "{client_ip}"
            proxy_request_header "X-Forwarded-Proto" "{scheme}"
        }
    }

    // 带有条件访问的管理区域
    location "/admin" {
        if "is_admin_ip" {
            // 添加未继承的标头
            use "security_headers"
            header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

            // 管理员 IP 直接访问
            root "/var/www/admin"

            // 特殊管理员标头
            header "X-Admin-Direct-Access" "true"
        }

        if_not "is_admin_ip" {
            // 添加未继承的标头
            use "security_headers"
            header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

            // 非管理员 IP 需要身份验证
            use "admin_protection"
        }
    }

    // PHP 应用程序
    fcgi_php "tcp://localhost:9000/"
}

// 具有条件配置的主网站
example.com {
    // TLS 配置
    tls "/etc/ssl/certs/example.com.crt" "/etc/ssl/private/example.com.key"
    auto_tls_contact "admin@example.com"

    // 基本设置
    root "/var/www/example.com"
    server_administrator_email "admin@example.com"

    // 使用安全标头代码段
    use "security_headers"

    // 导入条件代码段
    use "mobile_condition"
    use "admin_ip_condition"
    use "api_request_condition"
    use "static_asset_condition"
    use "development_condition"

    // 附加安全标头
    header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

    // 基于移动设备检测的条件配置
    if "is_mobile" {
        // 添加未继承的标头
        use "security_headers"
        header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

        // 移动设备特定设置
        header "X-Mobile-Detected" "true"
        root "/var/www/example.com/mobile"

        // 移动设备的较轻速率限制
        limit rate=50 burst=100
    }

    if_not "is_mobile" {
        // 添加未继承的标头
        use "security_headers"
        header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

        // 桌面设置
        header "X-Mobile-Detected" "false"
        limit rate=100 burst=200
    }

    // 管理员 IP 获得特殊处理
    if "is_admin_ip" {
        // 添加未继承的标头
        use "security_headers"
        header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

        // 对管理员 IP 不进行速率限制
        limit #false

        // 附加调试标头
        header "X-Admin-Access" "true"
        header "X-Client-IP" "{client_ip}"

        // 增强管理员访问日志记录
        log "/var/log/ferron/admin-access.log"
    }

    // 开发环境条件设置
    if "is_development" {
        // 添加未继承的标头
        use "security_headers"
        header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

        // 开发特定标头
        header "X-Environment" "development"
        header "X-Debug-Mode" "enabled"

        // 在开发中禁用缓存
        header "Cache-Control" "no-cache, no-store, must-revalidate"
        header "Pragma" "no-cache"
        header "Expires" "0"
    }

    if_not "is_development" {
        // 添加未继承的标头
        use "security_headers"
        header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

        // 生产缓存
        cache
        cache_vary "Accept-Encoding" "Accept-Language"
        file_cache_control "public, max-age=3600"
    }

    // URL 重写
    rewrite "^/old-section/(.*)" "/new-section/$1" last=#true

    // 错误页面
    error_page 404 "/var/www/errors/404.html"
    error_page 500 "/var/www/errors/500.html"

    // 带有条件缓存的静态资产位置
    location "/assets" remove_base=#true {
        root "/var/www/assets"

        if "is_static_asset" {
            // 添加未继承的标头
            use "security_headers"
            header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

            use "static_caching"
        }

        // Web 字体的 CORS
        if_not "is_static_asset" {
            // 添加未继承的标头
            use "security_headers"
            header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

            header "Access-Control-Allow-Origin" "*"
        }
    }

    // API 端点
    location "/api" {
        if "is_api_request" {
            // 添加未继承的标头
            use "security_headers"
            header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

            use "cors_headers"

            // API 特定速率限制
            limit rate=1000 burst=2000

            // 代理到 API 后端
            proxy "http://api-backend:8080"
            proxy_request_header_replace "X-Real-IP" "{client_ip}"
            proxy_request_header "X-Forwarded-Proto" "{scheme}"
        }
    }

    // 带有条件访问的管理区域
    location "/admin" {
        if "is_admin_ip" {
            // 添加未继承的标头
            use "security_headers"
            header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

            // 管理员 IP 直接访问
            root "/var/www/admin"

            // 特殊管理员标头
            header "X-Admin-Direct-Access" "true"
        }

        if_not "is_admin_ip" {
            // 添加未继承的标头
            use "security_headers"
            header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

            // 非管理员 IP 需要身份验证
            use "admin_protection"
        }
    }

    // PHP 应用程序
    fcgi_php "tcp://localhost:9000/"
}

// 具有条件配置的主网站
example.com {
    // TLS 配置
    tls "/etc/ssl/certs/example.com.crt" "/etc/ssl/private/example.com.key"
    auto_tls_contact "admin@example.com"

    // 基本设置
    root "/var/www/example.com"
    server_administrator_email "admin@example.com"

    // 使用安全标头代码段
    use "security_headers"

    // 导入条件代码段
    use "mobile_condition"
    use "admin_ip_condition"
    use "api_request_condition"
    use "static_asset_condition"
    use "development_condition"

    // 附加安全标头
    header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

    // 基于移动设备检测的条件配置
    if "is_mobile" {
        // 添加未继承的标头
        use "security_headers"
        header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

        // 移动设备特定设置
        header "X-Mobile-Detected" "true"
        root "/var/www/example.com/mobile"

        // 移动设备的较轻速率限制
        limit rate=50 burst=100
    }

    if_not "is_mobile" {
        // 添加未继承的标头
        use "security_headers"
        header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

        // 桌面设置
        header "X-Mobile-Detected" "false"
        limit rate=100 burst=200
    }

    // 管理员 IP 获得特殊处理
    if "is_admin_ip" {
        // 添加未继承的标头
        use "security_headers"
        header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

        // 对管理员 IP 不进行速率限制
        limit #false

        // 附加调试标头
        header "X-Admin-Access" "true"
        header "X-Client-IP" "{client_ip}"

        // 增强管理员访问日志记录
        log "/var/log/ferron/admin-access.log"
    }

    // 开发环境条件设置
    if "is_development" {
        // 添加未继承的标头
        use "security_headers"
        header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

        // 开发特定标头
        header "X-Environment" "development"
        header "X-Debug-Mode" "enabled"

        // 在开发中禁用缓存
        header "Cache-Control" "no-cache, no-store, must-revalidate"
        header "Pragma" "no-cache"
        header "Expires" "0"
    }

    if_not "is_development" {
        // 添加未继承的标头
        use "security_headers"
        header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

        // 生产缓存
        cache
        cache_vary "Accept-Encoding" "Accept-Language"
        file_cache_control "public, max-age=3600"
    }

    // URL 重写
    rewrite "^/old-section/(.*)" "/new-section/$1" last=#true

    // 错误页面
    error_page 404 "/var/www/errors/404.html"
    error_page 500 "/var/www/errors/500.html"

    // 带有条件缓存的静态资产位置
    location "/assets" remove_base=#true {
        root "/var/www/assets"

        if "is_static_asset" {
            // 添加未继承的标头
            use "security_headers"
            header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

            use "static_caching"
        }

        // Web 字体的 CORS
        if_not "is_static_asset" {
            // 添加未继承的标头
            use "security_headers"
            header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

            header "Access-Control-Allow-Origin" "*"
        }
    }

    // API 端点
    location "/api" {
        if "is_api_request" {
            // 添加未继承的标头
            use "security_headers"
            header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

            use "cors_headers"

            // API 特定速率限制
            limit rate=1000 burst=2000

            // 代理到 API 后端
            proxy "http://api-backend:8080"
            proxy_request_header_replace "X-Real-IP" "{client_ip}"
            proxy_request_header "X-Forwarded-Proto" "{scheme}"
        }
    }

    // 带有条件访问的管理区域
    location "/admin" {
        if "is_admin_ip" {
            // 添加未继承的标头
            use "security_headers"
            header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

            // 管理员 IP 直接访问
            root "/var/www/admin"

            // 特殊管理员标头
            header "X-Admin-Direct-Access" "true"
        }

        if_not "is_admin_ip" {
            // 添加未继承的标头
            use "security_headers"
            header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

            // 非管理员 IP 需要身份验证
            use "admin_protection"
        }
    }

    // PHP 应用程序
    fcgi_php "tcp://localhost:9000/"
}

// 具有条件配置的主网站
example.com {
    // TLS 配置
    tls "/etc/ssl/certs/example.com.crt" "/etc/ssl/private/example.com.key"
    auto_tls_contact "admin@example.com"

    // 基本设置
    root "/var/www/example.com"
    server_administrator_email "admin@example.com"

    // 使用安全标头代码段
    use "security_headers"

    // 导入条件代码段
    use "mobile_condition"
    use "admin_ip_condition"
    use "api_request_condition"
    use "static_asset_condition"
    use "development_condition"

    // 附加安全标头
    header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

    // 基于移动设备检测的条件配置
    if "is_mobile" {
        // 添加未继承的标头
        use "security_headers"
        header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

        // 移动设备特定设置
        header "X-Mobile-Detected" "true"
        root "/var/www/example.com/mobile"

        // 移动设备的较轻速率限制
        limit rate=50 burst=100
    }

    if_not "is_mobile" {
        // 添加未继承的标头
        use "security_headers"
        header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

        // 桌面设置
        header "X-Mobile-Detected" "false"
        limit rate=100 burst=200
    }

    // 管理员 IP 获得特殊处理
    if "is_admin_ip" {
        // 添加未继承的标头
        use "security_headers"
        header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

        // 对管理员 IP 不进行速率限制
        limit #false

        // 附加调试标头
        header "X-Admin-Access" "true"
        header "X-Client-IP" "{client_ip}"

        // 增强管理员访问日志记录
        log "/var/log/ferron/admin-access.log"
    }

    // 开发环境条件设置
    if "is_development" {
        // 添加未继承的标头
        use "security_headers"
        header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

        // 开发特定标头
        header "X-Environment" "development"
        header "X-Debug-Mode" "enabled"

        // 在开发中禁用缓存
        header "Cache-Control" "no-cache, no-store, must-revalidate"
        header "Pragma" "no-cache"
        header "Expires" "0"
    }

    if_not "is_development" {
        // 添加未继承的标头
        use "security_headers"
        header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

        // 生产缓存
        cache
        cache_vary "Accept-Encoding" "Accept-Language"
        file_cache_control "public, max-age=3600"
    }

    // URL 重写
    rewrite "^/old-section/(.*)" "/new-section/$1" last=#true

    // 错误页面
    error_page 404 "/var/www/errors/404.html"
    error_page 500 "/var/www/errors/500.html"

    // 带有条件缓存的静态资产位置
    location "/assets" remove_base=#true {
        root "/var/www/assets"

        if "is_static_asset" {
            // 添加未继承的标头
            use "security_headers"
            header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

            use "static_caching"
        }

        // Web 字体的 CORS
        if_not "is_static_asset" {
            // 添加未继承的标头
            use "security_headers"
            header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

            header "Access-Control-Allow-Origin" "*"
        }
    }

    // API 端点
    location "/api" {
        if "is_api_request" {
            // 添加未继承的标头
            use "security_headers"
            header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

            use "cors_headers"

            // API 特定速率限制
            limit rate=1000 burst=2000

            // 代理到 API 后端
            proxy "http://api-backend:8080"
            proxy_request_header_replace "X-Real-IP" "{client_ip}"
            proxy_request_header "X-Forwarded-Proto" "{scheme}"
        }
    }

    // 带有条件访问的管理区域
    location "/admin" {
        if "is_admin_ip" {
            // 添加未继承的标头
            use "security_headers"
            header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

            // 管理员 IP 直接访问
            root "/var/www/admin"

            // 特殊管理员标头
            header "X-Admin-Direct-Access" "true"
        }

        if_not "is_admin_ip" {
            // 添加未继承的标头
            use "security_headers"
            header "Strict-Transport-Security" "max-age=31536000; includeSubDomains"

            // 非管理员 IP 需要身份验证
            use "admin_protection"
        }
    }

    // PHP 应用程序
    fcgi_php "tcp://localhost:9000/"
}
```