---
title: PHP 托管
description: "使用 CGI 或 FastCGI（PHP-FPM 或 PHP-CGI）在 Ferron 上托管 PHP 站点，包含示例 KDL 配置与故障排查说明。"
---

Ferron 可以通过 CGI 或 FastCGI 运行 PHP 应用程序。对于大多数部署，推荐使用 FastCGI，因为 PHP 工作进程会在请求之间保持存活，从而减少进程启动开销并提高吞吐量。

## 通过 FastCGI 运行 PHP（推荐）

要通过 FastCGI（通常是 PHP-FPM）运行 PHP，请使用 `fcgi_php`：

```kdl
// 通过 FastCGI 使用 PHP 的示例配置。将“example.com”替换为您的域名。
example.com {
    root "/var/www/html" // 将 "/var/www/html" 替换为您的 PHP 应用目录
    fcgi_php "unix:///run/php/php8.4-fpm.sock" // 替换为您的 PHP FastCGI 套接字或 TCP URL

    // 如果使用 Unix 套接字通过 PHP-FPM 通信，请确保套接字可被 Ferron 访问。
    // 例如，在您的 PHP-FPM 池配置中：
    //   listen.owner = ferron
    //   listen.group = ferron
}
```

当您的 PHP FastCGI 服务器未通过 Unix 套接字暴露时，也可以将 `fcgi_php` 指向 TCP 监听器（例如 `tcp://127.0.0.1:9000/`）。

## 通过 CGI 运行 PHP

如果您特别想要经典的 CGI 执行方式，请启用 `cgi` 并映射 `.php` 扩展名：

```kdl
// 通过 CGI 使用 PHP 的示例配置。将“example.com”替换为您的域名。
example.com {
    root "/var/www/html" // 将 "/var/www/html" 替换为您的 PHP 应用目录
    cgi
    cgi_extension ".php"
}
```

CGI 可以正常工作，但对于生产负载通常比 FastCGI 慢，因为每次请求都会启动一个 PHP 进程。

## 注意事项与故障排查

- 如果 PHP 文件被下载而不是执行，请确认您已在正确的域名/位置块中启用了 `fcgi_php` 或 `cgi` + `cgi_extension ".php"`。
- 如果使用 CGI 模块配合 PHP-CGI，您可能需要在 CGI 的 `php.ini` 中设置 `cgi.force_redirect = 0`；否则请求可能因 force-cgi-redirect 警告而失败。
- 如果使用 `fcgi_php` 时遇到 `500 Internal Server Error`，请确认套接字或 TCP 端点存在且您的 PHP FastCGI 守护程序（PHP-FPM 或 FastCGI 模式的 PHP-CGI）正在运行。
- 如果使用 Unix 套接字，请确保 Ferron 可以访问套接字文件（PHP-FPM 池配置中的所有者/组/权限）。
- 使用 CGI 时，请将上传/下载目录放在 `cgi-bin` 之外，以避免上传文件被意外当作 CGI 执行。
- 有关“美观 URL”和前端控制器模式（常见于 PHP CMS/框架技术栈），请参阅 [URL 重写](/docs/use-cases/url-rewriting)。
- 有关框架/CMS 特定的重写与加固，请参阅 [Web 应用程序](/docs/use-cases/web-applications)。
- 有关指令详情（`cgi`、`cgi_extension`、`fcgi_php`），请参阅 [配置：应用后端](/docs/configuration/application-backends)。
