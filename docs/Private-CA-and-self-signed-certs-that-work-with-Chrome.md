> [!warning] 
> 此方法仅用于测试和开发。绝大多数用户不应使用此方法，因为它需要在您的每台设备上加载证书，这既容易出错又需要将来维护。相反，将精力集中在通过 [Let's Encrypt](https://letsencrypt.org/getting-started/) 获取真实证书上。如果您的 vaultwarden 实例不在公共 Internet 上，这甚至可以工作([使用 Let's Encrypt 证书运行私有 Vaultwarden 实例](Running-a-private-vaultwarden-instance-with-Let's-Encrypt-certs))。

!> 不支持此方法。请不要打开 GitHub 问题或在论坛上发帖询问如何使其工作。

---

为了让 bitwarden 使用自签名证书正常工作，Chrome 需要证书在证书的备用名称字段中包含域名。

创建 CA 密钥(您自己的小型本地证书颁发机构)：

```
openssl genpkey -algorithm RSA -aes128 -out private-ca.key -outform PEM -pkeyopt rsa_keygen_bits:2048
```

注意：你也可以使用旧的 `-des3` 代替 `-aes128`。

创建 CA 证书：

```
openssl req -x509 -new -nodes -sha256 -days 3650 -key private-ca.key -out self-signed-ca-cert.crt
```

注意：`-nodes` 参数防止在测试/安全环境中为私钥(密钥对)设置密码短语，否则每次启动/重启服务器时都必须输入密码短语。

创建一个bitwarden键：

```
openssl genpkey -algorithm RSA -out bitwarden.key -outform PEM -pkeyopt rsa_keygen_bits:2048
```

创建 bitwarden 证书请求文件：

```
openssl req -new -key bitwarden.key -out bitwarden.csr
```

使用以下内容创建一个文本文件`bitwarden.ext`，将域名更改为您的设置。

```
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1 = bitwarden.local
DNS.2 = www.bitwarden.local
```


创建从根 CA 签名的 bitwarden 证书：

```
openssl x509 -req -in bitwarden.csr -CA self-signed-ca-cert.crt -CAkey private-ca.key -CAcreateserial -out bitwarden.crt -days 365 -sha256 -extfile bitwarden.ext
```

注意：截至 2019 年 4 月 iOS 13+ 和 macOS 15+，服务器证书的到期时间不能 > 825，并且必须包含 ExtendedKeyUsage 扩展 https://support.apple.com/en-us/HT210176
 
将根证书和 bitwarden 证书添加到客户端计算机。

如需参考，请参见此处：https://deliciousbrains.com/ssl-certificate-authority-for-local-https-development/