---
title: PHP 托管
---

Ferron 支持使用 PHP-CGI、配置为 FastCGI 服务器的 PHP-CGI 或 PHP-FPM 运行 PHP 脚本。这使您可以使用 Ferron 托管使用基于 PHP 的 CMS（如 WordPress 或 Joomla）构建的网站。

要通过 CGI 使用 Ferron 配置 PHP，您可以使用此配置：

```kdl
// 通过 CGI 使用 PHP 的示例配置。将“example.com”替换为您的域名。
example.com {
    root "/var/www/html"
    cgi
    cgi_extension ".php"
}
```

要通过 FastCGI 使用 Ferron 配置 PHP，您可以使用此配置：

```kdl
// 通过 FastCGI 使用 PHP 的示例配置。将“example.com”替换为您的域名。
example.com {
    root "/var/www/html"
    fcgi_php "unix:///run/php/php8.4-fpm.sock" // 替换为具有 PHP FastCGI 守护程序套接字实际路径的 Unix 套接字 URL。
    // 此外，如果将 Unix 套接字与 PHP-FPM 一起使用，
    // 请将 PHP 池配置中的侦听器所有者和组设置为 Web 服务器用户（如果您使用 GNU/Linux 安装程序，则为 `ferron`）
    // 例如：
    //   listen.owner = ferron
    //   listen.group = ferron
}
```

为确保最佳的 Web 服务器性能和效率，建议使用 FastCGI 而不是 CGI，因为 FastCGI 使 PHP 进程持续运行，减少了为每个请求启动新进程的开销，从而提高了响应时间和资源利用率。
