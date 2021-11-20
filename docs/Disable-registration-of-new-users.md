默认情况下，任何可以访问您的实例的人都可以注册一个新帐户。要禁用此功能，请将 `SIGNUPS_ALLOWED` 环境变量设置为 `false`：

```sh
docker run -d --name bitwarden \
  -e SIGNUPS_ALLOWED=false \
  -v /vw-data/:/data/ \
  -p 80:80 \
  vaultwarden/server:latest
```

请注意，当`SIGNUPS_ALLOWED=false`时，`创建帐户`按钮仍将显示在网络保险库 UI 中，但实际尝试创建帐户将导致错误消息。上游 Bitwarden 并非旨在允许禁用注册，因此无法轻松解决此问题。

## 禁用组织邀请

即使`SIGNUPS_ALLOWED=false`，作为组织所有者或管理员的现有用户仍然可以邀请新用户。如果您也想禁用此功能，请参阅 [禁用邀请](Disable-invitations)。

## 限制某些电子邮件域的注册

您可以通过相应地设置`SIGNUPS_DOMAINS_WHITELIST`来限制注册到来自某些域的电子邮件地址。例如：

- `SIGNUPS_DOMAINS_WHITELIST=example.com`(单域)
- `SIGNUPS_DOMAINS_WHITELIST=example.com,example.net,example.org`(多个域)

!>  如果设置了`SIGNUPS_DOMAINS_WHITELIST`，则忽略`SIGNUPS_ALLOWED` 的值。

您可能还想设置`SIGNUPS_VERIFY=true`，这需要在新注册的用户成功登录之前进行电子邮件验证。这将防止有人使用具有正确域的虚假电子邮件地址进行注册。

## 通过管理页面的邀请

Vaultwarden 管理员可以通过 [启用管理页面](Enabling-admin-page) 邀请任何人，不管上述任何限制。