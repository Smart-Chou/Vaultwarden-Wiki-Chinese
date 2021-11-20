Dockerfile(Caddy 生成器)

```nginx
FROM caddy:builder AS builder
RUN xcaddy build --with github.com/caddy-dns/cloudflare

FROM caddy:latest
COPY --from=builder /usr/bin/caddy /usr/bin/caddy
```

构建命令

```bash
docker build -t [你的名字]/caddycfdns。
```

Caddyfile(作为反向代理)

```nginx
https://[您的域名]:443 {

  tls {
        dns cloudflare [API-KEY]
  }

  encode gzip

   header / {
       # 启用 HTTP 严格传输安全 (HSTS)
        Strict-Transport-Security "max-age=31536000;"
       # 启用跨站过滤器 (XSS) 并告诉浏览器阻止检测到的攻击
        X-XSS-Protection "1; mode=block"
       # 禁止在框架内渲染站点(点击劫持保护)
       X-Frame-Options "DENY"
       # 防止搜索引擎索引(可选)
       X-Robots-Tag "none"
       # 删除服务器名称
       -Server
   }
  # 协商端点也代理到 Rocket
  reverse_proxy /notifications/hub/negotiate vaultwarden:80

  # 通知重定向到 websockets 服务器
  reverse_proxy /notifications/hub Vaultwarden:3012

  # 将根目录代理到 Rocket
  reverse_proxy Vaultwarden:80 {
       # 将真正的远程IP发送给Rocket，以便Vaultwarden可以将其放入
       # 日志，以便fail2ban 可以禁止正确的IP。
       header_up X-Real-IP {remote_host}
  }
}
```

docker-compose.yml

```nginx
version: '3'

services:
  vaultwarden:
    image: vaultwarden/server
    restart: always
    volumes:
      - $PWD/vw-data:/data
    environment:
      WEBSOCKET_ENABLED: 'true' # Required to use websockets
      SIGNUPS_ALLOWED: 'false'   # set to false to disable signups
      DOMAIN: 'https://[DOMAIN]'
      SMTP_HOST: '[MAIL-SERVER]'
      SMTP_FROM: '[E-MAIL]'
      SMTP_PORT: '587'
      SMTP_SSL: 'true'
      SMTP_USERNAME: '[E-MAIL]'
      SMTP_PASSWORD: '[SMTP-PASS]'
#      ADMIN_TOKEN: '[RAND. GENERATE]'
#      YUBICO_CLIENT_ID: '[OPTIONAL]'
#      YUBICO_SECRET_KEY: '[OPTIONAL]'

  caddy:
    image: [YOUR-NAME]/caddycfdns
    restart: always
    volumes:
      - $PWD/Caddyfile:/etc/caddy/Caddyfile
      - caddy_data:/data
      - caddy_config:/config
      - caddy_log:/logs
    ports:
      - [PRIVATE-IP]:443:443
    environment:
      ACME_AGREE: 'true'
      CLOUDFLARE_EMAIL: '[YOUR-EMAIL]'
      CLOUDFLARE_API_TOKEN: '[YOUR-TOKEN]'
      DOMAIN: '[DOMAIN]'

volumes:
  caddy_data:
  caddy_config:
  caddy_log:
```
