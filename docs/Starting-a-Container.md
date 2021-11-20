请注意，`docker run` 命令的名称有点误导，因为它会创建一个容器，而不是仅仅启动它，这会导致在停止容器而不删除容器后使用 `docker run` 时发生冲突。有关简单的开始，请参见下文。

## 创建容器

持久化数据存储在容器内部的/data下，因此使用Docker进行持久化部署的唯一要求是在路径上挂载持久化卷：

```sh
# 使用 Docker：
docker run -d --name vaultwarden -v /vw-data/:/data/ -p 80:80 vaultwarden/server:latest
# 使用 Podman 作为非 root：
podman run -d --name vaultwarden -v /vw-data/:/data/:Z -e ROCKET_PORT=8080 -p 8080:8080 vaultwarden/server:latest
# 使用 Podman 作为 root：
sudo podman run -d --name vaultwarden -v vw-data:/data/:Z -p 80:80 vaultwarden/server:latest
```

这将保留 `/vw-data/` 下的任何持久数据，您可以将路径调整为适合您的任何内容。

该服务将在主机端口 80 或 8080 上公开。

对于非 x86 硬件或运行特定版本，您可以[使用哪个容器镜像](Which-Container-image-to-use)。

如果您的 docker/vaultwarden 在具有固定 IP 的设备上运行，您可以将主机端口绑定到该特定 IP，从而防止将主机端口暴露给整个世界或网络。在host-port和container-port前面添加IP地址(例如192.168.0.2)如下：

```
# 使用 Docker:
docker run -d --name vaultwarden -v /vw-data/:/data/ -p 192.168.0.2:80:80 vaultwarden/server:latest
```

## 启动容器

如果容器已被`docker stop vaultwarden`、重启或任何其他原因停止，您可以使用以下命令重新启动它
```
docker start vaultwarden
```

## 自定义容器启动

如果您想在容器启动时运行自定义启动脚本，您可以将单个脚本作为 `/etc/vaultwarden.sh` 和/或脚本目录安装到容器中作为 `/etc/vaultwarden。 d`。在后一种情况下，只运行带有`.sh`扩展名的文件，因此带有其他扩展名的文件(例如，数据/配置文件)可以驻留在同一个目录中。 (请参阅 [start.sh](https://github.com/dani-garcia/vaultwarden/blob/master/docker/start.sh)了解其工作原理的详细信息。)

自定义启动脚本可用于修补 Web Vault 文件或安装附加包、CA 证书等，而无需构建和维护您自己的 Docker 映像。

### 例子

假设您的脚本名为 `init.sh` 并包含以下内容：
```
echo "starting up"
```

您可以像这样在启动时运行脚本：
```
docker run -d --name vaultwarden -v $(pwd)/init.sh:/etc/vaultwarden.sh <other docker args...> vaultwarden/server:latest
```

如果你运行 `docker logs vaultwarden`，你现在应该看到 `starting up` 作为输出的第一行。

请注意，每次容器启动时都会运行 init 脚本(不仅仅是第一次)，因此这些脚本通常应该是幂等的(即，您可以多次运行脚本而不会出现不良/错误行为)。如果您的脚本自然没有此属性，您可以执行以下操作： 
```
if [ ! -e /.init ]; then
  touch /.init

  # run your init steps...
fi
```
