[Docker Compose](https://docs.docker.com/compose/) 是一个允许定义和配置多容器应用程序的工具。在我们的例子中，我们希望 Vaultwarden 服务器和代理将 WebSocket 请求重定向到正确的位置。

## Caddy 与 HTTP 挑战

此示例假设您已[安装](https://docs.docker.com/compose/install/) Docker Compose，您有一个用于 vaultwarden 实例的域名(例如，`vaultwarden.example.com`)，并且它将是公开的。

首先创建一个新目录并更改为该目录。接下来，在下面创建 `docker-compose.yml`，确保为 `DOMAIN` 和 `EMAIL` 变量替换适当的值。

```yaml
version: '3'

services:
  vaultwarden:
    image: vaultwarden/server:latest
    container_name: vaultwarden
    restart: always
    environment:
      - WEBSOCKET_ENABLED=true  # 启用 WebSocket 通知。
    volumes:
      - ./vw-data:/data

  caddy:
    image: caddy:2
    container_name: caddy
    restart: always
    ports:
      - 80:80  # ACME HTTP-01 挑战需要
      - 443:443
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile:ro
      - ./caddy-config:/config
      - ./caddy-data:/data
    environment:
      - DOMAIN=https://vaultwarden.example.com  # 你的域名。
      - EMAIL=admin@example.com                 # 用于 ACME 注册的电子邮件地址。
      - LOG_FILE=/data/access.log
```

在同一目录中，创建下面的`Caddyfile`。 (这个文件不需要修改。)
```
{$DOMAIN}:443 {
  log {
    level INFO
    output file {$LOG_FILE} {
      roll_size 10MB
      roll_keep 10
    }
  }

  # 使用 ACME HTTP-01 质询获取已配置域的证书。
  tls {$EMAIL}

  # 此设置可能与某些浏览器存在兼容性问题
  #(例如，在 Firefox 上下载附件)。尝试禁用此功能
  #如果你遇到问题。
  encode gzip

  # 通知重定向到 WebSocket 服务器
  reverse_proxy /notifications/hub vaultwarden:3012

  # 将其他所有内容代理给 Rocket
  reverse_proxy vaultwarden:80 {
       # Send the true remote IP to Rocket, so that vaultwarden can put this in the
       # log, so that fail2ban can ban the correct IP.
       header_up X-Real-IP {remote_host}
  }
}
```

运行
```bash
docker-compose up -d
```
以创建和启动容器。这个 `docker-compose.yml` 文件中的服务的专用网络将自动创建，只有 Caddy 是公开的。

```bash
docker-compose down
```
stops and destroys the containers.

[此处](https://github.com/sosandroid/docker-bitwarden_rs-caddy-synology) 提供了一个类似的基于 Caddy 的 Synology 示例。

## 球童与 DNS 挑战

此示例与上一个示例相同，但适用于您不希望您的实例可公开访问的情况(即，您只能从本地网络访问它)。此示例使用 Duck DNS 作为 DNS 提供程序。有关更多背景信息以及如何设置 Duck DNS 的详细信息，请参阅[使用 Let's Encrypt 证书运行私有 Vaultwarden 实例](Running-a-private-vaultwarden-instance-with-Let's-Encrypt-certs)。

首先创建一个新目录并更改为该目录。接下来，在下面创建 `docker-compose.yml`，确保为 `DOMAIN` 和 `EMAIL` 变量替换适当的值。

```yaml
version: '3'

services:
  vaultwarden:
    image: vaultwarden/server:latest
    container_name: vaultwarden
    restart: always
    environment:
      - WEBSOCKET_ENABLED=true  # Enable WebSocket notifications.
    volumes:
      - ./vw-data:/data

  caddy:
    image: caddy:2
    container_name: caddy
    restart: always
    ports:
      - 80:80
      - 443:443
    volumes:
      - ./caddy:/usr/bin/caddy  # Your custom build of Caddy.
      - ./Caddyfile:/etc/caddy/Caddyfile:ro
      - ./caddy-config:/config
      - ./caddy-data:/data
    environment:
      - DOMAIN=https://vaultwarden.example.com  # Your domain.
      - EMAIL=admin@example.com                 # The email address to use for ACME registration.
      - DUCKDNS_TOKEN=<token>                   # Your Duck DNS token.
      - LOG_FILE=/data/access.log
```

库存的 Caddy 构建(包括 Docker 镜像中的构建)不包含 DNS 挑战模块，因此接下来您需要[获取自定义 Caddy 构建](https://github.com/dani-garcia/vaultwarden/wiki/Running-a-private-vaultwarden-instance-with-Let%27s-Encrypt-certs#getting-a-custom-caddy-build)。将自定义构建重命名为 `caddy` 并将其移动到与 `docker-compose.yml` 相同的目录下。确保 `caddy` 文件是可执行的(例如，`chmod a+x caddy`)。上面的`docker-compose.yml` 文件将自定义构建绑定安装到`caddy:2` 容器中，替换了库存构建。

在同一目录中，创建下面的`Caddyfile`。 (这个文件不需要修改。)
```
{$DOMAIN}:443 {
  log {
    level INFO
    output file {$LOG_FILE} {
      roll_size 10MB
      roll_keep 10
    }
  }

  # 使用 ACME DNS-01 质询获取已配置域的证书。
  tls {
    dns duckdns {$DUCKDNS_TOKEN}
  }

  # 此设置可能与某些浏览器存在兼容性问题
  #(例如，在 Firefox 上下载附件)。尝试禁用此功能
  #如果你遇到问题。
  encode gzip

  # 通知重定向到 WebSocket 服务器
  reverse_proxy /notifications/hub vaultwarden:3012

  # 将其他所有内容代理给 Rocket
  reverse_proxy vaultwarden:80
}
```

与 HTTP 质询示例一样，运行
```bash
docker-compose up -d
```
去创建和启动容器。