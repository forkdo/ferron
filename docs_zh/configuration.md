---
title: 服务器配置 (旧版)
---

Ferron 也可以在与 Ferron 1.x 兼容的 `ferron.yaml` 文件中进行配置。以下是此服务器配置属性的说明。

## 仅全局配置属性

- **port** (_u16_ 或 _String_)
  - 服务器侦听的 HTTP 端口或地址-端口组合。这是服务器将接受传入 HTTP 连接的主端口。默认值：`80`
- **sport** (_u16_ 或 _String_)
  - 服务器侦听的 HTTPS 端口或地址-端口组合。这是服务器将接受传入 HTTPS 连接的主端口。默认值：`443`
- **secure** (_bool_)
  - 启用 HTTPS 的选项。设置为 `true` 时，服务器将使用 HTTPS 进行安全通信。默认值：`false`
- **http2Settings** (_Object_)
  - HTTP/2 设置。此对象包含与 HTTP/2 协议配置相关的各种设置。默认值：无
  - **子属性**:
    - **initialWindowSize** (_u32_)
      - HTTP/2 的初始窗口大小。此设置控制 HTTP/2 连接的初始流控制窗口大小。默认值：无
    - **maxFrameSize** (_u32_)
      - HTTP/2 的最大帧大小。此设置确定服务器将接受的最大帧有效负载。默认值：无
    - **maxConcurrentStreams** (_u32_)
      - HTTP/2 的最大并发流数。此设置限制了在任何给定时间可以打开的并发流的数量。默认值：无
    - **maxHeaderListSize** (_u32_)
      - HTTP/2 的最大标头列表大小。此设置控制服务器将接受的标头列表的最大大小。默认值：无
    - **enableConnectProtocol** (_bool_)
      - 启用 HTTP/2 CONNECT 协议。设置为 `true` 时，服务器将支持用于隧道 TCP 连接的 CONNECT 方法。默认值：`false`
- **logFilePath** (_String_)
  - 日志文件的路径。此设置指定服务器将写入其日志的文件路径。Ferron 写入的日志使用组合日志格式。默认值：无
- **errorLogFilePath** (_String_)
  - 错误日志文件的路径。此设置指定服务器将写入其错误日志的文件路径。默认值：无
- **enableHTTP2** (_bool_)
  - 启用 HTTP/2 的选项。设置为 `true` 时，服务器将支持 HTTP/2 协议。默认值：`true`
- **cert** (_String_)
  - TLS 证书的路径。此设置指定用于 HTTPS 连接的 TLS 证书的文件路径。默认值：无
- **key** (_String_)
  - 私钥的路径。此设置指定与 TLS 证书关联的私钥的文件路径。默认值：无
- **sni** (_Object_)
  - SNI 证书和密钥数据。此对象包含用于服务器名称指示 (SNI) 的证书和密钥数据，SNI 允许在同一 IP 地址上使用多个 SSL 证书。对象键是 SNI 主机名，而对象值是可以指定证书和私钥的对象。默认值：无
  - **子属性**:
    - **cert** (_String_)
      - SNI 证书的路径。此设置指定 SNI 证书的文件路径。默认值：无
    - **key** (_String_)
      - SNI 私钥的路径。此设置指定与 SNI 证书关联的私钥的文件路径。默认值：无
- **loadModules** (_Array&lt;String&gt;_)
  - 要加载的模块。此设置指定服务器在启动时应加载的模块数组。模块处理程序按此属性中定义的要加载的模块列表的顺序执行（例如，如果首先指定了 `cache` 模块，然后是 `fcgi`，则 `cache` 模块处理程序将首先执行，然后是 `fcgi`）。默认值：无
- **useClientCertificate** (_bool_)
  - 要求客户端提供其证书的选项。设置为 `true` 时，服务器将要求客户端出示有效的证书以进行身份验证。默认值：`false`
- **cipherSuite** (_Array&lt;String&gt;_)
  - 密码套件列表。此设置指定服务器将支持用于加密连接的密码套件数组。默认值：无
- **ecdhCurve** (_Array&lt;String&gt;_)
  - ECDH 曲线列表。此设置指定服务器将支持用于 ECDH 密钥交换的椭圆曲线数组。默认值：无
- **tlsMinVersion** (_String_)
  - 最低 TLS 版本 (TLSv1.2 或 TLSv1.3)。此设置指定服务器将接受的最低 TLS 版本。默认值：`"TLSv1.2"`
