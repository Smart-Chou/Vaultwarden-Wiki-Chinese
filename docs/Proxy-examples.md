在本文档中，`<SERVER>` 是指您访问 Vaultwarden 的 IP 或域。如果反向代理和 Vaultwarden 都在同一个系统上运行，只需使用 `localhost`。

默认情况下，Vaultwarden 在端口 80 上侦听 Web (REST API) 流量，在端口 3012 上侦听 WebSocket 流量(如果 [启用WebSocket通知](Enabling-WebSocket-notifications) 已启用)。反向代理应配置为终止 SSL/TLS 连接(最好在端口 443，HTTPS 的标准端口上)。然后，反向代理根据需要将传入的客户端请求传递到端口 80 或 3012 上的 vaultwarden，并在收到来自 vaultwarden 的响应后，将该响应传递回客户端。

请注意，当您将 vaultwarden 置于反向代理之后时，反向代理和 vaultwarden 之间的连接通常被假定为通过安全的专用网络，因此不需要加密。下面的示例假设您在此配置中运行，在这种情况下，您不应启用 vaultwarden 内置的 HTTPS 功能(即，不应设置 `ROCKET_TLS` 环境变量)。如果这样做，连接将失败，因为反向代理使用 HTTP 连接到 vaultwarden，但您将 vaultwarden 配置为期望使用 HTTPS。

