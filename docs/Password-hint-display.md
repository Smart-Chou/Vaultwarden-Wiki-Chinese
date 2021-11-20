通常，密码提示通过电子邮件发送。但由于 Vaultwarden 是针对小型或个人部署而设计的，因此密码提示页面也提供了提示，因此您无需配置电子邮件服务。如果要禁用此功能，可以使用 `SHOW_PASSWORD_HINT` 变量：

```sh
docker run -d --name vaultwarden \
  -e SHOW_PASSWORD_HINT=false \
  -v /vw-data/:/data/ \
  -p 80:80 \
  vaultwarden/server:latest
```