- **tlsMaxVersion** (_String_)
  - 最高 TLS 版本 (TLSv1.2 或 TLSv1.3)。此设置指定服务器将接受的最高 TLS 版本。默认值：`"TLSv1.3"`
- **disableNonEncryptedServer** (_bool_)
  - 如果 HTTPS 服务器正在运行，则禁用 HTTP 服务器的选项。设置为 `true` 时，服务器将仅接受 HTTPS 连接并禁用 HTTP 服务器。默认值：`false`
- **blocklist** (_Array&lt;String&gt;_)
  - IP 阻止列表。此设置指定服务器将阻止访问其服务的 IP 地址数组。阻止列表仅适用于非正向代理请求。默认值：无
- **enableOCSPStapling** (_bool_)
  - 启用 OCSP 装订的选项。设置为 `true` 时，服务器将使用 OCSP 装订向客户端提供证书吊销状态。具有 `Must-Staple` 扩展的证书在启用自动 TLS 的情况下将无法工作。默认值：`true`
- **environmentVariables** (_Object_)
  - 环境变量。此对象包含服务器在运行期间将使用的环境变量。默认值：无
- **enableAutomaticTLS** (_bool_)
  - 通过 Let's Encrypt 启用自动 TLS 的选项。自动 TLS 将使用 TLS-ALPN-01 ACME 质询。证书的域名将从主机配置中提取（通配符域被忽略，因为 TLS-ALPN-01 ACME 质询不支持它们）。当 HTTPS 端口设置为 `443` 时，自动 TLS 将起作用。默认值：`false`
- **automaticTLSContactEmail** (_String_)
  - 自动 TLS 用于在 Let's Encrypt 中创建帐户的电子邮件地址。Let's Encrypt 可以使用此电子邮件地址发送通知。默认值：无
- **automaticTLSContactCacheDirectory** (_String_)
  - 自动 TLS 用于存储缓存数据（例如缓存的证书）的目录路径。默认值：无
- **automaticTLSLetsEncryptProduction** (_bool_)
  - 启用生产 Let's Encrypt ACME 端点的选项。如果设置为 `false`，将使用暂存 Let's Encrypt ACME 端点。默认值：`true`
- **loadBalancerHealthCheckWindow** (_u32_; _rproxy_ 模块)
  - 负载均衡器报告的每个失败连接之间的时间窗口（以毫秒为单位）。默认值：`5000`
- **timeout** (_u32_ 或 `null`)
  - 服务器处理请求的最长时间（以毫秒为单位），超过此时间服务器将重置连接。如果设置为 `null`，则禁用超时。不建议禁用超时，因为禁用它可能会使服务器容易受到慢速 HTTP 攻击。默认值：`300000`
- **maximumCacheEntries** (_u32_ 或 `null`; _cache_ 模块)
  - 可在缓存中存储的最大缓存条目数。如果设置为 `null`，缓存理论上可以存储无限的条目。条目的缓存键取决于请求方法、请求 URL、“Host”标头值和变化的请求标头。默认值：`1024`
- **useAutomaticTLSHTTPChallenge** (_bool_; Ferron 1.1.0 及更高版本)
  - 为自动 TLS 启用基于 HTTP 的 ACME 质询的选项。设置为 `true` 时，服务器将使用 HTTP-01 ACME 质询而不是 TLS-ALPN-01 质询。默认值：`false`
- **enableHTTP3** (_bool_; Ferron 1.1.0 及更高版本)
  - 启用 HTTP/3 的选项。设置为 `true` 时，服务器将支持 HTTP/3 协议。目前，HTTP/3 支持是实验性的。需要启用 `TLS_AES_128_GCM_SHA256`，否则 HTTP/3 服务器将根本无法启动。默认值：`false`
- **wsgiClearModuleImportPath** (_bool_; _wsgi_ 模块; Ferron 1.1.0 及更高版本)
  - 启用清除 Python 模块导入路径的选项。将此选项设置为 `true` 可提高与涉及多个 WSGI 应用程序的设置的兼容性，但在 WSGI 应用程序中不得在函数内部使用模块导入。默认值：`false`
- **asgiClearModuleImportPath** (_bool_; _asgi_ 模块; Ferron 1.1.0 及更高版本)
  - 启用清除 Python 模块导入路径的选项。将此选项设置为 `true` 可提高与涉及多个 ASGI 应用程序的设置的兼容性，但在 ASGI 应用程序中不得在函数内部使用模块导入。默认值：`false`

