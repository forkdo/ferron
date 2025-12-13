---
title: Web 应用程序
---

Ferron 与各种 Web 应用程序兼容，例如使用 WordPress、Joomla、Laravel 等构建的应用程序。

## WordPress

WordPress 是一个非常流行的开源内容管理系统 (CMS)，它允许网站所有者以网站和博客的形式主要管理 Web 内容。

为了让 WordPress 在 Ferron 中支持 URL 重写，您可以安装并激活 [Ferron URL 重写支持插件](https://github.com/ferronweb/ferron-rewrite-support)。

您可以将以下配置用于基于 WordPress 构建的网站：

```kdl
# 使用 WordPress 的示例配置。将“example.com”替换为您的域名。
example.com {
    root "/var/www/wordpress" # 替换为 WordPress 安装目录的路径

    # 拒绝访问某些文件和目录
    status 403 regex="/\\."
    status 403 regex="^/(?:uploads|files)/.*\\.php(?:$|[#?])"

    # 漂亮链接
    rewrite "^/(.*)" "/index.php/$1" file=#false directory=#false last=#true

    fcgi_php "unix:///run/php/php8.4-fpm.sock" # 替换为 PHP-FPM 套接字的路径
    # 此外，如果将 Unix 套接字与 PHP-FPM 一起使用，
    # 请将 PHP 池配置中的侦听器所有者和组设置为 Web 服务器用户（如果您使用 GNU/Linux 安装程序，则为 `ferron`）
    # 例如：
    #   listen.owner = ferron
    #   listen.group = ferron
}
```

## Joomla

Joomla 是一个开源内容管理系统 (CMS)，以其可扩展性和灵活性而闻名，使其适用于从简单博客到复杂电子商务平台和公司网站的各种网站。

您可以将以下配置（不带缓存）用于基于 Joomla 构建的网站：

```kdl
# 使用 Joomla 的示例配置。将“example.com”替换为您的域名。
example.com {
    root "/var/www/joomla" # 替换为 Joomla 安装目录的路径

    # 拒绝访问某些目录和文件
    status 403 regex="^/(?:images|cache|media|logs|tmp)/.*\\.(?:php|pl|py|jsp|asp|sh|cgi)(?:$|[#?])"

    # 漂亮链接
    rewrite "^/api(?:/(.*))?" "/api/index.php/$1" file=#false directory=#false last=#true
    rewrite "^/(.*)" "/index.php/$1" file=#false directory=#false last=#true

    fcgi_php "unix:///run/php/php8.4-fpm.sock" # 替换为 PHP-FPM 套接字的路径
    # 此外，如果将 Unix 套接字与 PHP-FPM 一起使用，
    # 请将 PHP 池配置中的侦听器所有者和组设置为 Web 服务器用户（如果您使用 GNU/Linux 安装程序，则为 `ferron`）
    # 例如：
    #   listen.owner = ferron
    #   listen.group = ferron
}
```

如果您在 Ferron 中启用 HTTP 缓存（使用 `cache` 指令），您可以安装 [Joomla 的服务器缓存](https://www.web-expert.gr/en/joomla-extensions/item/127-nginx-server-cache-joomla) 扩展。此扩展设置 `Cache-Control` 标头，然后可供 `cache` 模块使用。

## Laravel

Laravel 是一个免费的开源 PHP Web 框架，用于开发 Web 应用程序。

您可以将以下配置用于使用 Laravel 构建的网站：

```kdl
# 使用 Laravel 的示例配置。将“example.com”替换为您的域名。
example.com {
    root "/var/www/laravel/public" # 替换为 Laravel 应用程序“public”目录的路径

    # 漂亮链接
    rewrite "^/(.*)" "/index.php/$1" file=#false directory=#false last=#true

    fcgi_php "unix:///run/php/php8.4-fpm.sock" # 替换为具有 PHP FastCGI 守护程序套接字实际路径的 Unix 套接字 URL。
    # 此外，如果将 Unix 套接字与 PHP-FPM 一起使用，
    # 请将 PHP 池配置中的侦听器所有者和组设置为 Web 服务器用户（如果您使用 GNU/Linux 安装程序，则为 `ferron`）
    # 例如：
    #   listen.owner = ferron
    #   listen.group = ferron
}
```

## YaBB

YaBB 是一个用 Perl 编写的免费互联网论坛软件包。

您可以将以下配置用于由 YaBB 提供支持的论坛：

```kdl
# 使用 YaBB 的示例配置。将“example.com”替换为您的域名。
example.com {
    root "/var/www/yabb" # 替换为 Joomla 安装目录的路径（包括“cgi-bin”目录）
    cgi

    # 将索引重定向到 YaBB
    status 301 regex="^/(?:$|[?#])" location="/cgi-bin/yabb2/YaBB.pl"

    # 禁止访问某些目录和文件
    status 403 regex="^/cgi-bin/yabb2/(?:Convert|Backups|Templates|Members|Sources|Admin|Messages|Languages|Variables|Boards|Help|Modules)(?:$|[/?#])"
}
```
