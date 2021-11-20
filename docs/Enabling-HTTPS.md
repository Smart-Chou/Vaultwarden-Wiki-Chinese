为了正确操作 vaultwarden，现在非常需要启用 [HTTPS](https://en.wikipedia.org/wiki/HTTPS)，因为 Bitwarden 网络保险库使用 [网络加密 API](https://developer.mozilla .org/en-US/docs/Web/API/SubtleCrypto），大多数浏览器仅在 HTTPS 上下文中可用。

有几种方法可以启用 HTTPS：

-（推荐）将 vaultwarden 置于代表 vaultwarden 处理 HTTPS 连接的 [反向代理](https://en.wikipedia.org/wiki/Reverse_proxy) 之后。
-（不推荐）启用 vaultwarden 内置的 HTTPS 功能（通过 [Rocket](https://rocket.rs/) Web 框架）。 Rocket 的 HTTPS 实现相对不成熟和有限。此方法也不支持 [[WebSocket 通知|启用-WebSocket-通知]]。

有关这些选项的更多详细信息，请参阅 [启用 HTTPS](#enabling-https) 部分。

要使 HTTPS 服务器正常工作，它还需要 SSL/TLS 证书，因此您需要决定如何获取它。同样，有几个选项：

-（推荐）使用 [ACME 客户端](https://letsencrypt.org/docs/client-options/) 获取 [Let's Encrypt](https://letsencrypt.org/) 证书。一些反向代理（例如，[Caddy](https://caddyserver.com/)）也具有使用 ACME 协议获取证书的内置支持。
-（推荐）如果您信任 [Cloudflare](https://www.cloudflare.com/) 代理您的流量，您可以让他们处理您的 SSL/TLS 证书的发布。请注意，上游 Bitwarden 网络保险库 (https://vault.bitwarden.com/) 在 Cloudflare 后面运行。
-（不推荐）[[设置私有 CA|Private-CA-and-self-signed-certs-that-work-with-Chrome]] 并颁发您自己的（自签名）证书。有各种与此相关的陷阱和不便，因此请注意警告。

有关这些选项的更多详细信息，请参阅 [获取 SSL/TLS 证书](#getting-ssltls-certificates) 部分。要使移动应用程序正常工作，您必须正确配置 [OCSP 装订](https://en.wikipedia.org/wiki/OCSP_stapling) 设置。

##启用HTTPS

### 通过反向代理

常用的反向代理有不少；一些示例配置可以在 [[Proxy examples|Proxy-examples]] 中找到。如果您不熟悉反向代理并且没有偏好，请首先考虑 [Caddy](https://caddyserver.com/)，因为它内置了对获取 Let's Encrypt 证书的支持。 [[Using Docker Compose|Using-Docker-Compose]] 文章有一个使用 Caddy 的很好的例子。

###通过火箭

：警告：不建议使用此方法。

要在 `vaultwarden` 本身中启用 HTTPS，请设置 `ROCKET_TLS` 环境变量，其格式如下：
```
ROCKET_TLS={certs="/path/to/certs.pem",key="/path/to/key.pem"}
```
在哪里：
- `certs` 是 PEM 格式的 SSL/TLS 证书链的路径。
- `key` 是 SSL/TLS 证书对应的私钥文件的路径（PEM 格式）。

笔记：
- `ROCKET_TLS` 行中使用的文件名 _extensions_ 不必像示例中那样必须是 `.pem`，有些地方可能会使用其他扩展名颁发证书，如证书的 `.crt` 和 `.key` 的证书私钥。文件 _format_ 必须是 PEM，即 base64 编码。由于 PEM 格式是 openssl 的默认格式，因此您可以简单地将 .cert、.cer、.crt 和 .key 文件重命名为 .pem，反之亦然，或者 - 作为替代方案 - 在 `ROCKET_TLS` 中使用 .crt 或 .key 作为文件扩展名线。
- 使用 RSA 证书/密钥。 Rocket 当前无法处理 ECC 证书/密钥，并输出误导性错误消息，例如

  > `[ERROR] environment variable ROCKET_TLS={certs="/ssl/ecdsa.crt",key="/ssl/ecdsa.key"} could not be parsed`

  (环境变量本身的格式没有问题；这是 Rocket 无法解析的证书/密钥内容。)
- 如果在 Docker 下运行，请记住 vaultwarden 在容器内运行时将解析 `ROCKET_TLS` 值，因此请确保 `certs` 和 `key` 路径是它们在容器内的显示方式（这可能与路径不同在 Docker 主机系统上）。

```sh
docker run -d --name bitwarden \
  -e ROCKET_TLS='{certs="/ssl/certs.pem",key="/ssl/key.pem"}' \
  -v /ssl/keys/:/ssl/ \
  -v /vw-data/:/data/ \
  -p 443:80 \
  vaultwarden/server:latest
```

您需要挂载 ssl 文件（-v 参数），并且需要转发适当的端口（-p 参数），通常是用于 HTTPS 连接的端口 443。如果您选择的端口号不同于 443，例如 3456，请记住在连接到服务时明确提供该端口号，例如：`https://bitwarden.local:3456`。

：警告：确保您的证书文件包含完整的信任链。对于 certbot，这意味着使用 `fullchain.pem` 而不是 `cert.pem`。完整的链应该包括两个证书：叶证书（与 `cert.pem` 中的内容相同），然后是 R3 或 E1 [中间证书]（https://letsencrypt.org/certificates/#intermediate-certificates）。例如，默认情况下，Android 在其系统信任存储中不包含任何 Let's Encrypt 中间证书，因此如果您不提供完整链，Android 客户端可能无法连接。

用于获取证书的软件通常使用符号链接。如果是这种情况，docker 容器需要可以访问这两个位置。

示例：[certbot](https://certbot.eff.org/) 将在 `/etc/letsencrypt/live/mydomain/` 中创建一个文件夹，其中包含所需的 `fullchain.pem` 和 `privkey.pem` 文件

这些文件被符号链接到`../../archive/mydomain/privkey.pem`

所以要从 bitwarden 容器中使用：

```sh
docker run -d --name bitwarden \
  -e ROCKET_TLS='{certs="/ssl/live/mydomain/fullchain.pem",key="/ssl/live/mydomain/privkey.pem"}' \
  -v /etc/letsencrypt/:/ssl/ \
  -v /bw-data/:/data/ \
  -p 443:80 \
  vaultwarden/server:latest
```

### 检查证书是否有效

当您的 vaultwarden 服务器可供外界使用时，您可以使用 [Comodo SSL Checker](https://comodosslstore.com/ssltools/ssl-checker.php)、[Qualys' SSL Labs](https://www.ssllabs .com/ssltest/) 或 [Digicert SSL Certficate Checker](https://www.digicert.com/help/) 来检查您的 SSL 证书是否有效，包括链。如果没有链 Android 设备将无法连接。

您也可以使用[Qualys' SSL Labs](https://www.ssllabs.com/ssltest/analyze.html) 来检查，但该实验室不支持自定义端口。另外请记住选中“不要在板上显示结果”复选框，否则您的系统将在“最近看到”列表中可见。

如果您运行的本地服务器没有连接到公共互联网，您可以使用 `openssl` 命令、[testssl.sh](https://testssl.sh/) 或 [SSLScan](https:// github.com/rbsec/sslscan/) 来验证您的证书的有效性。

执行以下操作以验证证书是否与链一起安装。
将 vault.domain.com 更改为您自己的域名。
```bash
openssl s_client -showcerts -connect vault.domain.com:443 -servername vault.domain.com

# or with a different port
openssl s_client -showcerts -connect vault.domain.com:7070 -servername vault.domain.com
```
输出的开头应如下所示（使用 Let's Encrypt 证书时）：
```
CONNECTED(00000003)
depth=2 O = Digital Signature Trust Co., CN = DST Root CA X3
verify return:1
depth=1 C = US, O = Let's Encrypt, CN = R3
verify return:1
depth=0 CN = vault.domain.com
verify return:1
```

验证有 3 个不同的深度（注意它从 0 开始）。
在输出中，您应该会看到来自 Let's Encrypt 本身的 base64 编码证书。

#### 检查 OSCP 有效性

如果 OCSP Stapling 无法正常工作，则连接移动应用程序将失败并显示消息“链验证失败”。
[Digicert SSL Certficate Checker](https://www.digicert.com/help/) 的吊销检查部分包含“OCSP Staple: Good”，一旦 OCSP 装订设置正确。您的网络服务器必须能够连接到作为证书 X509v3 扩展的一部分的“权威信息访问”URL，以便 OCSP 装订工作。

## 获取 SSL/TLS 证书

### 通过让我们加密

[Let's Encrypt](https://letsencrypt.org/) 免费颁发 SSL/TLS 证书。

为此，您的 Vaultwarden 实例必须有一个 DNS 名称（即，您不能简单地使用 IP 地址）。如果您的保管库管理员可在公共 Internet 上访问，则 Let's Encrypt 更容易设置，但即使您的实例是私有的（即，只能在您的 LAN 上访问），仍然可以通过 [DNS 挑战]（运行- a-private-vaultwarden-instance-with-Let's-Encrypt-certs）。

如果您已经拥有或控制一个域，那么只需为您的 Vaultwarden 实例的 IP 地址添加一个 DNS 名称。如果没有，您可以购买一个域名，尝试在 [Freenom](https://www.freenom.com/) 上免费获得一个，或者使用像 [Duck DNS](https:// www.duckdns.org/) 以获取现有域下的名称（例如，`my-bitwarden.duckdns.org`）。

为您的实例指定 DNS 名称后，请使用 [ACME 客户端](https://letsencrypt.org/docs/client-options/) 获取您的 DNS 名称的证书。 [Certbot](https://certbot.eff.org/) 和 [acme.sh](https://github.com/acmesh-official/acme.sh) 是两个最受欢迎的独立客户端。一些反向代理，如 [Caddy](https://caddyserver.com/) 也有内置的 ACME 客户端。

### 通过 Cloudflare

[Cloudflare](https://www.cloudflare.com/) 为个人提供免费服务。如果您信任他们代理您的流量并充当您的 DNS 提供商，您也可以让他们处理您的 SSL/TLS 证书的颁发。

注册域并为 vaultwarden 实例添加 DNS 记录后，登录 Cloudflare 仪表板并选择“SSL/TLS”，然后选择“源服务器”。生成原始证书（您可以选择有效期最长为 15 年）并配置 Vaultwarden 以使用它。如果您选择了 15 年有效期，则在可预见的未来您将无需更新此原产地证书。

请注意，原始证书仅用于保护 Cloudflare 和 Vaultwarden 之间的通信。 Cloudflare 将自动处理用于客户端和 Cloudflare 之间通信的证书的颁发和更新。

此外，如果您使用的是 vaultwarden 内置的 Rocket HTTPS 服务器，请确保选择“RSA”作为原始证书的私钥类型，因为 Rocket 目前不支持 ECC/ECDSA 证书。