## 主机配置属性

- **locations** (_Array&lt;Object&gt;_)
  - 为特定主机指定的位置列表。此属性的子属性将合并到组合的服务器配置中。URL 不会被重写。要重写 URL，请在位置配置中配置 URL 重写映射。默认值：无
- **errorConfig** (_Array&lt;Object&gt;_; Ferron 1.3.0 及更高版本)
  - 为特定主机指定的错误配置列表。此属性的子属性将合并到组合的服务器配置中。如果使用，则使用错误配置的请求处理程序处理的请求中可能缺少请求正文。默认值：无

## 位置配置属性

- **path** (_String_)
  - 为位置指定的路径。位置与 URL 解码的请求 URL 匹配。如果它相同，或者解码的请求 URL 在指定路径之上，则将使用位置配置。默认值：无

## 错误配置属性

- **scode** (_u16_; Ferron 1.3.0 及更高版本)
  - 为错误配置指定的状态代码。如果指定，则如果原始错误状态代码与指定的代码匹配，则将使用错误配置。如果未指定，则错误配置将应用于所有原始错误状态代码。默认值：无

## 全局、主机、错误和位置配置属性

- **domain** (_String_)
  - 主机的域名。此设置指定与主机关联的域名。默认值：无
- **ip** (_String_)
  - 主机的 IP 地址。此设置指定与主机关联的 IP 地址。默认值：无
- **serverAdministratorEmail** (_String_)
  - 服务器管理员的电子邮件地址。此设置指定服务器管理员的电子邮件地址，可用于联系目的。默认值：无
- **customHeaders** (_Object_)
  - 自定义 HTTP 标头。此对象包含服务器将在其响应中包含的自定义 HTTP 标头。您还可以在自定义 HTTP 标头值中使用 `{path}` 占位符，该占位符将被请求 URL 替换。默认值：无
- **disableToHTTPSRedirect** (_bool_)
  - 禁用从 HTTP 服务器到 HTTPS 服务器的重定向的选项。设置为 `true` 时，服务器不会自动将 HTTP 请求重定向到 HTTPS。默认值：`false`
- **wwwredirect** (_bool_)
  - 启用重定向到以“www.”开头的域名的选项。设置为 `true` 时，服务器将自动将请求重定向到“www”子域。默认值：`false`
- **enableIPSpoofing** (_bool_)
  - 启用通过 X-Forwarded-For 标头识别客户端原始 IP 地址的选项。设置为 `true` 时，服务器将使用 X-Forwarded-For 标头来识别客户端的 IP 地址。默认值：`false`
- **allowDoubleSlashes** (_bool_)
  - 在 URL 清理程序中允许双斜杠的选项。设置为 `true` 时，服务器将允许 URL 中出现双斜杠，这对于某些类型的 URL 重写可能很有用。默认值：`false`
- **rewriteMap** (_Array&lt;Object&gt;_)
  - URL 重写映射。此设置指定服务器将应用于传入请求的 URL 重写规则数组。默认值：无
  - **子属性**:
    - **regex** (_String_)
      - URL 重写的正则表达式。此设置指定服务器将用于匹配要重写的 URL 的正则表达式模式。默认值：无
    - **replacement** (_String_)
      - URL 重写的替换字符串。此设置指定服务器将用于重写匹配的 URL 的替换字符串。默认值：无
    - **isNotFile** (_bool_)
      - 仅当路径不是文件时才应用规则的选项。设置为 `true` 时，服务器仅在路径不对应于文件时才应用重写规则。默认值：`false`
    - **isNotDirectory** (_bool_)
      - 仅当路径不是目录时才应用规则的选项。设置为 `true` 时，服务器仅在路径不对应于目录时才应用重写规则。默认值：`false`
    - **allowDoubleSlashes** (_bool_)
      - 在重写的 URL 中允许双斜杠的选项。设置为 `true` 时，服务器将允许在重写的 URL 中出现双斜杠。默认值：`false`
    - **last** (_bool_)
      - 在此规则之后停止处理更多规则的选项。设置为 `true` 时，服务器将在此规则之后停止处理更多重写规则。默认值：`false`
