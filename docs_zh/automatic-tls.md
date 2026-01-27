---
title: 自动 TLS
---

Ferron 支持通过 Let's Encrypt 实现自动 TLS，并支持 TLS-ALPN-01、HTTP-01（Ferron 1.1.0 及更新版本）以及 DNS-01（Ferron 2.0.0 及更新版本）ACME 挑战。证书的域名将从主机配置中提取（对于 TLS-ALPN-01 和 HTTP-01 ACME 挑战，通配符域名将被忽略）。

自动 TLS 功能用于自动获取 TLS 证书，无需手动导入 TLS 证书或使用外部工具（如 Certbot）来获取 TLS 证书。这使得获取 TLS 证书的过程更加便捷高效。

Ferron 同时支持 Let's Encrypt 的生产环境和测试环境目录。测试环境的 Let's Encrypt 目录可用于测试目的，以及验证服务器和自动 TLS 是否配置正确。

此外，Ferron 2.0.0 及更新版本支持在 Ferron 运行用户的 home 目录中使用默认的操作系统特定的 ACME 缓存目录（如果 home 目录可用），这使得自动 TLS 的配置更加简便。

以下是配置自动 TLS 并使用 Let's Encrypt 生产环境目录的 Ferron 配置示例：

```kdl
* {
    auto_tls
    auto_tls_contact "someone@example.com" // 将 "someone@example.com" 替换为实际的邮箱地址
    auto_tls_cache "/path/to/letsencrypt-cache" // 将 "/path/to/letsencrypt-cache" 替换为实际的缓存目录。可选属性，但建议设置
    auto_tls_letsencrypt_production
}

// 将 "example.com" 替换为您的网站域名
example.com {
    root "/var/www/html"
}
```

## 关于 Cloudflare 代理（及其他 HTTPS 代理）的说明

Ferron 默认使用 TLS-ALPN-01 ACME 挑战来实现自动 TLS，但如果您的网站位于终止 TLS 的代理之后，则此方式将无法工作，因为 TLS-ALPN-01 挑战是在 TLS 握手级别进行的。

您可以改用 HTTP-01 挑战，它在 HTTP 级别工作。您可以添加一个 `auto_tls_challenge "http-01"` 全局配置指令，例如：

```kdl
* {
    auto_tls
    auto_tls_contact "someone@example.com" // 将 "someone@example.com" 替换为实际的邮箱地址
    auto_tls_cache "/path/to/letsencrypt-cache" // 将 "/path/to/letsencrypt-cache" 替换为实际的缓存目录。可选属性，但建议设置
    auto_tls_letsencrypt_production

    // 使用 HTTP-01 挑战替代 TLS-ALPN-01，因为服务器位于 HTTPS 代理之后。
    auto_tls_challenge "http-01"
}

// 将 "example.com" 替换为您的网站域名
example.com {
    root "/var/www/html"
}
```

## DNS 提供商

Ferron 2.0.0 及更新版本支持 DNS-01 ACME 挑战来实现自动 TLS。DNS-01 ACME 挑战需要在 `auto_tls_challenge` 指令的 `provider` 属性中配置一个 DNS 提供商。

以下是配置自动 TLS 并使用 Let's Encrypt 生产环境目录以及假设的 `example` DNS 提供商的 Ferron 配置示例：

```kdl
* {
    auto_tls
    auto_tls_contact "someone@example.com" // 将 "someone@example.com" 替换为实际的邮箱地址
    auto_tls_cache "/path/to/letsencrypt-cache" // 将 "/path/to/letsencrypt-cache" 替换为实际的缓存目录。可选属性，但建议设置
    auto_tls_letsencrypt_production
    auto_tls_challenge "dns-01" provider="example" some_prop="value" // "some_prop" 属性用于配置 DNS 提供商
}

// 将 "example.com" 替换为您的网站域名
example.com {
    root "/var/www/html"
}
```

### Amazon Route 53 (`route53`)

