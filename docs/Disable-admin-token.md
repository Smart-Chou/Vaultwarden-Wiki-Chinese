**重要**：任何人都可以访问您的管理页面

如果您想使用另一种方法对 `/admin` 页面进行身份验证，则可以将 `DISABLE_ADMIN_TOKEN` 变量设置为 true。这将禁用用于身份验证的内置“ADMIN_TOKEN”，同时启用管理面板。任何有权访问该 URL 的人都可以访问管理面板。您将需要采取额外的步骤来保护它。这包括外部和本地。

```sh
docker run -d --name bitwarden \
  -e DISABLE_ADMIN_TOKEN=true \
  -v /vw-data/:/data/ \
  -p 80:80 \
  vaultwarden/server:latest
```