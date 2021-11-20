设置 Fail2ban 将防止攻击者对您的保险库登录进行暴力破解。如果您的实例是公开可用的，这一点尤其重要。

## 目录

- [先决条件](#先决条件)
- [安装](#安装)
- [Debian / Ubuntu / 树莓派操作系统](#debian-ubuntu-raspberry-pi-os)
- [Fedora / Centos](#fedora-centos)
- [Synology DSM](#synology-dsm)
- [网络保险库的设置](#网络保险库的设置)
- [过滤器](#过滤器)
- [Jail](#Jail)
- [设置管理页面](#设置管理页面)
- [筛选](#筛选)
- [Jail](#jail)
- [测试Fail2Ban](#测试-fail2ban)
- [SELinux 问题](#SELinux-问题)

## 先决条件

- 文件名位于每个代码块的顶部。
- 从 1.5.0 版开始，Vaultwarden 支持记录到文件。请设置：[登录](Logging)
- 尝试使用虚假帐户登录 Web Vault 并检查以下格式的日志文件

```
[YYYY-MM-DD hh:mm:ss][vaultwarden::api::identity][ERROR] 用户名或密码不正确。再试一次。 IP：XXX.XXX.XXX.XXX。用户名：email@domain.com。
```

## 安装

### Debian / Ubuntu / Raspberry Pi OS

```bash
sudo apt-get install fail2ban -y
```

### Fedora / Centos

EPEL repository is necessary (CentOS 7)  

```bash
sudo yum install epel-release
sudo yum install fail2ban -y
```

### Synology DSM

对于 Synology，由于各种原因需要做更多的工作。完整的解决方案是用 Docker Compose [那里](https://github.com/sosandroid/docker-fail2ban-synology) 推送的。主要问题是：

1. 内嵌的IP禁令系统对Docker的容器不起作用
2. 内嵌的iptables不支持`REJECT`块类型
3. Docker GUI 不允许一些高级设置
4. 修改系统配置不防升级

因此，我们将在 Docker 容器中使用 Fail2ban。 [Crazy-max/docker-fail2ban](https://github.com/crazy-max/docker-fail2ban) 提供了很好的解决方案，Synology 的 Docker GUI 将被忽略。从命令行到 SSH，这里是步骤。作为惯例，`volumeX` 将适应您的 Synology 配置。

0. 获得root权限

```bash
sudo -i
```

1. 创建永久文件夹

```bash
mkdir -p /volumeX/docker/fail2ban/action.d/
mkdir -p /volumeX/docker/fail2ban/jail.d/
mkdir -p /volumeX/docker/fail2ban/filter.d/
```

2. 用`DROP`块类型替换`REJECT`

```INI
# /volumeX/docker/fail2ban/action.d/iptables-common.local

[Init]
blocktype = DROP
[Init?family=inet6]
blocktype = DROP
```

3. 创建`docker-compose`文件

```yml
# /volumeX/docker/fail2ban/docker-compose.yml

version: '3'
services:
  fail2ban:
    container_name: fail2ban
    restart: always
    image: crazymax/fail2ban:latest
    environment: 
      - TZ=Europe/Paris
      - F2B_DB_PURGE_AGE=30d
      - F2B_LOG_TARGET=/data/fail2ban.log
      - F2B_LOG_LEVEL=INFO
      - F2B_IPTABLES_CHAIN=INPUT

    volumes:
      - /volumeX/docker/fail2ban:/data
      - /volumeX/docker/vw-data:/vaultwarden:ro

    network_mode: "host"

    privileged: true
    cap_add:
      - NET_ADMIN
      - NET_RAW
```

4. 使用命令行启动容器

```bash
cd /volumeX/docker/fail2ban
docker-compose up -d
```

您应该会看到在 Synology 的 Docker GUI 中运行的容器。配置过滤器和Jail后，您必须重新加载

## 网络保险库的设置

按照惯例，`path_f2b` 表示 Fail2ban 工作所需的路径。这取决于您的系统。例如。在 Synology 上，我们谈论的是 `/volumeX/docker/fail2ban/` 而在其他一些系统上，我们谈论的是 `/etc/fail2ban/`

### 筛选

创建并填写以下文件

```INI
# path_f2b/filter.d/vaultwarden.local

[INCLUDES]
before = common.conf

[Definition]
failregex = ^.*Username or password is incorrect\. Try again\. IP: <ADDR>\. Username:.*$
ignoreregex =
```

**提示：** 如果您在 `fail2ban.log` 中收到以下错误消息 (CentOS 7, Fail2Ban v0.9.7)  
`fail2ban.filter         [5291]: ERROR   No 'host' group in '^.*Username or password is incorrect\. Try again\. IP: <ADDR>\. Username:.*$'`  
请在 `vaultwarden.local` 中使用 `<HOST>` 而不是 `<ADDR>`

**提示：**如果您在 vaultwarden.log 中看到 127.0.0.1 作为登录失败的 IP 地址，那么您可能使用的是反向代理并且 fail2ban 将无法正常工作：
```
[YYYY-MM-DD hh:mm:ss][vaultwarden::api::identity][ERROR] Username or password is incorrect. Try again. IP: 127.0.0.1. Username: email@example.com.
```
要解决此问题，请通过 X-Real-IP 标头将真正的远程地址转发给 Vaultwarden。如何执行此操作取决于您使用的代理。例如，在 Caddy 2.x 中，当您定义反向代理时，请定义 `header_up X-Real-IP {remote_host}`。有关更多信息，请参阅 [代理示例](Proxy-examples)。

### Jail

创建并填写以下文件

```INI
# path_f2b/jail.d/vaultwarden.local

[vaultwarden]
enabled = true
port = 80,443,8081
filter = vaultwarden
banaction = %(banaction_allports)s
logpath = /path/to/vaultwarden.log
maxretry = 3
bantime = 14400
findtime = 14400
```

注意：Docker 使用 FORWARD 链而不是默认的 INPUT 链。因此，在使用 Docker 时，将`banaction`行替换为以下`action`：

```INI
action = iptables-allports[name=vaultwarden, chain=FORWARD]
```

**笔记**：
如果在 Docker 容器之前使用反向代理，请不要使用它。如果使用代理，如 apache2 或 nginx，请使用代理的端口，不要使用 `chain=FORWARD`，只有在使用 Docker 时才**无**代理！

**注意上面的注释**：
至少对于使用 caddy 作为反向代理在 Docker (CentOS 7) 上运行来说不是这样。 `chain=FORWARD` 绝对没问题，可以使用 caddy 作为反向代理。

重新加载 fail2ban 以使更改生效：

```bash
sudo systemctl reload fail2ban
```

随意更改您认为合适的选项。

## 设置管理页面
如果您通过设置`ADMIN_TOKEN` 环境变量启用了管理控制台，则可以防止攻击者使用Fail2Ban 暴力破解管理令牌。该过程与 Web Vault 的过程相同。

### 筛选

创建并填写以下文件
```INI
# path_f2b/filter.d/vaultwarden-admin.local

[INCLUDES]
before = common.conf

[Definition]
failregex = ^.*Invalid admin token\. IP: <ADDR>.*$
ignoreregex =
```

**提示：** 如果您在 `fail2ban.log` 中收到以下错误消息
`错误NOK：(`^.*无效的管理令牌\\。IP：<ADDR>.*$'`中没有`主机`组)`
请在 `vaultwarden-admin.local` 中使用 `<HOST>` 而不是 `<ADDR>`

### Jail

创建并填写以下文件

```INI
# path_f2b/jail.d/vaultwarden-admin.local

[vaultwarden-admin]
enabled = true
port = 80,443
filter = vaultwarden-admin
banaction = %(banaction_allports)s
logpath = /path/to/vaultwarden.log
maxretry = 3
bantime = 14400
findtime = 14400
```

注意：Docker 使用 FORWARD 链而不是默认的 INPUT 链。因此，在使用 Docker 时，将`banaction`行替换为以下`action`：

```INI
action = iptables-allports[name=vaultwarden-admin, chain=FORWARD]
```

Reload fail2ban for changes to take effect:

```bash
sudo systemctl reload fail2ban
```

## 测试 Fail2Ban

现在只需尝试使用任何电子邮件登录Vaultwarden(它不必是有效的电子邮件，只需一种电子邮件格式)
如果它工作正常并且您的 IP 被禁止，您可以通过运行以下命令取消禁止 IP：

```bash
# With Docker
sudo docker exec -t fail2ban fail2ban-client set vaultwarden unbanip XX.XX.XX.XX
# Without Docker
sudo fail2ban-client set vaultwarden unbanip XX.XX.XX.XX
```

如果 Fail2Ban 似乎不起作用，请验证 Vaultwarden 日志文件的路径是否正确。对于 Docker：如果未生成和/或更新指定的日志文件，请确保将 `EXTENDED_LOGGING` 环境变量设置为 true(这是默认值)并且日志文件的路径是 Docker 内部的路径(当您使用 `/bw-data/:/data/` 日志文件应该在 `/data/...` 中以位于容器之外)。

还要验证 Docker 容器的时区是否与主机的时区匹配。通过将日志文件中显示的时间与主机操作系统时间进行比较来检查这一点。如果它们不同，则有多种方法可以解决此问题。一种选择是使用选项 `-e "TZ=<timezone>"` 启动 Docker。有效时区列表是 [here](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)(例如`-e "TZ=Australia/Melbourne"`)

如果您使用 podman 而不是 Docker，似乎通过 `-e "TZ=<timezone>"` 设置时区不起作用。这可以通过遵循本指南来解决(使用 alpine 镜像时)：[https://wiki.alpinelinux.org/wiki/Setting_the_timezone](https://wiki.alpinelinux.org/wiki/Setting_the_timezone)。

## SELinux 问题

当您使用 SELinux 时，SELinux 可能会阻止 fail2ban 读取日志。如果是这样，请执行以下步骤：
`sudo tail /var/log/audit/audit.log`。在那里你应该看到一些类似的东西(当然，实际的审计 ID 会因你的情况而异)：

```
type=AVC msg=audit(1571777936.719:2193): avc:  denied  { search } for  pid=5853 comm="fail2ban-server" name="containers" dev="dm-0" ino=1144588 scontext=system_u:system_r:fail2ban_t:s0 tcontext=unconfined_u:object_r:container_var_lib_t:s0 tclass=dir permissive=0
```

要真正找出原因，您可以使用`grep 'type=AVC msg=audit(1571777936.719:2193)' /var/log/audit/audit.log |审计2为什么`。 `audit2allow -a` 将提供有关如何创建模块并允许 fail2ban 访问这些日志的具体说明。按照这些步骤，你就大功告成了！ fail2ban 现在应该可以正常工作了。