通常使用 [Docker Compose](https://docs.docker.com/compose/) 将容器化服务链接在一起(例如，Vaultwarden 和反向代理)。有关此示例，请参阅 [使用Docker Compose](Using-Docker-Compose)
可以使用 Mozilla 的 [SSL 配置生成器](https://ssl-config.mozilla.org) 生成网络服务器的安全 TLS 协议和密码配置。已知所有支持的浏览器和移动应用程序都可以使用`现代`配置。

## Caddy 2.x

Caddy 2 可以在某些情况下自动启用 HTTPS，检查[docs](https://caddyserver.com/docs/automatic-https).

在 Caddyfile 语法中，`{$VAR}` 表示环境变量 `VAR` 的值。 如果您愿意，也可以直接指定一个值而不是替换一个 env var 值。

```
{$DOMAIN}:443 {
  log {
    level INFO
    output file {$LOG_FILE} {
      roll_size 10MB
      roll_keep 10
    }
  }

  # 如果您想通过 ACME(Let's Encrypt 或 ZeroSSL)获得证书，请取消注释。
  # tls {$EMAIL}

  # 或者，如果您提供自己的证书，则取消注释。你也可以使用这个选项
  # 如果您在 Cloudflare 后面运行。
  # tls {$SSL_CERT_PATH} {$SSL_KEY_PATH}

  # 此设置可能与某些浏览器存在兼容性问题
  #(例如，在 Firefox 上下载附件)。尝试禁用此功能
  #如果你遇到问题。
  encode gzip

  # 取消注释以提高安全性(警告：仅在您了解含义时才使用！)
  # header {
  #      # 启用 HTTP 严格传输安全 (HSTS)
  #      Strict-Transport-Security "max-age=31536000;"
  #      # 启用跨站点过滤器 (XSS) 并告诉浏览器阻止检测到的攻击
  #      X-XSS-Protection "1; mode=block"
  #      # 禁止在框架内渲染站点(点击劫持保护)
  #      X-Frame-Options "DENY"
  #      # 防止搜索引擎索引(可选)
  #      X-Robots-Tag "none"
  #      # 删除服务器名称
  #      -Server
  # }

  # 取消注释以仅允许从本地网络访问管理界面
  # @insecureadmin {
  #   not remote_ip 192.168.0.0/16 172.16.0.0/12 10.0.0.0/8
  #   path /admin*
  # }
  # redir @insecureadmin /

  # 通知重定向到 websockets 服务器
  reverse_proxy /notifications/hub <SERVER>:3012

  # 将其他所有内容代理给 Rocket
  reverse_proxy <SERVER>:80 {
       # 将真正的远程IP发送给Rocket，以便Vaultwarden可以将其放入
       # 日志，以便fail2ban 可以禁止正确的IP
       header_up X-Real-IP {remote_host}
  }
}
```

## lighttpd (by forkbomb9)

```
server.modules += ( "mod_proxy" )

$HTTP["host"] == "vault.example.net" {
    $HTTP["url"] == "/notifications/hub" {
       # WebSocket proxy
       proxy.server  = ( "" => ("vaultwarden" => ( "host" => "<SERVER>", "port" => 3012 )))
       proxy.forwarded = ( "for" => 1 )
       proxy.header = (
           "https-remap" => "enable",
           "upgrade" => "enable",
           "connect" => "enable"
       )
    } else {
       proxy.server  = ( "" => ("vaultwarden" => ( "host" => "<SERVER>", "port" => 4567 )))
       proxy.forwarded = ( "for" => 1 )
       proxy.header = ( "https-remap" => "enable" )
    }
}
```

在 Vaultwarden 环境中，您必须将`IP_HEADER`设置为`X-Forwarded-For`而不是`X-Real-IP`。

## Nginx (by shauder)

```nginx
server {
  listen 443 ssl http2;
  server_name vault.*;
  
  # Specify SSL config if using a shared one.
  #include conf.d/ssl/ssl.conf;
  
  # Allow large attachments
  client_max_body_size 128M;

  location / {
    proxy_pass http://<SERVER>:80;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
  }
  
  location /notifications/hub {
    proxy_pass http://<SERVER>:3012;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
  }
  
  location /notifications/hub/negotiate {
    proxy_pass http://<SERVER>:80;
  }

  # Optionally add extra authentication besides the ADMIN_TOKEN
  # If you don't want this, leave this part out
  location /admin {
    # See: https://docs.nginx.com/nginx/admin-guide/security-controls/configuring-http-basic-authentication/
    auth_basic "Private";
    auth_basic_user_file /path/to/htpasswd_file;

    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;

    proxy_pass http://<SERVER>:80;
  }

}
```

如果您遇到 504 Gateway Timeout 问题，请通过向 `server {` 部分添加更长的超时时间来告诉 nginx 等待更长时间的保管库，例如：

```nginx
  proxy_connect_timeout       777;
  proxy_send_timeout          777;
  proxy_read_timeout          777;
  send_timeout                777;
```

## Nginx with sub-path (by BlackDex)

在此示例中，Vaultwarden 将通过 https://bitwarden.example.tld/vault/
提供 如果你想使用任何其他子路径，比如 `bitwarden` 或 `secret-vault`，你应该在下面的例子中更改 `/vault/` 以匹配。

为此，您需要配置您的`DOMAIN`变量以使其匹配，因此它应该如下所示：

```ini
; 添加子路径！否则这是行不通的！
DOMAIN=https://bitwarden.example.tld/vault/
```

```nginx
# 在这里定义服务器IP和端口。
upstream vaultwarden-default { server 127.0.0.1:8080; }
upstream vaultwarden-ws { server 127.0.0.1:3012; }

# 将 HTTP 重定向到 HTTPS
server {
    listen 80;
    listen [::]:80;
    server_name vaultwarden.example.tld;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name vaultwarden.example.tld;

    # Specify SSL Config when needed
    #ssl_certificate /path/to/certificate/letsencrypt/live/vaultwarden.example.tld/fullchain.pem;
    #ssl_certificate_key /path/to/certificate/letsencrypt/live/vaultwarden.example.tld/privkey.pem;
    #ssl_trusted_certificate /path/to/certificate/letsencrypt/live/vaultwarden.example.tld/fullchain.pem;

    client_max_body_size 128M;

    ## Using a Sub Path Config
    # Path to the root of your installation
    location /vault/ {
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto $scheme;

      proxy_pass http://vaultwarden-default;
    }

    location /vault/notifications/hub/negotiate {
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto $scheme;

      proxy_pass http://vaultwarden-default;
    }

    location /vault/notifications/hub {
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection $http_connection;
      proxy_set_header X-Real-IP $remote_addr;

      proxy_pass http://vaultwarden-ws;
    }

    # Optionally add extra authentication besides the ADMIN_TOKEN
    # If you don't want this, leave this part out
    location ^~ /vault/admin {
      # See: https://docs.nginx.com/nginx/admin-guide/security-controls/configuring-http-basic-authentication/
      auth_basic "Private";
      auth_basic_user_file /path/to/htpasswd_file;

      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto $scheme;

      proxy_pass http://vaultwarden-default;
    }

}
```

## Nginx (by ypid)

Ansible 库存示例，使用 DebOps 将 nginx 配置为 Vaultwarden 的反向代理。我选择在 URL 中使用 PSK 以提高安全性，以免将 API 暴露给 Internet 上的每个人，因为客户端应用程序尚不支持客户端证书(我已对其进行了测试)。注意：使用subpath/PSK需要对源代码打补丁重新编译，参考：https://github.com/dani-garcia/vaultwarden/issues/241#issuecomment-436376497。 /admin 未经测试。有关安全的子路径托管的一般讨论，请参阅：https://github.com/debops/debops/issues/1233

```
bitwarden__fqdn: 'vault.example.org'

nginx__upstreams:

  - name: 'bitwarden'
    type: 'default'
    enabled: True
    server: 'localhost:8000'

nginx__servers:

  - name: '{{ bitwarden__fqdn }}'
    filename: 'debops.bitwarden'
    by_role: 'debops.bitwarden'
    favicon: False
    root: '/usr/share/vaultwarden/web-vault'

    location_list:

      - pattern: '/'
        options: |-
          deny all;

      - pattern: '= /ekkP9wtJ_psk_changeme_Hr9CCTud'
        options: |-
          return 307 $scheme://$host$request_uri/;

      ## 然后，nginx 也需要设置所有安全 HTTP 标头。
      # - pattern: '/ekkP9wtJ_psk_changeme_Hr9CCTud/'
      #   options: |-
      #     alias /usr/share/vaultwarden/web-vault/;

      - pattern: '/ekkP9wtJ_psk_changeme_Hr9CCTud/'
        options: |-
          proxy_set_header Host              $host;
          # proxy_set_header X-Real-IP         $remote_addr;
          # proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
          proxy_set_header X-Forwarded-Proto $scheme;
          proxy_set_header X-Forwarded-Port  443;

          proxy_pass http://bitwarden;

      ##不要使用图标功能，只要它显示来自域
      ## 我们对服务器的凭据。
      - pattern: '/ekkP9wtJ_psk_changeme_Hr9CCTud/icons/'
        options: |-
          access_log off;
          log_not_found off;
          deny all;
```

## Nginx (NixOS)(by tklitschi)

NixOS nginx 配置示例。有关 NixOS 部署的更多信息，请参阅 [部署 Wiki 页面](Deployment-examples)。

```nix
{ config, ... }:
{
  security.acme.acceptTerms = true;
  security.acme.email = "me@example.com";
  security.acme.certs = {

    "vw.example.com" = {
      group = "vaultwarden";
      keyType = "rsa2048";
      allowKeysForGroup = true;
    };
  };

  services.nginx = {
    enable = true;

    recommendedGzipSettings = true;
    recommendedOptimisation = true;
    recommendedProxySettings = true;
    recommendedTlsSettings = true;

    virtualHosts = {
      "vw.example.com" = {
        forceSSL = true;
        enableACME = true;
        locations."/" = {
          proxyPass = "http://localhost:8812"; #changed the default rocket port due to some conflict
          proxyWebsockets = true;
        };
        locations."/notifications/hub" = {
          proxyPass = "http://localhost:3012";
          proxyWebsockets = true;
        };
        locations."/notifications/hub/negotiate" = {
          proxyPass = "http://localhost:8812";
          proxyWebsockets = true;
        };
      };
    };
  };
}
```

## Apache (by fbartels)

请记住启用 `mod_proxy_wstunnel` 和 `mod_proxy_http`，例如：`a2enmod proxy_wstunnel` 和 `a2enmod proxy_http`。

```
<VirtualHost *:443>
    SSLEngine on
    ServerName bitwarden.$hostname.$domainname

    SSLCertificateFile ${SSLCERTIFICATE}
    SSLCertificateKeyFile ${SSLKEY}
    SSLCACertificateFile ${SSLCA}
    ${SSLCHAIN}

    ErrorLog ${APACHE_LOG_DIR}/bitwarden-error.log
    CustomLog ${APACHE_LOG_DIR}/bitwarden-access.log combined

    RewriteEngine On
    RewriteCond %{HTTP:Upgrade} =websocket [NC]
    RewriteRule /notifications/hub(.*) ws://<SERVER>:3012/$1 [P,L]
    ProxyPass / http://<SERVER>:80/

    ProxyPreserveHost On
    ProxyRequests Off
    RequestHeader set X-Real-IP %{REMOTE_ADDR}s
</VirtualHost>
```

## Apache in a sub-location (by ss89)

修改您的 docker 启动以包含子位置。

```
; 添加子位置！否则这是行不通的！
DOMAIN=https://$hostname.$domainname/$sublocation/
```

确保您在 apache 配置中的某处加载了 websocket 代理模块。 它看起来像：

```
LoadModule proxy_wstunnel_module modules/mod_proxy_wstunnel.so`
```

在某些操作系统上，您可以使用 a2enmod，例如：`a2enmod proxy_wstunnel` 和 `a2enmod proxy_http`。

```
<VirtualHost *:443>
    SSLEngine on
    ServerName $hostname.$domainname

    SSLCertificateFile ${SSLCERTIFICATE}
    SSLCertificateKeyFile ${SSLKEY}
    SSLCACertificateFile ${SSLCA}
    ${SSLCHAIN}

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined

    <Location /$sublocation/> #adjust here if necessary
        RewriteEngine On
        RewriteCond %{HTTP:Upgrade} =websocket [NC]
        RewriteRule /notifications/hub(.*) ws://<SERVER>:3012/$1 [P,L]
        ProxyPass http://<SERVER>:80/$sublocation/

        ProxyPreserveHost On
        RequestHeader set X-Real-IP %{REMOTE_ADDR}s
    </Location>
</VirtualHost>
```

## Traefik v1 (docker-compose example)

```yaml
labels:
    - traefik.enable=true
    - traefik.docker.network=traefik
    - traefik.web.frontend.rule=Host:bitwarden.domain.tld
    - traefik.web.port=80
    - traefik.hub.frontend.rule=Host:bitwarden.domain.tld;Path:/notifications/hub
    - traefik.hub.port=3012
    - traefik.hub.protocol=ws
```

## Traefik v2 (docker-compose example by hwwilliams)

**Traefik v1 labels migrated to Traefik v2**

```yaml
labels:
  - traefik.enable=true
  - traefik.docker.network=traefik
  - traefik.http.routers.bitwarden-ui.rule=Host(`bitwarden.domain.tld`)
  - traefik.http.routers.bitwarden-ui.service=bitwarden-ui
  - traefik.http.services.bitwarden-ui.loadbalancer.server.port=80
  - traefik.http.routers.bitwarden-websocket.rule=Host(`bitwarden.domain.tld`) && Path(`/notifications/hub`)
  - traefik.http.routers.bitwarden-websocket.service=bitwarden-websocket
  - traefik.http.services.bitwarden-websocket.loadbalancer.server.port=3012
```

**迁移标签加上 HTTP 到 HTTPS 重定向**

这些标签假设 Traefik 中为端口 80 和 443 定义的入口点分别是`web`和`websecure`。

这些标签还假设您已经在 Traefik 中定义了一个默认的证书解析器。

```yaml
labels:
  - traefik.enable=true
  - traefik.docker.network=traefik
  - traefik.http.middlewares.redirect-https.redirectScheme.scheme=https
  - traefik.http.middlewares.redirect-https.redirectScheme.permanent=true
  - traefik.http.routers.bitwarden-ui-https.rule=Host(`bitwarden.domain.tld`)
  - traefik.http.routers.bitwarden-ui-https.entrypoints=websecure
  - traefik.http.routers.bitwarden-ui-https.tls=true
  - traefik.http.routers.bitwarden-ui-https.service=bitwarden-ui
  - traefik.http.routers.bitwarden-ui-http.rule=Host(`bitwarden.domain.tld`)
  - traefik.http.routers.bitwarden-ui-http.entrypoints=web
  - traefik.http.routers.bitwarden-ui-http.middlewares=redirect-https
  - traefik.http.routers.bitwarden-ui-http.service=bitwarden-ui
  - traefik.http.services.bitwarden-ui.loadbalancer.server.port=80
  - traefik.http.routers.bitwarden-websocket-https.rule=Host(`bitwarden.domain.tld`) && Path(`/notifications/hub`)
  - traefik.http.routers.bitwarden-websocket-https.entrypoints=websecure
  - traefik.http.routers.bitwarden-websocket-https.tls=true
  - traefik.http.routers.bitwarden-websocket-https.service=bitwarden-websocket
  - traefik.http.routers.bitwarden-websocket-http.rule=Host(`bitwarden.domain.tld`) && Path(`/notifications/hub`)
  - traefik.http.routers.bitwarden-websocket-http.entrypoints=web
  - traefik.http.routers.bitwarden-websocket-http.middlewares=redirect-https
  - traefik.http.routers.bitwarden-websocket-http.service=bitwarden-websocket
  - traefik.http.services.bitwarden-websocket.loadbalancer.server.port=3012
```

## HAproxy (by BlackDex)

将这些行添加到您的 haproxy 配置中。

```
frontend vaultwarden
    bind 0.0.0.0:80
    option forwardfor header X-Real-IP
    http-request set-header X-Real-IP %[src]
    default_backend vaultwarden_http
    use_backend vaultwarden_ws if { path_beg /notifications/hub } !{ path_beg /notifications/hub/negotiate }

backend vaultwarden_http
    # Enable compression if you want
    # compression algo gzip
    # compression type text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript
    server vwhttp 0.0.0.0:8080

backend vaultwarden_ws
    server vwws 0.0.0.0:3012
```

## HAproxy (by [@williamdes](https://github.com/williamdes))

将这些行添加到您的 HAproxy 配置中。

```
backend static-success-default
  mode http
  errorfile 503 /usr/local/etc/haproxy/static/index.static.default.html
  errorfile 200 /usr/local/etc/haproxy/static/index.static.default.html

frontend http-in
    bind *:80
    bind *:443 ssl crt /acme.sh/domain.tld/domain.tld.pem
    option forwardfor header X-Real-IP
    http-request set-header X-Real-IP %[src]
    default_backend static-success-default

    # Define hosts
    acl host_bitwarden_domain_tld hdr(Host) -i bitwarden.domain.tld

    ## 找出使用哪个
    use_backend vaultwarden_http if host_bitwarden_domain_tld !{ path_beg /notifications/hub } or { path_beg /notifications/hub/negotiate }
    use_backend vaultwarden_ws if host_bitwarden_domain_tld { path_beg /notifications/hub } !{ path_beg /notifications/hub/negotiate }

backend vaultwarden_http
    # 如果需要，启用压缩
    # 压缩算法 gzip
    # 压缩类型 text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript
    # 如果您在 docker-compose 中使用 haproxy，则可以使用容器主机名
    server vw_http 0.0.0.0:8080

backend vaultwarden_ws
    # 如果您在 docker-compose 中使用 haproxy，则可以使用容器主机名
    server vw_ws 0.0.0.0:3012
```

## HAproxy inside PfSense (by [@RichardMawdsley](https://github.com/RichardMawdsley))

作为 GUI 设置，下面的详细信息说明供您在需要的地方添加。

* 假设您已经有基本的 HTTP>HTTPS 重定向设置 [基本设置](https://blog.devita.co/pfsense-to-proxy-traffic-for-websites-using-pfsense/)

### 后端创建

Backend 1:

```
Mode	Name	                   Forwardto	     Address	     Port	 Encrypt(SSL)	SSL checks	Weight	Actions
active 	Vaultwarden                Address+Port:     IPADDRESSHERE   80          no             no
```

Backend 2:

```
Mode	Name	                   Forwardto	     Address	     Port	 Encrypt(SSL)	SSL checks	Weight	Actions
active 	Vaultwarden-Notifications  Address+Port:     IPADDRESSHERE   3012        no             no
```

### 前端创建 - 1 - 域

**ACCESS CONTROL LIST**

```
ACL00
Host matches:
no
no
FQDN.com     -  NOTE:  This needs to be your root domain.  
 	
ACL00
Path starts with:
no
yes
/big-ass-randomised-test-that-really-no-one-is-ever-going-to-type-DONT-USE-THIS-LINE-THOUGH-make-your-own-up

ACL01
Host matches:
no
no
VAULTWARDEN.MYDOMAIN.COM

ACL01
Host matches:
no
no
EXAMPLE-OTHER-SUB-DOMAIN-1.MYDOMAIN.COM

ACL01
Host matches:
no
no
EXAMPLE-OTHER-SUB-DOMAIN-2.MYDOMAIN.COM
```

**ACTIONS - 1 - Domain**

```
http-request allow
See below
ACL01

http-request deny
See below
ACL00
```

### 前端创建 - 2 - VaultWarden

**ACCESS CONTROL LIST**

```
ACL1
Path starts with:
no
yes
/notifications/hub  
 	
ACL2
Path starts with:
no
no
/notifications/hub/negotiate  
 	
ACL3
Path starts with:
no
no
/notifications/hub  
 	
ACL4
Path starts with:
no
yes
/notifications/hub/negotiate

ACL5
Path starts with:
no
no
/admin
```

**ACTIONS - 2 - VaultWarden**

```
Use Backend
See below
ACL1  
backend: VaultWarden
 	
Use Backend
See below
ACL2  
backend: VaultWarden
 	
Use Backend
See below
ACL3  
backend: VaultWarden-Notifications
 	
Use Backend
See below
ACL4
backend: VaultWarden-Notifications

http-request deny
See below
ACL5
```

**Updates**

```
在 30/07 以上更新 - 我在第一个配置之后意识到，因为 ACL1-4 有`Not`，他们正在将任何东西与他们的行动相匹配。所以 BlahBlahMcGee.FQDN.com 正在通过。这不是故意的，所以上面添加了 ACL5 来解决这个问题，它还消除了对默认后端的需要。
30/07 再次更新 - ^ 是的，没用。这一切都源于 HaProxy 不允许在 ACL 中使用`AND`。叹。现在有了上面的内容，您就可以为根域配置一个前端。这有一个否认本身，以及任何未指定的内容。因此，如果您要通过多个其他子域，则需要将它们全部添加到 ACL01 下。现在一切正常！
```

**Important Notes**

```
1) 您必须使域前端与允许列表中的任何其他子域保持同步
2) 在域前端，ACL01 必须位于 Actions 表的顶部 - 或至少高于 ACL00
3) ACL 名称的重复使用是有意的。不，我没有打错它们。 ACL00、ACL01 等
```

**OPTIONAL**

```
上述 ACL5 拒绝访问 /admin 门户。我不是特别喜欢没有任何形式的 2FA 且只有密码的管理门户。因此，当我不使用它时，我只是拒绝访问。如果我需要它，请取消阻止，完成所需的工作并重新阻止。
```

完成！ - 去测试！

This in turn will add the equivilent of below to your config (note this is an extract for example).

```
acl			ACL00	var(txn.txnhost) -m str -i VAULTWARDEN.MYDOMAIN.COM
acl			ACL00	var(txn.txnpath) -m beg -i /big-ass-randomised-test-that-really-no-one-is-ever-going-to-type-DONT-USE-THIS-LINE-THOUGH-make-your-own-up
acl			ACL01	var(txn.txnhost) -m str -i EXAMPLE-OTHER-SUB-DOMAIN-1.MYDOMAIN.COM
acl			ACL01	var(txn.txnhost) -m str -i EXAMPLE-OTHER-SUB-DOMAIN-2.MYDOMAIN.COM
acl			ACL1	var(txn.txnpath) -m beg -i /notifications/hub
acl			ACL2	var(txn.txnpath) -m beg -i /notifications/hub/negotiate
acl			ACL3	var(txn.txnpath) -m beg -i /notifications/hub
acl			ACL4	var(txn.txnpath) -m beg -i /notifications/hub/negotiate
acl			ACL5	var(txn.txnpath) -m beg -i /admin

http-request allow  if  ACL01 
http-request deny   if  !ACL00 
http-request deny   if  !ACL5 
http-request deny   if  ACL5 
use_backend VaultWarden_ipvANY  if  !ACL1 
use_backend VaultWarden_ipvANY  if  ACL2 
use_backend VaultWarden-Notifications_ipvANY  if  ACL3 
use_backend VaultWarden-Notifications_ipvANY  if  !ACL4 
```

为了进行测试，如果您在浏览器中导航到 /notifications/hub，那么您应该会看到一个页面，上面写着`WebSocket 协议错误：无法解析 WebSocket 密钥。`……这意味着它可以正常工作！ - 所有其他子页面都应该出现 Rocket 错误。