- **enableRewriteLogging** (_bool_)
  - 启用 URL 重写日志记录的选项。设置为 `true` 时，服务器将记录其执行的所有 URL 重写。默认值：`false`
- **wwwroot** (_String_)
  - 将从中提供静态文件的 Web 根目录。此设置指定服务器将从中提供静态文件的根目录。默认值：无
- **disableTrailingSlashRedirects** (_bool_)
  - 如果路径指向目录，则禁用重定向的选项。设置为 `true` 时，如果路径指向目录，服务器将不会自动将请求重定向到带有尾部斜杠的路径。默认值：`false`
- **users** (_Array&lt;Object&gt;_)
  - 用户列表。此设置指定服务器将用于身份验证的用户对象数组。建议使用 `ferron-passwd` 工具生成用户对象。默认值：无
  - **子属性**:
    - **name** (_String_)
      - 用户名。此设置指定用户的用户名。默认值：无
    - **pass** (_String_)
      - 密码哈希。此设置指定用户的哈希密码。默认值：无
- **nonStandardCodes** (_Array&lt;Object&gt;_)
  - 非标准状态代码。此设置指定服务器将用于特定响应的非标准 HTTP 状态代码数组。默认值：无
  - **子属性**:
    - **scode** (_u16_)
      - 状态代码。此设置指定非标准 HTTP 状态代码。如果设置为 `401`，则启用 HTTP 基本身份验证（用户在 `users` 配置属性中指定）。默认值：无
    - **url** (_String_)
      - 要匹配或重定向到的 URL。此设置指定服务器将用于匹配请求的 URL 模式或服务器将重定向请求的 URL。默认值：无
    - **regex** (_String_)
      - 匹配 URL 的正则表达式。此设置指定服务器将用于匹配 URL 的正则表达式模式。默认值：无
    - **location** (_String_)
      - 重定向位置。此设置指定服务器将重定向请求的位置。默认值：无
    - **realm** (_String_)
      - 基本身份验证的领域。此设置指定服务器将用于基本身份验证的领域。默认值：无
    - **disableBruteProtection** (_bool_)
      - 禁用暴力破解保护的选项。设置为 `true` 时，服务器将禁用指定状态代码的暴力破解保护。默认值：`false`
    - **userList** (_Array&lt;String&gt;_)
      - 允许访问的用户列表。此设置指定允许访问与状态代码关联的资源的用户名数组。默认值：无
    - **users** (_Array&lt;String&gt;_)
      - 允许访问的 IP 地址列表。此设置指定允许访问与状态代码关联的资源的 IP 地址数组。默认值：无
- **errorPages** (_Array&lt;Object&gt;_)
  - 自定义错误页面。此设置指定服务器将用于特定 HTTP 状态代码的自定义错误页面数组。默认值：无
  - **子属性**:
    - **scode** (_u16_)
      - 状态代码。此设置指定将使用自定义错误页面的 HTTP 状态代码。默认值：无
    - **path** (_String_)
      - 错误页面的路径。此设置指定自定义错误页面的绝对文件路径。默认值：无
- **enableETag** (_bool_)
  - 启用 ETag 生成的选项。设置为 `true` 时，服务器将为响应生成 ETag 标头，可用于缓存目的。默认值：`true`
- **enableCompression** (_bool_)
  - 启用 HTTP 压缩的选项。设置为 `true` 时，服务器将使用 gzip 或其他压缩算法压缩响应以减少带宽使用。默认值：`true`
- **enableDirectoryListing** (_bool_)
  - 启用目录列表的选项。设置为 `true` 时，当请求目录时，服务器将生成并显示文件和目录列表。默认值：`false`
- **proxyTo** (_String_ or _Array&lt;String&gt;_; _rproxy_ 模块)
  - 反向代理将向其发送请求的基本 URL。支持 HTTP 和 HTTPS URL。也可以指定一个基本 URL 数组（请求将被随机分发）。默认值：无
- **secureProxyTo** (_String_ or _Array&lt;String&gt;_; _rproxy_ 模块)
  - 如果客户端通过 HTTPS 连接，反向代理将向其发送请求的基本 URL。支持 HTTP 和 HTTPS URL。也可以指定一个基本 URL 数组（请求将被随机分发）。默认值：无
- **cacheVaryHeaders** (_Array&lt;String&gt;_; _cache_ 模块)
  - 可在缓存中变化的请求标头列表。补充“Vary”响应标头。默认值：无
