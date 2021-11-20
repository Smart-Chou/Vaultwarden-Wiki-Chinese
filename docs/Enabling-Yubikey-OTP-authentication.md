要启用 YubiKey 身份验证，您必须设置“YUBICO_CLIENT_ID”和“YUBICO_SECRET_KEY”环境变量。

如果没有指定 `YUBICO_SERVER`，它将使用默认的 YubiCloud 服务器。您可以为默认的 YubiCloud [此处](https://upgrade.yubico.com/getapikey/) 生成“YUBICO_CLIENT_ID”和“YUBICO_SECRET_KEY”。

笔记：
* 为了生成 API 密钥或将 YubiKey 与 OTP 服务器一起使用，必须注册它。在 [YubiKey 个性化工具](https://www.yubico.com/products/services-software/personalization-tools/use/) 中配置好您的密钥后，您可以在 [此处](https: //upload.yubico.com/)。
* 由于上游问题，服务器版本 1.6.0 或更早版本的 aarch64 版本不支持 Yubikey 功能 - 请参阅 [#262](https://github.com/dani-garcia/vaultwarden/issues/262)。 

```sh
docker run -d --name bitwarden \
  -e YUBICO_CLIENT_ID=12345 \
  -e YUBICO_SECRET_KEY=ABCDEABCDEABCDEABCDE= \
  -v /vw-data/:/data/ \
  -p 80:80 \
  vaultwarden/server:latest
```
