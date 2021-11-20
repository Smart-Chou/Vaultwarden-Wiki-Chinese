# 应用配置

下面的小节涵盖了与 Vaultwarden 本身相关的强化。

## 禁用注册和(可选)邀请

默认情况下，Vaultwarden 允许任何匿名用户在未经邀请的情况下在服务器上注册新帐户。如果您有权访问管理页面，这不是必需的，但这对您在服务器上的第一个用户很有用，建议您在管理面板中禁用它(如果启用了管理面板)或 [[使用环境变量| Disable-registration-of-new-users]] 以防止攻击者在您的 vaultwarden 服务器上创建帐户。

Vaultwarden 还允许注册用户邀请其他新用户在服务器上创建帐户并加入他们的组织。这不会造成直接风险(只要您信任您的用户)，但可以在管理面板或 [[使用环境变量|Disable-invitations]] 中禁用它。

## 禁用密码提示显示
vaultwarden 在登录页面上显示密码提示，以适应未配置 SMTP 的小型/本地部署，攻击者可能会滥用这些信息来促进对服务器上用户的密码猜测攻击。这可以通过取消选中`显示密码提示`选项或 [[使用环境变量|密码提示显示]] 在管理面板中禁用。

# HTTPS/TLS 配置

以下小节介绍了与 HTTPS / TLS 相关的强化。

## 严格的 SNI

[SNI](https://en.wikipedia.org/wiki/Server_Name_Indication) 是 Web 浏览器请求 HTTPS 服务器为特定站点(例如，`bitwarden.example.com`)提供 SSL/TLS 证书的方式。假设`bitwarden.example.com` 的IP 地址为`1.2.3.4`。理想情况下，您希望您的实例只能通过 https://bitwarden.example.com 而不是 https://1.2.3.4 访问。这是因为出于各种原因不断扫描 IP 地址，如果可以通过这种方式检测到您的 Vaultwarden 实例，它就会成为更明显的目标。例如，一个简单的 [Shodan 搜索](https://www.shodan.io/search?query=bitwarden) 会显示一些可通过 IP 地址访问的 Bitwarden 实例。

## 反向代理

通常，您应该避免通过 vaultwarden 的内置 [[Rocket TLS support|Enabling-HTTPS]] 启用 HTTPS，尤其是在您的实例可公开访问的情况下。 Rocket本身列出了以下[警告](https://rocket.rs/v0.4/guide/configuration/#configuring-tls)：

> Rocket 的内置 TLS 尚未准备好用于生产。它仅供开发使用。

例如，Rocket TLS 不支持 [strict SNI](#Strict-SNI) 或 ECC 证书(仅 RSA)。

有关一些示例反向代理配置，请参阅 [[代理示例|代理示例]]。

# Docker 配置

下面的小节涵盖了与 Docker 相关的强化。

## 以非root用户身份运行

默认情况下，Vaultwarden Docker 映像配置为以`root`用户身份运行容器进程。这允许 vaultwarden 读取/写入任何数据 [bind-mounted](https://docs.docker.com/storage/bind-mounts/) 到容器中而没有权限问题，即使该数据由另一个用户拥有(例如，您在 Docker 主机上的用户帐户)。
默认配置提供了安全性和可用性的良好平衡——在无特权的 Docker 容器中以 root 身份运行本身提供了合理的隔离级别，同时也使那些不一定精通如何管理的用户更容易设置Linux 上的所有权/权限。但是，作为一般策略，以所需的最低权限运行进程在安全方面会更好；对于用像 Rust 这样的内存安全语言编写的程序来说，这不是什么问题，但请注意，Vaultwarden 确实也使用了一些用 C 编写的库代码(SQLite、OpenSSL、MySQL、PostgreSQL 等)。

在 Docker 中以非 root 用户 (uid/gid 1000) 运行容器进程 (vaultwarden)：

    docker run -u 1000:1000 -e ROCKET_PORT=8080 -p <host-port>:8080 \
           [...other args...] \
           vaultwarden/server:latest

许多 Linux 发行版中的默认用户具有 uid/gid 1000(运行 `id` 命令来验证)，因此如果您希望能够轻松访问您的 Vaultwarden 数据而无需更改为其他用户，那么这是一个很好的使用价值，但是您可以根据需要调整 uid/gid。请注意，您很可能需要指定一个数字 uid/gid，因为 vaultwarden 容器不共享用户/组名称到 uid/gid 的相同映射(例如，比较 `/etc/passwd` 和 `/将容器中的 etc/group` 文件复制到 Docker 主机上的文件)。

 `ROCKET_PORT`默认为80，这是一个[特权端口](https://www.w3.org/Daemon/User/Installation/PrivilegedPorts.html)；当以非 root 用户身份运行时，它需要为 1024 或更高，否则当 vaultwarden 尝试绑定和侦听该端口上的连接时，您将收到权限被拒绝错误。

在`docker-compose`中做同样的事情：

    services:
      vaultwarden:
        image: vaultwarden/server:latest
        container_name: bitwarden
        user: 1000:1000
        environment:
          - ROCKET_PORT=8080

        ... 其他配置 ...

由于此处更改了`ROCKET_PORT`，请确保同时更新您的反向代理配置，以将 Vaultwarden 流量代理到端口 8080(或您选择的任何更高端口)而不是 80。

## 将数据挂载到容器中

通常，只有 vaultwarden 需要正常运行的数据才应该挂载到 vaultwarden 容器中(通常，这只是您的数据目录，也可能是一个包含 SSL/TLS 证书和私钥的目录)。例如，不要挂载你的整个主目录，`/var/run/docker.sock` 等，除非你有特定的原因并且知道你在做什么。

此外，如果您不希望 Vaultwarden 修改您正在安装的数据(例如，证书)，那么 [以只读方式安装](https://docs.docker.com/storage/bind-mounts/#use-a-read-only-bind-mount) 通过将 `:ro` 添加到卷规范(例如，`docker run -v /home/username/bitwarden-ssl:/ssl:ro`)。

＃ 各种各样的

## 暴力缓解

当不使用双因素身份验证时，(理论上)可以暴力破解用户密码，从而获得对他们帐户的访问权限。缓解这种情况的一种相对简单的方法是设置 fail2ban，它会在登录尝试失败太多次后阻止 ipadresses。但是：在多个反向代理(例如 cloudflare)后面使用它时应该小心。
参见：[[Fail2Ban 设置|Fail2Ban 设置]]

## 隐藏在子目录下

传统上，Bitwarden 实例驻留在子域的根目录(即`bitwarden.example.com`，而不是`bitwarden.example.com/some/path`)。上游 Bitwarden 服务器目前仅支持子域根，而 vaultwarden 增加了对 [[alternate base directory|Using-an-alternate-base-dir]] 的支持。对于某些用户来说，这很有用，因为他们只能访问一个子域并希望在不同目录下运行多个服务。在这种情况下，他们通常会选择一些明显的东西，比如`mysubdomain.example.com/bitwarden`。但是，您也可以使用它来提供额外的保护层，方法是将 vaultwarden 放在类似 `mysubdomain.example.com/bitwarden/<mysecretstring>` 的地方，其中 `<mysecretstring>` 有效地充当密码。有些人可能会争辩说，这是 [通过默默无闻实现安全](https://en.wikipedia.org/wiki/Security_through_obscurity)，但实际上是[深度防御](https://en.wikipedia.org/wiki/Defense_in_depth_(computing)) -- 子目录的保密性只是一个额外的安全层，并不打算成为安全的主要手段(这仍然是用户主密码的强度)。