默认情况下，`vaultwarden/server` 使用 root 用户在容器内运行服务。如果您希望以非 root 用户身份运行容器，您需要设置以下几点：

1. 确保安装在容器内的目录可由用户写入。例如，如果您决定以 `nobody` 身份运行，则该目录需要 id 为 65534 的用户可写。有关在容器内指定用户的其他方法，请参阅 [docker 文档](https://docs.docker.com/engine/reference/run/#user)，在我们的示例中，我们将使用 `nobody`。

```bash
# 在主机上创建目录，将其更改为您首选的路径
sudo mkdir /vw-data

# 使用用户ID设置所有者。
# 请注意，所有权必须与 /etc/passwd *inside* 容器中的用户匹配，而不是在您的主机上
sudo chown 65534 /vw-data

# 赋予所有者对该文件夹的完全权限
sudo chmod u+rwx /vw-data
```

2. 使用适当的参数启动容器。定义用户并确保从设置为`1024`或更高的端口开始

```bash
docker run -d \
  --name vaultwarden \
  --user nobody \
  -e ROCKET_PORT=1024 \
  -v /vw-data/:/data/ \
  -p 80:1024 \
  vaultwarden/server:latest
```

请注意，端口映射(`-p 80:1024`)反映了 `ROCKET_PORT` 设置。 