要启用 U2F 身份验证，您必须从具有有效证书的 HTTPS 域（使用包含的
HTTPS 选项或使用反向代理）。我们建议使用 Let's Encrypt 的免费证书。

之后，您需要将 `DOMAIN` 环境变量设置为提供 Vaultwarden 的相同地址：

```sh
docker run -d --name vaultwarden \
  -e DOMAIN=https://vw.domain.tld \
  -v /vw-data/:/data/ \
  -p 80:80 \
  vaultwarden/server:latest
```

请注意，该值必须包含 `https://`，并且在不使用 `443` 时，它可能会在末尾包含一个端口（格式为 `https://vw.domain.tld:port`）。