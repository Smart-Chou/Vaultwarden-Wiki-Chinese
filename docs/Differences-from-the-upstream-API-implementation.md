## 邀请用户加入组织

### 启用 SMTP

受邀用户将收到一封电子邮件，其中包含一个有效期为 5 天的链接。点击链接后，用户可以选择创建帐户或登录。新​​用户需要创建一个新帐户；被邀请加入新组织的现有用户只需登录。在任一步骤之后，他们将在管理界面中显示为`已接受`，并在组织管理员确认后将添加到组织。

### 没有启用 SMTP

受邀用户不会收到邀请邮件；相反，所有已注册的用户都将出现在界面中，就好像他们已经接受了邀请一样。组织管理员然后只需要确认他们是合适的组织成员并授予他们访问共享机密的权限。

尚未注册的受邀用户将在组织管理界面中显示为`已邀请`。同时创建一个邀请记录，允许用户注册，即使[禁止新用户注册](Disable-registration-of-new-users)。 (除非您[禁用邀请](Disable-invitations))他们将在注册后自动变为`已接受`。从那里组织管理员可以确认他们以授予他们对组织的访问权限。

## 在未加密的连接上运行

强烈建议通过 HTTPS 运行 Vaultwarden 服务。然而服务器本身虽然 [开启HTTPS](Enabling-HTTPS) 并不严格要求这样的设置。这使得在您通常可以信任连接(内部和安全网络、通过 VPN 访问等)的情况下启动服务更容易一些，或者当您想将服务置于 HTTP 代理之后，这将进行加密在代理端。

如果您使用非常强大的主密码，并且避免在容易受到 MITM 攻击的连接上使用 Web Vault，那么通过 HTTP 运行仍然相当安全，攻击者可以将 javascript 注入您的界面。但是，某些形式的 2FA 在此设置中可能不起作用，并且 [Vault 在 Chrome 中的此配置中不起作用](https://github.com/bitwarden/web/issues/254)。