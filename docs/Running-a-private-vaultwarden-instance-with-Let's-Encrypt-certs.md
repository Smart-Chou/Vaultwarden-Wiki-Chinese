假设您想运行一个只能从您的本地网络访问的 vaultwarden 实例，但您希望您的实例启用 HTTPS，并使用由广泛接受的 CA 签署的证书，而不是管理您自己的 [私有 CA](Private-CA-and-self-signed-certs-that-work-with-Chrome)(以避免必须将私有 CA 证书加载到您所有的设备)。

本文演示了如何使用 [Caddy](https://caddyserver.com/) Web 服务器创建这样的设置，该服务器具有对各种 DNS 提供商的内置 ACME 支持。我们将配置 Caddy 以通过 ACME [DNS 挑战](https://letsencrypt.org/docs/challenge-types/#dns-01-challenge) 获取 Let's Encrypt 证书——使用更常见的 HTTP 挑战会出现问题在这里，因为它依赖于 Let's Encrypt 服务器能够访问您的内部 Web 服务器。

(请注意，本文以更通用的术语介绍了 DNS 挑战设置，但许多用户可能会发现使用 Docker Compose 来集成 Caddy 和 vaultwarden 最容易。请参阅 [使用 Docker Compose](Using-Docker-Compose#caddy-with-dns-challenge) 以获得特定于此的示例。)

涵盖了两个 DNS 提供商：

- [Duck DNS](https://www.duckdns.org/) -- 这会在`duckdns.org` 下为您提供一个子域(例如，`my-vw.duckdns.org`)。如果您还没有域，则此选项最简单。
- [Cloudflare](https://www.cloudflare.com/) -- 这使您可以将 vaultwarden 实例置于您拥有或控制的域下。请注意，Cloudflare 可以仅用作 DNS 提供商(即，没有 Cloudflare 最著名的代理功能)。如果您目前没有域名，您可以在 [Freenom](https://www.freenom.com/) 上免费获得一个。

使用 Web 服务器、[ACME 客户端](https://letsencrypt.org/docs/client-options/) 和 DNS 提供商的其他组合当然可以创建类似的设置，但您必须找出差异在细节。

## 获取自定义 Caddy 构建

默认情况下，Caddy 中没有内置 DNS 质询支持，因为大多数人不使用这种质询方法，并且它需要为每个 DNS 提供商自定义实现。

获取带有必要 DNS 挑战模块的 Caddy 版本的最简单方法是通过 https://caddyserver.com/download。选择您的平台，勾选`github.com/caddy-dns/cloudflare`(对于Cloudflare 支持)和/或`github.com/caddy-dns/duckdns`(对于Duck DNS 支持)，然后点击`Download `.

如果您更喜欢从源代码构建，可以使用 [`xcaddy`](https://caddyserver.com/docs/build#xcaddy)。例如，要创建一个包含 Cloudflare 和 Duck DNS 支持的构建：

    xcaddy build --with github.com/caddy-dns/cloudflare --with github.com/caddy-dns/duckdns

将 `caddy` 二进制文件移动到 `/usr/local/bin/caddy` 或路径中的其他适当目录。或者，运行 `sudo setcap cap_net_bind_service=+ep /usr/local/bin/caddy` 以允许 `caddy` 在不以 root 身份运行的情况下侦听特权端口(< 1024)。

## Duck DNS setup

如果您还没有帐户，请在 https://www.duckdns.org/ 创建一个。为您的 vaultwarden 实例创建一个子域(例如，`my-vw.duckdns.org`)，将其 IP 设置为您的 vaultwarden 主机的私有 IP(例如，`192.168.1.100`)。记下您帐户的令牌([UUID](https://en.wikipedia.org/wiki/UUID) 格式的字符串)。 Caddy 将需要此令牌来解决 DNS 挑战。

创建一个名为`Caddyfile`的文件，内容如下:

```
{$DOMAIN}:443 {
    tls {
        dns duckdns {$DUCKDNS_TOKEN}
    }
    reverse_proxy localhost:8080
    reverse_proxy /notifications/hub localhost:3012
}
```

创建一个名为 `caddy.env` 的文件，内容如下(根据需要替换每个值)：
```
DOMAIN=my-vw.duckdns.org
DUCKDNS_TOKEN=00112233-4455-6677-8899-aabbccddeeff
```

通过运行启动`caddy`
```
caddy run -envfile caddy.env
```

通过运行启动`Vaultwarden`
```
export ROCKET_PORT=8080
export WEBSOCKET_ENABLED=true

./vaultwarden
```

您现在应该可以通过 https://my-vw.duckdns.org 访问您的 vaultwarden 实例。

## Cloudflare 设置

如果您还没有帐户，请在 https://www.cloudflare.com/ 创建一个；您还必须前往您的域名注册商，将您的名称服务器设置为 Cloudflare 分配给您的名称服务器。为您的 vaultwarden 实例创建一个子域(例如，`vw.example.com`)，将其 IP 设置为您的 vaultwarden 主机的私有 IP(例如，`192.168.1.100`)。例如：

![A记录配置](https://i.imgur.com/BBvy4Yj.png)

为 DNS 挑战创建 API 令牌(有关更多背景信息，请参阅 https://github.com/libdns/cloudflare/blob/master/README.md)：

- 在右上角，单击人物图标并导航到`我的个人资料`，然后选择`API 令牌`选项卡。
- 点击`Create Token`按钮，然后点击`Edit zone DNS`上的`Use template`。
- 如果您喜欢更具描述性的名称，请编辑`令牌名称`字段。
- 在 `Permissions` 下，应该已经填充了 `Zone / DNS / Edit` 权限。添加另一个权限：`Zone / Zone / Read`。
- 在`区域资源`下，设置`包含/特定区域/example.com`(将`example.com`替换为您的域)。
- 在 `TTL` 下，设置一个结束日期，让您的代币变为非活动状态。您可能希望在未来选择一个。
- 创建令牌并复制令牌值。

您的令牌列表应如下所示：

![API 令牌配置](https://i.imgur.com/FoOv9Ww.png)

创建一个名为`Caddyfile`的文件，内容如下：
```
{$DOMAIN}:443 {
    tls {
        dns cloudflare {$CLOUDFLARE_API_TOKEN}
    }
    reverse_proxy localhost:8080
    reverse_proxy /notifications/hub localhost:3012
}
```
``

创建一个名为 `caddy.env` 的文件，内容如下(根据需要替换每个值)：
```
DOMAIN=vw.example.com
CLOUDFLARE_API_TOKEN=<your-api-token>
```

通过运行启动`caddy`
```
caddy run -envfile caddy.env
```

通过运行启动`Vaultwarden`
```
export ROCKET_PORT=8080
export WEBSOCKET_ENABLED=true

./vaultwarden
```

您现在应该可以通过 https://vw.example.com 访问您的 Vaultwarden 实例。

## 使用`lego` CLI 获取证书

在上面的 DuckDNS 示例中，Caddy 使用 `lego` 库通过 DNS 质询获取证书。
`lego` 也有一个 CLI，您可以使用它直接获取证书，例如如果您想使用 Caddy 以外的反向代理。
(注意：此示例使用`lego`，但还有其他独立的 ACME 客户端支持 DNS 质询方法(请参阅 [DNS 挑战](#dns-挑战) 部分。)

以下是如何执行此操作的示例：

1. 从 https://github.com/go-acme/lego/releases 为您的系统下载预构建的 `lego` 二进制文件。将内容解压到某个目录，例如`/usr/local/lego`。
2. 从那个目录，运行`DUCKDNS_TOKEN=<token> ./lego -a --dns daddns -d my-vw.duckdns.org -m me@example.com run`，
   为令牌、域和电子邮件地址替换适当的值。这会为您注册 Let's Encrypt 和
   为您的域获取证书。
3. 设置每周一次的 cron 作业来运行 `DUCKDNS_TOKEN=<token> ./lego --dns daddns -d my-vw.duckdns.org -m me@example.com refresh`。
   这会在您的证书即将到期时更新您的证书。

(注意：`lego` 默认请求 ECC/ECDSA 证书。如果您使用的是 vaultwarden 内置的 [开启HTTPS](Enabling-HTTPS#通过火箭)，您将需要请求 RSA 证书。在 `上面的lego`命令，添加选项`--key-type rsa2048`。)

在此示例中，您需要使用以下生成的输出来配置您的反向代理：

- `/usr/local/lego/.lego/certificates/my-vw.duckdns.org.crt`(证书)
- `/usr/local/lego/.lego/certificates/my-vw.duckdns.org.key`(私钥)

## 故障排除

### DNS 问题

如果您的子域出现 DNS 解析错误(例如，`DNS_PROBE_FINISHED_NXDOMAIN`或`ERR_NAME_NOT_RESOLVED`)，则您的 DNS 解析器可能会阻止解析，因为：

1. 出于安全原因，它会阻止动态 DNS 服务。
2. 它会阻止解析为私有 (RFC 1918) IP 地址的域，以防止 [DNS 重新绑定](https://en.wikipedia.org/wiki/DNS_rebinding) 攻击或其他原因。

无论哪种情况，您都可以尝试使用其他 DNS 解析器，例如 Google 的`8.8.8.8`或 Cloudflare 的`1.1.1.1`。在第二种情况下，如果您在 dnsmasq 或 Unbound 等本地 DNS 服务器后面运行，您可以将其配置为完全禁用 DNS 重新绑定保护，或允许某些域返回私有地址。

## 参考

### DNS 挑战

- https://community.letsencrypt.org/t/dns-providers-who-easily-integrate-with-lets-encrypt-dns-validation/86438
- https://caddy.community/t/how-to-use-dns-provider-modules-in-caddy-2/8148

### Caddy Cloudflare 模块

- https://github.com/caddy-dns/cloudflare
- https://go-acme.github.io/lego/dns/cloudflare/

### Caddy Duck DNS 模块

- https://github.com/caddy-dns/duckdns
- https://go-acme.github.io/lego/dns/duckdns/