此 DNS 提供商使用 [Amazon Route 53 API](https://docs.aws.amazon.com/Route53/latest/APIReference/Welcome.html) 来认证和授权与 ACME 相关的 DNS 记录。此提供商在 Ferron 2.0.0 中添加。

#### 指令规范示例

```kdl
auto_tls_challenge "dns-01" provider="route53" access_key_id="your_key_id" secret_access_key="your_secret_access_key" region="aws-region" hosted_zone_id="your_hosted_zone_id"
```

#### 附加属性

- `access_key_id` - AWS 访问密钥 ID（可选）
- `secret_access_key` - AWS 秘密访问密钥（可选）
- `region` - AWS 区域（可选）
- `profile_name` - AWS 配置文件名称（可选）
- `hosted_zone_id` - Amazon Route 53 托管区域 ID（可选）

### bunny.net (`bunny`)

此 DNS 提供商使用 [bunny.net API](https://docs.bunny.net/reference) 来认证和授权与 ACME 相关的 DNS 记录。此提供商在 Ferron 2.4.0 中添加。

#### 指令规范示例

```kdl
auto_tls_challenge "dns-01" provider="bunny" api_key="your_api_key"
```

#### 附加属性

- `api_key` - bunny.net API 密钥（必需）

### Cloudflare (`cloudflare`)

此 DNS 提供商使用 [Cloudflare API](https://developers.cloudflare.com/api/resources/dns/) 来认证和授权与 ACME 相关的 DNS 记录。此提供商在 Ferron 2.0.0 中添加。

#### 指令规范示例

```kdl
auto_tls_challenge "dns-01" provider="cloudflare" api_key="your_api_key" email="your_email@example.com"
```

#### 附加属性

- `api_key` - Cloudflare API 密钥（必需）
- `email` - Cloudflare 账户邮箱地址（可选）

### deSEC (`desec`)

此 DNS 提供商使用 [deSEC API](https://desec.readthedocs.io/en/latest/index.html) 来认证和授权与 ACME 相关的 DNS 记录。此提供商在 Ferron 2.0.0 中添加。

#### 指令规范示例

```kdl
auto_tls_challenge "dns-01" provider="desec" api_token="your_api_token"
```

#### 附加属性

- `api_token` - deSEC API 令牌（必需）

### DigitalOcean (`digitalocean`)

此 DNS 提供商使用 [DigitalOcean API](https://docs.digitalocean.com/reference/api/digitalocean/) 来认证和授权与 ACME 相关的 DNS 记录。此提供商在 Ferron 2.4.0 中添加。

#### 指令规范示例

```kdl
auto_tls_challenge "dns-01" provider="digitalocean" oauth_token="your_token"
```

#### 附加属性

- `token` - DigitalOcean OAuth 令牌（必需）

### OVH (`ovh`)

此 DNS 提供商使用 [OVH API](https://api.ovh.com/console/) 来认证和授权与 ACME 相关的 DNS 记录。此提供商在 Ferron 2.4.0 中添加。

#### 指令规范示例

```kdl
auto_tls_challenge "dns-01" provider="ovh" application_key="your_application_key" application_secret="your_application_secret" consumer_key="your_consumer_key" endpoint="ovh-eu"
```

#### 附加属性

- `application_key` - OVH 应用密钥（必需）
- `application_secret` - OVH 应用密钥（必需）
- `consumer_key` - OVH 消费者密钥（必需）
- `endpoint` - OVH 端点。支持的值包括 `ovh-eu`、`ovh-ca`、`kimsufi-eu`、`kimsufi-ca`、`soyoustart-eu` 和 `soyoustart-ca`（必需）

### Porkbun (`porkbun`)

此 DNS 提供商使用 [Porkbun API](https://porkbun.com/api/json/v3/documentation) 来认证和授权与 ACME 相关的 DNS 记录。此提供商在 Ferron 2.0.0 中添加。

#### 指令规范示例

```kdl
auto_tls_challenge "dns-01" provider="porkbun" api_key="your_api_key" secret_key="your_secret_key"
```

#### 附加属性

- `api_key` - Porkbun API 密钥（必需）
- `secret_key` - Porkbun 秘密 API 密钥（必需）

### RFC 2136 (`rfc2136`)

此 DNS 提供商使用 [RFC 2136 协议](https://tools.ietf.org/html/rfc2136) 来认证和授权与 ACME 相关的 DNS 记录。此提供商可用于支持 RFC 2136 的服务器，如 Bind9。此提供商在 Ferron 2.0.0 中添加。

#### 指令规范示例

```kdl
auto_tls_challenge "dns-01" provider="rfc2136" server="udp://127.0.0.1:53" key_name="dnskey" key_secret="your_key_secret" key_algorithm="hmac-sha256"
```

#### 附加属性

- `server` - DNS 服务器地址 URL，使用 "tcp" 或 "udp" 方案（必需）
- `key_name` - DNS 服务器密钥名称（必需）
- `key_secret` - DNS 服务器密钥密钥，使用 Base64 编码（必需）
- `key_algorithm` - DNS 服务器密钥算法。支持的值包括 `hmac-md5`、`gss`、`hmac-sha1`、`hmac-sha224`、`hmac-sha256`、`hmac-sha256-128`、`hmac-sha384`、`hmac-sha384-192`、`hmac-sha512` 和 `hmac-sha512-256`（必需）

## 其他 DNS 提供商

如果您想将 Ferron 与额外的 DNS 提供商配合使用，可以查看[编译说明](https://github.com/ferronweb/ferron/blob/2.x/COMPILATION.md)。