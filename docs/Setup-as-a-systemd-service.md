这些说明要求您具有 [预建二进制文件](Pre-built-binaries)。如果您生成了 docker 镜像，您可能需要查看 [*使用 systemd docker 运行](Running-with-systemd-docker)

## 设置

使 Vaultwarden 在系统启动时启动并使用 systemd 的其他功能(例如隔离、日志记录等)需要一个 `.service` 文件。以下是一个可用的起点：

```ini
[Unit]
Description=Bitwarden Server (Rust Edition)
Documentation=https://github.com/dani-garcia/vaultwarden
# 如果你使用像 mariadb、mysql 或 postgresql 这样的数据库，
# 你必须像下面这样添加它们并取消它们的注释
# 通过删除它之前的 `#`。这可确保您的
# 数据库服务器在 Vaultwarden 之前("After")启动并且已经
# 在启动 Vaultwarden 之前成功启动("Requires")。

# 只有sqlite
After=network.target

# MariaDB
# After=network.target mariadb.service
# Requires=mariadb.service

# Mysql
# After=network.target mysqld.service
# Requires=mysqld.service

# PostgreSQL
# After=network.target postgresql.service
# Requires=postgresql.service


[Service]
# 用户/组 Vaultwarden 在其下运行。工作目录(见下文)应该允许对该用户/组的读写访问
User=vaultwarden
Group=vaultwarden
# 用于配置的.env 文件的位置
EnvironmentFile=/etc/vaultwarden.env
# 编译后的二进制文件的位置
ExecStart=/usr/bin/Vaultwarden
# 设置合理的连接和进程限制
LimitNOFILE=1048576
LimitNPROC=64
# 将 Vaultwarden 与系统的其余部分隔离
PrivateTmp=true
PrivateDevices=true
ProtectHome=true
ProtectSystem=strict
# 只允许写入以下目录，并设置为工作目录(用户和密码数据存放在这里)
WorkingDirectory=/var/lib/vaultwarden
ReadWriteDirectories=/var/lib/vaultwarden
# 允许Vaultwarden绑定0-1024范围内的端口
AmbientCapabilities=CAP_NET_BIND_SERVICE

[Install]
WantedBy=multi-user.target
```

更改所有路径以匹配您的安装(`WorkingDirectory` 和 `ReadWriteDirectory` 应该相同)，
将此文件命名为`Vaultwarden.service`并将其放入`/etc/systemd/system`。

如果您必须更改现有的 systemd 文件(由您安装的软件包提供给您)，您可以使用
```
$ sudo systemctl edit vaultwarden.service
```

要让 systemd 知道您的新文件或您所做的任何更改，请运行
```
$ sudo systemctl daemon-reload
```

## 用法

要启动这个`服务`，运行
```
$ sudo systemctl start vaultwarden.service
```

要启用自动启动，请运行
```
$ sudo systemctl enable vaultwarden.service
```

以同样的方式，您可以`停止`、`重新启动`和`禁用`服务。

### 更新 Vaultwarden

编译新版本的 vaultwarden 后，您可以复制编译后的(新)二进制文件并替换现有(旧)二进制文件，然后重新启动服务：
```
$ sudo systemctl restart vaultwarden.service
```

### 卸载Vaultwarden

在做任何其他事情之前，您应该停止并禁用该服务：
```
$ sudo systemctl disable --now vaultwarden.service
```

然后您可以删除二进制文件、`.env` 文件、web-vault 文件夹(如果已安装)和用户数据(如果需要)。记住还要删除专门创建的用户、组和防火墙规则(如果需要)和 systemd 文件。

删除 systemd 文件后，您应该通过以下方式让 systemd 知道它：
```
$ sudo systemctl daemon-reload
```

### 日志和状态视图

如果要查看日志输出，请运行
```
$ journalctl -u vaultwarden.service
```

或者要查看更简洁的服务状态，请运行
```
$ systemctl status vaultwarden.service
```

## 故障排除

### 旧 systemd 版本的沙盒选项

在 RHEL 7(和 debian 8)中，使用的 systemd 不支持某些使用的隔离选项。 ([#445](https://github.com/dani-garcia/vaultwarden/issues/445),[#363](https://github.com/dani-garcia/vaultwarden/issues/363))
这可能会导致以下错误之一：
```
Failed at step NAMESPACE spawning /home/vaultwarden/vaultwarden: Permission denied
```

or 
```
Failed to parse protect system value
```

要解决此问题，您可以通过在包含
`PrivateTmp`、`PrivateDevices`、`ProtectHome`、`ProtectSystem` 和`ReadWriteDirectories`。虽然注释掉所有这些可能会起作用，但不建议这样做，因为这些是很好的安全措施。要查看您的 systemd 支持哪些选项，请查看以下输出
```
$ systemctl --version
```

确定您的 systemd 版本并与 [systemd/NEWS.md](https://github.com/systemd/systemd/blob/master/NEWS) 进行比较。

编辑完你的 `.service` 文件后，不要忘记
```
$ sudo systemctl daemon-reload
```

在(重新)开始您的服务之前。

### 服务无法启动

systemd 日志(`journalctl -eu vaultwarden.service`)中显示以下错误：

```
Feb 18 05:29:10 staging-bitwarden systemd[1]: Started Bitwarden Server (Rust Edition).
Feb 18 05:29:10 staging-bitwarden systemd[49506]: vaultwarden.service: Failed to execute command: Resource temporarily unavailable
Feb 18 05:29:10 staging-bitwarden systemd[49506]: vaultwarden.service: Failed at step EXEC spawning /usr/bin/vaultwarden: Resource temporarily unavailable
Feb 18 05:29:10 staging-bitwarden systemd[1]: vaultwarden.service: Main process exited, code=exited, status=203/EXEC
Feb 18 05:29:10 staging-bitwarden systemd[1]: vaultwarden.service: Failed with result 'exit-code'.
```

众所周知，当 vaultwarden 在容器(LXC 等)或本地运行时会发生这种情况。服务文件中的参数`LimitNPROC=64`阻止服务启动。注释掉该特定参数会导致服务正确启动。

**注意**：systemd 覆盖文件将不起作用，该行必须被注释掉/删除。最简单的方法是通过
```
# systemctl edit --full vaultwarden.service
```
然后重新加载守护进程并重新启动。

### 环境变量未加载

请注意，systemd 不支持与 `EnvironmentFile` 文件中的变量在同一行中的注释。在这个 `.env` 文件示例中，变量`WEBSOCKET_ENABLED`不会被加载。
```
ROCKET_PORT=XXXX
WEBSOCKET_ENABLED=true # enable websocket
```

来源：[#1607](/dani-garcia/vaultwarden/issues/1607)

## 更多信息

有关 .service 文件的更多信息，请参阅 [systemd.service](https://www.freedesktop.org/software/systemd/man/systemd.service.html) 和[systemd.exec](https://www.freedesktop.org/software/systemd/man/systemd.exec.html)（用于安全性配置）的联机帮助页。