- **cacheIgnoreHeaders** (_Array&lt;String&gt;_; _cache_ 模块)
  - 不会存储在缓存中的响应标头列表。默认值：无
- **maximumCacheResponseSize** (_u64_ or `null`; _cache_ 模块)
  - 要缓存的最大响应大小（以字节为单位）。如果为 `null`，则理论上最大响应大小不受限制。默认值：`2097152`
- **cgiScriptExtensions** (_Array&lt;String&gt;_; _cgi_ 模块)
  - CGI 脚本扩展名，将通过 `cgi-bin` 目录外的 CGI 处理程序处理。默认值：无
- **cgiScriptInterpreters** (_Object_; _cgi_ 模块)
  - CGI 处理程序使用的 CGI 脚本解释器。对象键表示使用特定解释器的扩展名，而对象值可以是表示 CGI 脚本的第一个参数的 _Array&lt;String&gt;（第一个参数是解释器的路径），或为 `null` 以删除默认解释器。默认值：无，为 _.pl_, _.py_, _.sh_, _.ksh_, _.csh_, _.rb_ 和 _.php_ 扩展名以及 Windows 的 _.exe_, _.bat_ 和 _.vbs_ 扩展名设置了默认解释器。
- **scgiTo** (_String_; _scgi_ 模块)
  - SCGI 客户端将向其发送请求的基本 URL。支持 TCP（例如 `"tcp://localhost:4000/"`）和 Unix 套接字 URL（仅在 Unix 系统上；例如 `"unix:///run/scgi.sock"`）。默认值：`"tcp://localhost:4000/"`
- **scgiPath** (_String_; _scgi_ 模块)
  - 如果请求 URL 以此开头，则 SCGI 客户端将处理该请求的基本 URL。如果未指定，SCGI 客户端将处于非活动状态。默认值：无
- **fcgiScriptExtensions** (_Array&lt;String&gt;_; _fcgi_ 模块)
  - FastCGI 脚本扩展名，将通过指定的 FastCGI 路径之外的 FastCGI 处理程序处理。默认值：无
- **fcgiTo** (_String_; _fcgi_ 模块)
  - FastCGI 客户端将向其发送请求的基本 URL。支持 TCP（例如 `"tcp://localhost:4000/"`）和 Unix 套接字 URL（仅在 Unix 系统上；例如 `"unix:///run/fcgi.sock"`）。默认值：`"tcp://localhost:4000/"`
- **fcgiPath** (_String_; _fcgi_ 模块)
  - 如果请求 URL 以此开头，则 FastCGI 客户端将处理该请求的基本 URL。如果未指定，SCGI 客户端将处于非活动状态。默认值：无
- **authTo** (_String_; _fauth_ 模块)
  - Web 服务器将向其发送请求以进行转发身份验证的基本 URL。支持 HTTP 和 HTTPS URL。默认值：无
- **forwardedAuthCopyHeaders** (_Array&lt;String&gt;_; _fauth_ 模块)
  - 将从转发的身份验证服务器响应复制到原始请求的响应标头列表。默认值：无
- **enableLoadBalancerHealthCheck** (_bool_; _rproxy_ 模块)
  - 启用负载均衡器被动健康检查的选项。如果负载均衡器的连接失败，负载均衡器会记录失败的连接，如果失败连接过多，负载均衡器会暂时将后端服务器标记为“坏”并且不会选择该后端服务器。默认值：`false`
- **loadBalancerHealthCheckMaximumFails** (_u32_; _rproxy_ 模块)
  - 在后端服务器被标记为“坏”且不被负载均衡器选择之前，报告的最大失败连接数。默认值：`3`
- **disableProxyCertificateVerification** (_bool_; _rproxy_ 模块)
  - 禁用反向代理的后端服务器证书验证的选项。不建议在生产环境中使用此设置为 `true`。默认值：`false`
- **wsgiApplicationPath** (_String_; _wsgi_ 模块; Ferron 1.1.0 及更高版本)
  - 包含 WSGI 应用程序的文件的路径。WSGI 应用程序必须具有 `application` WSGI 回调。默认值：无
- **wsgiPath** (_String_; _wsgi_ 模块; Ferron 1.1.0 及更高版本)
  - 如果请求 URL 以此开头，则 WSGI 处理程序将处理该请求的基本 URL。默认值：`"/"`
