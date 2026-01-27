---
title: 服务模块
---

你可以通过 Rust 编写的模块来扩展 Ferron。

以下模块已内置在 Ferron 中，并默认启用：

- _cache_ - 此模块启用服务器响应缓存。
- _cgi_ - 此模块启用 CGI 程序的执行。
- _dcompress_ (Ferron 2.1.0 及更新版本) - 此模块启用动态内容的 HTTP 压缩。
- _fauth_ - 此模块启用转发至认证服务器的认证功能。
- _fcgi_ - 此模块启用连接 FastCGI 服务器的支持。
- _fproxy_ - 此模块启用正向代理功能。
- _fproxyauth_ (Ferron 2.4.0 及更新版本) - 此模块通过 HTTP Basic 认证启用正向代理认证。
- _limit_ (Ferron 2.0.0 及更新版本) - 此模块启用速率限制。
- _replace_ (Ferron 2.0.0 及更新版本) - 此模块启用在响应体中替换字符串。
- _rproxy_ - 此模块启用反向代理功能。
- _scgi_ - 此模块启用连接 SCGI 服务器的支持。
- _static_ (Ferron 2.0.0 及更新版本) - 此模块启用静态文件服务。

Ferron 还支持可在编译时启用的额外模块。

由 Ferron 提供的额外模块来自以下仓库：

- [ferron-modules-python](https://github.com/ferronweb/ferron-modules-python.git) - 提供利用 Python 的网关接口（ASGI、WSGI）。
- [ferron-module-example](https://github.com/ferronweb/ferron-module-example.git) - 对路径为 "/hello" 的请求返回 "Hello World!"。

如果你希望使用带有额外模块的 Ferron，可以查阅 [编译说明](https://github.com/ferronweb/ferron/blob/2.x/COMPILATION.md)。

## 模块说明

### _cache_ 模块

_cache_ 模块是 Ferron 的一个简单的内存缓存模块，它与 "Cache-Control" 和 "Vary" 头部协同工作。该缓存是跨所有线程共享的。

### _cgi_ 模块

若要使用此模块运行 PHP 脚本，你可能需要调整 PHP 配置文件（通常位于 `/etc/php/<php version>/cgi/php.ini`），将 `cgi.force_redirect` 属性设置为 0。如果不进行此更改，PHP-CGI 将显示一条警告，指出 PHP-CGI 二进制文件是使用 `force-cgi-redirect` 编译的。建议将用户上传和下载的目录设置在 _cgi-bin_ 之外，以防止 _cgi_ 模块将带有 shebang 或 ELF 二进制文件的上传脚本误认为 CGI 应用程序，从而避免引发恶意软件感染、远程代码执行漏洞或 500 Internal Server Error 等问题。

### _fauth_ 模块

该模块的灵感来源于 [Traefik 的 ForwardAuth 中间件](https://doc.traefik.io/traefik/middlewares/http/forwardauth/)。如果认证服务器返回 2xx 状态码，则允许访问并执行初始请求；否则，将认证服务器的响应返回。

传递给认证服务器的请求头包括：

- **X-Forwarded-Method** - 原始请求使用的 HTTP 方法
- **X-Forwarded-Proto** - 如果原始请求是加密的，则为 `"https"`，否则为 `"http"`。
- **X-Forwarded-Host** - 原始请求中 _Host_ 头的值
- **X-Forwarded-Uri** - 原始请求 URI
- **X-Forwarded-For** - 客户端的 IP 地址

### _fcgi_ 模块

PHP-FPM 可能以与 Ferron 不同的用户身份运行，因此你可能需要为 PHP-FPM 用户设置权限。

如果你仅将 PHP-FPM 用于 Ferron，可以在 PHP-FPM 池配置文件中（例如 `/etc/php/8.2/fpm/pool.d/www.conf`）将 `listen.owner` 和 `listen.group` 属性设置为 Ferron 用户。

### _fproxy_ 模块

如果你使用 _fproxy_ 模块，则本地网络中的主机和本地主机也可通过代理访问。如果你不希望这些主机通过代理访问，可以使用防火墙进行阻止。

### _limit_ 模块

该模块使用令牌桶（Token Bucket）算法。速率限制基于每个 IP 地址进行。

### _replace_ 模块

如果你在静态文件服务中使用此模块，建议通过 `compressed #false` 禁用静态文件压缩，否则替换功能将无法生效。

### _rproxy_ 模块

当配置属性 _proxyTo_ 或 _secureProxyTo_ 被指定时，反向代理功能即被启用。

传递给后端服务器的请求头包括：

- **X-Forwarded-Proto** - 如果原始请求是加密的，则为 `"https"`，否则为 `"http"`。
- **X-Forwarded-Host** - 原始请求中 _Host_ 头的值
- **X-Forwarded-For** - 客户端的 IP 地址