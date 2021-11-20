您可以将 Vaultwarden 配置为通过 SMTP 代理发送电子邮件：

```
docker run -d --name vaultwarden \
  -e SMTP_HOST=<smtp.domain.tld> \
  -e SMTP_FROM=<vaultwarden@domain.tld> \
  -e SMTP_PORT=587 \
  -e SMTP_SSL=true \
  -e SMTP_USERNAME=<username> \
  -e SMTP_PASSWORD=<password> \
  -v /vw-data/:/data/ \
  -p 80:80 \
  vaultwarden/server:latest
```

当 `SMTP_SSL` 设置为 `true`(这是默认值)时，只会接受 TLSv1.1 和 TLSv1.2 协议，并且 `SMTP_PORT` 将默认为 `587`。如果设置为`false`，`SMTP\_PORT`将默认为`25`，并且将尝试机会加密(在 2020 年 3 月 12 日之前不会尝试使用代码进行加密)。这可能非常不安全，仅当您知道自己在做什么时才使用此设置。要以显式模式运行 SMTP，请将 `SMTP_EXPLICIT_TLS` 设置为 `true`。如果您无需登录即可发送电子邮件，则可以简单地不设置`SMTP\_USERNAME`和`SMTP\_PASSWORD`。

请注意，如果启用了 SMTP 和邀请，邀请将通过电子邮件发送给新用户。您必须使用 Vaultwarden 实例的基本 URL 设置 `DOMAIN` 配置选项，才能正确生成邀请链接：

```
docker run -d --name vaultwarden \
...
-e DOMAIN=https://vault.example.com \
...
```

用户邀请链接的有效期为 5 天，之后需要发送新邀请。

### SMTP 服务器

正确配置 SMTP 服务器/中继并非易事。 Vaultwarden 使用的邮件程序库也不是最容易进行故障排除的。因此，除非您对自己进行设置特别感兴趣，否则使用外部服务可能更容易。

以下是一些具有免费层级的服务，允许每天发送 100-200 封电子邮件(对于大多数用例来说已经足够了)：

* [SendGrid](https://sendgrid.com)
* [MailJet](https://www.mailjet.com)

### 这里有一些众所周知的服务的合理默认值

### 一般的

邮件服务器侦听端口 25 主要只是为了接受来自其他邮件服务器的邮件，并且仅接收它们作为最终位置的邮件。
此外，许多互联网提供商会阻止传出端口 25 以防止垃圾邮件。
大多数邮件服务器需要登录才能使用端口 587 或端口 465。
端口 587 称为提交端口，大多数时候只能在使用用户名和密码时使用。在客户端和服务器之间的通信期间，端口 587 开始时未加密并升级为 TLS 加密连接。
端口 465 从一开始就是 SSL 加密的，并且根本没有通过该端口进行纯文本通信。

每个端口的一些常规设置。

- 用于使用端口 465 的邮件服务器

    ```ini
    SMTP_PORT=465
    SMTP_SSL=false
    SMTP_EXPLICIT_TLS=true
    ```

- 用于使用端口 587(有时为 25)的邮件服务器

    ```ini
    SMTP_PORT=587
    SMTP_SSL=true
    SMTP_EXPLICIT_TLS=false
    ```

- 用于根本不支持加密的邮件服务器。

    ```ini
    SMTP_PORT=25
    SMTP_SSL=false
    SMTP_EXPLICIT_TLS=false
    ```

#### Google/Gmail

您需要为 Vaultwarden 生成应用密码才能使用 Gmail。
请按照此处的步骤操作：https://support.google.com/accounts/answer/185833?hl=zh-CN&ref_topic=7189145
最后你会看到一个密码(中间没有空格，只是为了方便输入)，我们这个密码。
全SSL：

```ini
  # 域：gmail.com、googlemail.com
  SMTP_HOST=smtp.gmail.com
  SMTP_PORT=465
  SMTP_SSL=false
  SMTP_EXPLICIT_TLS=true
  SMTP_USERNAME=<邮件地址>
  SMTP_PASSWORD=<不安全的应用程序密码>
```

启动TLS：

```ini
  # 域：gmail.com、googlemail.com
  SMTP_HOST=smtp.gmail.com
  SMTP_PORT=587
  SMTP_SSL=true
  SMTP_EXPLICIT_TLS=false
  SMTP_USERNAME=<邮件地址>
  SMTP_PASSWORD=<不安全的应用程序密码>
```

另见：https://web.archive.org/web/20210925161633/https://webewizard.com/2019/09/17/Using-Lettre-With-Gmail/

#### Hotmail/Outlook/Office365

```ini
  # 域名：hotmail.com、outlook.com、office365.com
  SMTP_HOST=smtp-mail.outlook.com
  SMTP_PORT=587
  SMTP_SSL=true
  SMTP_EXPLICIT_TLS=false
  SMTP_USERNAME=<邮件地址>
  SMTP_PASSWORD=<密码>
  SMTP_AUTH_MECHANISM="Login"
```

### 发送网格

将 `<full-api-key>` 替换为从 SendGrid 生成的以 `SG` 开头的 API-Key。
还要确保 API-Key 具有完整的`邮件发送`权限，否则您无法使用此密钥登录。
启动TLS：

```ini
  SMTP_HOST=smtp.sendgrid.net
  SMTP_PORT=587
  SMTP_SSL=真
  SMTP_EXPLICIT_TLS=false
  SMTP_USERNAME=apikey
  SMTP_PASSWORD=<full-api-key>
  SMTP_AUTH_MECHANISM="登录"
```

完整的 SSL：

```ini
  SMTP_HOST=smtp.sendgrid.net
  SMTP_PORT=465
  SMTP_SSL=假
  SMTP_EXPLICIT_TLS=true
  SMTP_USERNAME=apikey
  SMTP_PASSWORD=<full-api-key>
  SMTP_AUTH_MECHANISM="登录"
```

### 带有特殊字符的密码

如果您想在密码中使用一些特殊字符，可能需要对其中一些字符进行转义以免混淆环境变量解析器。
例如，可以使用`\` 或`'` 或`"`，但有时需要对它们进行转义，以便实际使用。 如果您使用特殊字符，最好总是在密码周围使用单引号。
我们以下面的密码为例：`~^",a.%\,'}b&@|/c!1(#}`
以下是一些可能会破坏环境变量解析的字符，例如 `\`、`'` 和 `"`。 单个`\`通常用于转义其他字符，因此如果要使用单个`\`，则需要键入`\\`。
此外，引号 `'` 和 `"` 可能会导致一些问题，因此让我们将此密码括在单引号内并转义特殊字符。
为了让上面的密码起作用，我们需要输入 `'~^",a.%\\,\'}b&@|/c!1(#}'`，在这里你看到我们转义了 `\`和 `'` 字符并使用单引号将整个密码括起来。 所以：`~^",a.%\,'}b&@|/c!1(#}` 变成了`'~^",a.%\\,\'}b&@|/c!1(# }'`
