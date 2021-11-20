默认情况下，Vaultwarden 仅记录到 [标准输出](https://en.wikipedia.org/wiki/Standard_streams#Standard_output_(stdout)) (stdout)。您还可以将其配置为记录到文件。

## 记录到文件

从版本 1.5.0 开始支持记录到文件。您可以使用`LOG_FILE`环境变量指定日志文件的路径：

```sh
docker run -d --name vaultwarden \
...
  -e LOG_FILE=/data/vaultwarden.log \
...
```

设置此环境变量后，日志消息将记录到标准输出和日志文件中。如果您在 Docker 中运行，您很可能希望使用从 Docker 主机挂载的文件路径(例如 `data` 文件夹)；否则，如果容器重新启动或删除，您的日志文件将丢失(或至少很难找到)。

##更改日志级别

要减少日志消息的数量，您可以将日志级别设置为`警告`(默认为`信息`)。 [日志级别](https://docs.rs/log/0.4.7/log/enum.Level.html#variants) 可以通过环境变量`LOG_LEVEL`进行调整，同时还可以设置`EXTENDED_LOGGING=true`。注意：使用日志级别`警告`或`错误`仍然允许 [Fail2Ban](Fail2Ban-Setup) 正常工作。

`LOG_LEVEL` 选项是：`trace`、`debug`、`info`、`warn`、`error`或`off`。

```sh
docker run -d --name vaultwarden \
...
  -e LOG_LEVEL=warn -e EXTENDED_LOGGING=true \
...
```

## 查看日志

如果在 Docker 中运行：`docker logs <container-name>`

如果通过 `systemd` 运行：`journalctl -u vaultwarden.service`(或任何你的服务名称)

否则，检查标准输出的重定向位置，或设置`LOG_FILE`环境变量并查看该文件。