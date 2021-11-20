LDAP 集成是使用查询 LDAP 并邀请用户加入您的 Vaultwarden 实例的小型服务执行的。该服务毫无创意地命名为 [vaultwarden_ldap](https://github.com/ViViDboarder/vaultwarden_ldap)。

它尚未作为二进制文件分发，但有一个可用的 Docker 镜像 [vividboarder/vaultwarden_ldap](https://hub.docker.com/r/vividboarder/vaultwarden_ldap)。

在部署之前，您必须[[启用您的保管库管理员页面|启用管理页面]]。这将启用 LDAP 同步服务将用于邀请用户的 API。您设置的“ADMIN_TOKEN”将在配置 LDAP 同步服务时使用。您还必须确保**不**禁用邀请功能。要验证这一点，请仔细检查环境变量“INVITATIONS_ALLOWED”是否未设置为“false”。

还建议从您的 Vaultwarden 实例[[启用电子邮件发送|SMTP-配置]]，以便通知您的用户可以创建帐户。如果您不这样做，如果他们使用他们的 LDAP 电子邮件地址，他们仍然可以注册，但您必须自己通知他们。

完成这些步骤后，您可以配置和部署 LDAP 同步服务。可以在服务 [Readme](https://github.com/ViViDboarder/vaultwarden_ldap) 本身上找到最新的说明，但它涉及创建带有连接信息的 `config.toml` 文件您的 Vaultwarden 实例、您的 LDAP 实例以及您想用来查找用户的 LDAP 查询。