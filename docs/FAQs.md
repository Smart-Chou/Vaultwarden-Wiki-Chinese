# FAQs

### Vaultwarden 是否与 Bitwarden 项目或 8bit Solutions LLC 相关联？

简短的回答，**不**。 有时两个项目的开发人员之间有一些联系，但没有合作。 除此之外，Vaultwarden 项目仅使用 8bit Solutions LLC 提供的 Web Vault 和一些补丁来使其与我们的实现一起工作。

### Vaultwarden 可以连接到 Oracle MySQL V8.x 数据库吗？

使用 Oracle MySQL v8.x 尝试启动 Vaultwarden 时，您可能会收到以下警告

```
[vaultwarden::util][WARN] Can't connect to database, retrying: DieselConError.
[CAUSE] BadConnection(
    "Authentication plugin \'caching_sha2_password\' cannot be loaded: /usr/lib/x86_64-linux-gnu/mariadb18/plugin/caching_sha2_password.so: cannot open shared object file: No such file or directory",
)
```

Oracle MySQL v8.x 默认使用更安全的密码散列方法，这很好，但目前我们的版本不支持。 您需要以特定方式创建 Vaultwarden 用户，以便它使用旧的本机密码散列。

```sql
-- 在 MySQLv8 安装上使用它
CREATE USER 'vaultwarden'@'localhost' IDENTIFIED WITH mysql_native_password BY 'yourpassword';
```

如果您已经创建了用户并且只想更改散列方法，请使用以下命令：

```sql
-- 将密码类型从 caching_sha2_password 更改为 native
ALTER USER 'vaultwarden'@'localhost' IDENTIFIED WITH mysql_native_password BY 'yourpassword';
```

另请参阅：[使用 MariaDB - 创建数据库和用户](https://github.com/dani-garcia/vaultwarden/wiki/Using-the-MariaDB-(MySQL)-Backend#create-database-and-user)

### 我的客户端(桌面、移动、Web)不工作，我无法登录或它抱怨证书无效。

Bitwarden 客户端需要安全连接才能正常工作而不会出现任何问题。尽管有些客户端可以在没有安全连接的情况下工作，但我们不建议这样做。 大多数情况下，当人们使用证书但仍有问题时，他们使用的是所谓的自签名证书。虽然这些可以提供安全连接，但某些平台不允许或不支持这一点。 我们建议使用 Lets Encrypt 之类的服务来提供有效且大多数设备默认接受的证书。 请参阅以下页面：

* [启用-HTTPS](https://github.com/dani-garcia/vaultwarden/wiki/Enabling-HTTPS)
* [使用 Let's Encrypt 证书运行私有 vaultwarden 实例](https://github.com/dani-garcia/vaultwarden/wiki/Running-a-private-vaultwarden-instance-with-Let's-Encrypt-certs)

### 为什么我看不到所有保管库项目的图标？

没有显示图标的原因有多种。 如果它只是用于一些保险库项目，则可能是我们无法提取它。某些站点启用了一些保护，这会导致我们的实施失败。他们中的大多数需要 Javascript 才能工作。

也可能是 Vaultwarden 服务器无法访问 Internet 或解析 DNS 查询。 您可以检查`/admin/diagnostics`页面，看看您是否可以解析 DNS 查询并连接到 Internet。 如果确实有效，则可能还有防火墙或传出 Internet 代理可能会阻止这些请求。

### Websocket 连接显示错误的 IP 地址

这不是我们可以解决的问题。我们使用的库不支持任何形式的`X-Forwarded-For`或`Forward`标头。 除非您在没有任何代理的情况下直接运行 vaultwarden 或运行透明代理，否则它会始终显示所使用的反向代理的 IP，这可能会导致它显示正确的 IP。这不是记录的重要部分，如果您使用反向代理，您可能还会在其日志中看到此请求，该请求将具有正确的 IP。

### 我可以将 Vaultwarden 作为 Azure WebApp 运行吗

不幸的是，Azure WebApp 使用 CIFS/Samba 作为不支持锁定的卷存储。这会导致 SQLite 数据库文件出现问题。 有两种方法可以解决这个问题。

1. 不要使用 SQLite，而是使用 MariaDB/MySQL 或 Posgresql 作为数据库后端。
2. 尝试使用 `ENABLE_DB_WAL` 环境变量禁用 WAL，方法是将其值设置为 `false`。这需要在一个新文件上完成，因此您需要删除之前创建的 `db.sqlite3` 文件并再次重新启动 Vaultwarden 应用程序。

### 我在常见问题解答中没有找到我的答案，下一步该怎么做？

好吧，请尝试搜索并点击我们精彩的 [Wiki](https://github.com/dani-garcia/vaultwarden/wiki)。 如果这对您没有帮助，请尝试查看 [Github 讨论](https://github.com/dani-garcia/vaultwarden/discussions) 或 [Vaultwarden 论坛](https://vaultwarden.discourse.group) .

如果这也导致没有解决方案，您可以尝试搜索开放和封闭的 [问题](https://github.com/dani-garcia/vaultwarden/issues)。

如果您仍然没有找到答案，您可以在 [Github 讨论](https://github.com/dani-garcia/vaultwarden/discussions) 或 [Vaultwarden 论坛](https://vaultwarden.discourse) 上开始一个主题.group/)，或加入我们的 [聊天室](https://matrix.to/#/#vaultwarden:matrix.org)。