- **wsgidApplicationPath** (_String_; _wsgid_ 模块; Ferron 1.1.0 及更高版本)
  - 包含 WSGI 应用程序的文件的路径。WSGI 应用程序必须具有 `application` WSGI 回调。默认值：无
- **wsgidPath** (_String_; _wsgid_ 模块; Ferron 1.1.0 及更高版本)
  - 如果请求 URL 以此开头，则 WSGI 处理程序（带有预分叉进程池）将处理该请求的基本 URL。默认值：`"/"`
- **asgiApplicationPath** (_String_; _asgi_ 模块; Ferron 1.1.0 及更高版本)
  - 包含 ASGI 应用程序的文件的路径。ASGI 应用程序必须具有 `application` ASGI 回调。默认值：无
- **asgiPath** (_String_; _asgi_ 模块; Ferron 1.1.0 及更高版本)
  - 如果请求 URL 以此开头，则 ASGI 处理程序将处理该请求的基本 URL。默认值：`"/"`
- **proxyInterceptErrors** (_bool_; _rproxy_ 模块; Ferron 1.3.0 及更高版本)
  - 启用后端错误响应拦截的选项。如果设置为 `true`，服务器将使用与后端错误响应相同的状态代码调用默认错误处理程序。默认值：`false`
- **disableProxyXForwarded** (_bool_; _rproxy_ 模块; Ferron 1.3.6 及更高版本)
  - 禁用后端服务器的 X-Forwarded-\* 标头的选项。默认值：`false`

## 服务器配置包含

服务器配置包含可以在服务器配置的 `include` 部分中指定（与 `global` 和 `host` 处于同一级别）。`include` 部分是与要合并的配置文件名匹配的 glob 模式列表。

## 示例配置

以下是 Ferron Web 服务器的示例配置：

```yaml
global:
  port: 8080
  sport: 8443
  secure: true
  enableHTTP2: true
  http2Settings:
    initialWindowSize: 65536
    maxFrameSize: 16384
    maxConcurrentStreams: 100
    maxHeaderListSize: 8192
    enableConnectProtocol: true
  logFilePath: "/var/log/ferron/access.log"
  errorLogFilePath: "/var/log/ferron/error.log"
  cert: "/etc/ssl/certs/fallback.crt"
  key: "/etc/ssl/private/fallback.key"
  tlsMinVersion: "TLSv1.2"
  tlsMaxVersion: "TLSv1.3"
  disableNonEncryptedServer: false
  enableOCSPStapling: true
  blocklist:
    - "192.168.1.100"
    - "10.0.0.5"
  enableCompression: true
  enableDirectoryListing: false
  environmentVariables:
    APP_MODE: "production"
    MAX_THREADS: "16"
  loadModules:
    - "rproxy"
  sni:
    "example.com":
      cert: "/etc/ssl/certs/example-com.crt"
      key: "/etc/ssl/private/example-com.key"
    "*.example.com":
      cert: "/etc/ssl/certs/example-com.crt"
      key: "/etc/ssl/private/example-com.key"

hosts:
  - domain: "example.com"
    serverAdministratorEmail: "admin@example.com"
    customHeaders:
      X-Frame-Options: "DENY"
      X-Content-Type-Options: "nosniff"
    rewriteMap:
      - regex: "^/old-path/(.*)"
        replacement: "/new-path/$1"
        isNotFile: true
        isNotDirectory: true
        last: true
    wwwroot: "/var/www/example"
    errorPages:
      - scode: 404
        path: "/var/www/example/errors/404.html"
      - scode: 500
        path: "/var/www/example/errors/500.html"
    locations:
      - path: "/static"
        wwwroot: "/var/www/static"
        rewriteMap:
          - regex: "^/static(?:[/?#](.*))?"
            replacement: "/$1"
            last: true
  - domain: "api.example.com"
    serverAdministratorEmail: "api-admin@example.com"
    disableToHTTPSRedirect: false
    allowDoubleSlashes: false
    enableETag: true
    users:
      - user: "admin"
        pass: "$2b$10$hashedpassword12345"
    nonStandardCodes:
      - scode: 401
        url: "/restricted.html"
    proxyTo: "http://backend-service:5000"
    # 取消注释以启用错误配置
    # errorConfig:
    #   - scode: 404
    #     proxyTo: "http://backend-fallback:5000"
# # 取消注释以启用配置文件包含
#include:
#  - /etc/ferron.d/**/*.yaml
```
