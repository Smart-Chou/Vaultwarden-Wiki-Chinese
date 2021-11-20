更新很简单，您只需确保保留已安装的卷。如果你在例子中使用了绑定挂载的路径 [[here|Starting-a-Container]]，你只需要`pull`最新的镜像，`stop`和`rm`当前容器，然后启动一个新的一个和以前一样的方式：

```sh
# 拉取最新版本
docker pull Vaultwarden/server:latest

# 停止并移除旧容器
docker stop vaultwarden
docker rm vaultwarden

# 启动一个挂载数据的新容器
docker run -d --name vaultwarden -v /vw-data/:/data/ -p 80:80 vaultwarden/server:latest
```
然后访问 [http://localhost:80](http://localhost:80)

如果您没有为持久数据绑定挂载卷，则需要一个中间步骤，使用中间容器保存数据：

```sh
# 拉取最新版本
docker pull Vaultwarden/server:latest

# 创建中间容器来保存数据
docker run --volumes-from vaultwarden --name vaultwarden_data busybox true

# 停止并移除旧容器
docker stop vaultwarden
docker rm vaultwarden

# 启动一个挂载数据的新容器
docker run -d --volumes-from vaultwarden_data --name vaultwarden -p 80:80 vaultwarden/server:latest

# 可选地删除中间容器
docker rm vaultwarden_data

# 或者，您可以保留数据容器以备将来更新，在这种情况下，您可以跳过最后一步。
```

您还可以使用 [Watchtower](https://containrrr.dev/watchtower/) 之类的工具来自动执行更新过程。 Watchtower 可以定期检查 Docker 镜像的更新，拉取更新的镜像，并使用更新的镜像重新创建容器。

## 使用 docker-compose 时更新

```sh
docker-compose stop
docker-compose pull
docker-compose start
```

## 使用 systemd 服务时更新（在本例中为 Debian/Raspbian）

```sh
sudo systemctl restart vaultwarden.service
sudo docker system prune -f
# WARNING 这可能会删除与 Vaultwarden 无关的已停止或未使用的容器等
# 小心，看看你需要哪些容器

docker ps -a
#显示停止的容器

# 警告！这将删除：
#   - 所有停止的#容器
#   - 至少一个容器未使用的所有网络
#   - 所有悬空镜像
#   - 所有悬空构建缓存
#你可以列出docker镜像
docker images
#在那里你会看到所有未使用的镜像
#
```
restart 命令将停止容器，拉取最新的镜像，再次运行容器。
prune 命令将删除现在的旧容器（-f 代表：不要求确认）。

如果需要，将这些放入 cronjob 中（时间可以更改）：
```
$ sudo crontab -e
0 2 * * * sudo systemctl restart vaultwarden.service

0 3 * * * sudo /usr/bin/docker system prune -f
```
使用命令

`which docker`

如果`/usr/bin/docker` 不是docker的正确路径