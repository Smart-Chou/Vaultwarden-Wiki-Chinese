**重要**：强烈建议在启用此功能之前激活 HTTPS，以避免可能的 MITM 攻击。

此页面允许服务器管理员查看所有注册用户并删除它们。它还允许邀请新用户，即使注册被禁用。

要启用管理页面，您需要设置身份验证令牌。这个令牌可以是任何东西，但建议使用长的、随机生成的字符串，例如运行`openssl rand -base64 48`。 **将此令牌保密，这是访问服务器管理区域的密码！**

要设置令牌，请使用“ADMIN_TOKEN”变量：

```sh
docker run -d --name bitwarden \
  -e ADMIN_TOKEN=some_random_token_as_per_above_explanation \
  -v /vw-data/:/data/ \
  -p 80:80 \
  vaultwarden/server:latest
```

之后，该页面将在`/admin` 子目录中可用。

您第一次在管理页面中保存设置时，将在您的“DATA_FOLDER”中生成“config.json”。此文件中的值将优先于相应的环境变量。

请注意，在单击“保存”按钮之前，管理页面中的配置更改不会生效。例如，如果您正在测试 SMTP 设置，并且您更改了“SMTP 身份验证机制”设置，然后单击“发送测试电子邮件”来测试更改，这将不会按预期工作——因为您没有单击“保存” `，`SMTP Auth 机制` 更改不会生效。

**注意：** 更改 `ADMIN_TOKEN` 后，当前登录的管理员仍然可以使用他们的旧登录令牌 [最多 20 分钟](https://github.com/dani-garcia/vaultwarden/blob/master/src/api/admin.rs#L87)。