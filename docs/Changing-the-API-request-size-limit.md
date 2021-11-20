默认情况下，API 调用限制为 10MB。对于大多数情况，这应该足够了，但是如果您想支持大型导入，这可能会限制您。另一方面，您可能希望将请求大小限制为小于该值以防止 API 滥用和可能的 DOS 攻击，尤其是在资源有限的情况下。

要设置限制，您可以使用 `ROCKET_LIMITS` 变量。此处的示例显示了正文中发布的 json 的 10MB 限制(这是默认设置)：

```sh
docker run -d --name vaultwarden \
  -e ROCKET_LIMITS={json=10485760} \
  -v /vw-data/:/data/ \
  -p 80:80 \
  vaultwarden/server:latest
```