---
title: 服务器模块
---

您可以使用 Rust 编写的模块来扩展 Ferron。

以下模块内置于 Ferron 中并默认启用：

- _cache_ - 此模块启用服务器响应缓存。
- _cgi_ - 此模块启用 CGI 程序的执行。
- _dcompress_ (Ferron 2.1.0 及更高版本) - 此模块为动态内容启用 HTTP 压缩。
- _fauth_ - 此模块启用转发到身份验证服务器的身份验证。
- _fcgi_ - 此模块启用连接到 FastCGI 服务器的支持。
- _fproxy_ - 此模块启用正向代理功能。
- _limit_ (Ferron 2.0.0 及更高版本) - 此模块启用速率限制。
- _replace_ (Ferron 2.0.0 及更高版本) - 此模块启用响应正文中字符串的替换。
- _rproxy_ - 此模块启用反向代理功能。
- _scgi_ - 此模块启用连接到 SCGI 服务器的支持。
- _static_ (Ferron 2.0.0 及更高版本) - 此模块启用静态文件服务。

Ferron 还支持可在编译时启用的其他模块。

Ferron 提供的其他模块来自以下存储库：

- [ferron-modules-python](https://github.com/ferronweb/ferron-modules-python.git) - 提供利用 Python 的网关接口 (ASGI, WSGI)。
- [ferron-module-example](https://github.com/ferronweb/ferron-module-example.git) - 对于“/hello”请求路径，响应“Hello World!”。

如果您想将 Ferron 与其他模块一起使用，可以查看[编译说明](https://github.com/ferronweb/ferron/blob/2.x/COMPILATION.md)。

## 模块说明

### _cache_ 模块

_cache_ 模块是 Ferron 的一个简单的内存缓存模块，可与“Cache-Control”和“Vary”标头一起使用。缓存在所有线程之间共享。

### _cgi_ 模块

要使用此模块运行 PHP 脚本，您可能需要调整 PHP 配置文件，该文件通常位于 `/etc/php/<php version>/cgi/php.ini`，方法是将 `cgi.force_redirect` 属性设置为 0。如果您不进行此更改，PHP-CGI 将显示一条警告，指示 PHP-CGI 二进制文件是使用启用的 `force-cgi-redirect` 编译的。建议对用户上传和下载使用 _cgi-bin_ 之外的目录，以防止 _cgi_ 模块错误地将带有 shebang 和 ELF 二进制文件的上传脚本视为 CGI 应用程序，这可能导致恶意软件感染、远程代码执行漏洞或 500 内部服务器错误等问题。

### _fauth_ 模块

此模块的灵感来自 [Traefik 的 ForwardAuth 中间件](https://doc.traefik.io/traefik/middlewares/http/forwardauth/)。如果身份验证服务器以 2xx 状态代码回复，则允许访问，并执行初始请求。否则，将发回身份验证服务器的响应。

向身份验证服务器提供以下请求标头：

- **X-Forwarded-Method** - 原始请求使用的 HTTP 方法
- **X-Forwarded-Proto** - 如果原始请求已加密，则为 `"https"`，否则为 `"http"`。
- **X-Forwarded-Host** - 原始请求中 _Host_ 标头的值
- **X-Forwarded-Uri** - 原始请求 URI
- **X-Forwarded-For** - 客户端的 IP 地址

### _fcgi_ 模块

PHP-FPM 可能在与 Ferron 不同的用户下运行，因此您可能需要为 PHP-FPM 用户设置权限。

如果您仅将 PHP-FPM 用于 Ferron，则可以在 PHP-FPM 池配置文件（例如 `/etc/php/8.2/fpm/pool.d/www.conf`）中将 `listen.owner` 和 `listen.group` 属性设置为 Ferron 用户。

### _fproxy_ 模块

如果您正在使用 _fproxy_ 模块，则本地网络和本地主机上的主机也可以从代理访问。如果您不希望这些主机可以从代理访问，可以使用防火墙阻止它们。

### _limit_ 模块

此模块使用令牌桶算法。速率限制是基于每个 IP 地址的。

### _replace_ 模块

如果您将此模块与静态文件服务一起使用，建议使用 `compressed #false` 禁用静态文件压缩，否则替换将无法正常工作。

### _rproxy_ 模块

当指定 _proxyTo_ 或 _secureProxyTo_ 配置属性时，将启用反向代理功能。

向后端服务器提供以下请求标头：

- **X-Forwarded-Proto** - 如果原始请求已加密，则为 `"https"`，否则为 `"http"`。
- **X-Forwarded-Host** - 原始请求中 _Host_ 标头的值
- **X-Forwarded-For** - 客户端的 IP 地址
