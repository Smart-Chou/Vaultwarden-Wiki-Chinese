vaultwarden 目前不提供独立的二进制文件作为单独的下载，但对于具有基于 Alpine 的 Docker 映像可用的平台(当前为 x86-64 和 ARMv7)，您可以从官方 Docker 映像中提取独立的、静态链接的二进制文件。每个 Docker 映像还包括一个匹配的 Web Vault 构建(与平台无关)。

## 在安装了 Docker 的情况下提取二进制文件

假设您要为正在运行的平台提取二进制文件：
```
docker pull vaultwarden/server:alpine
docker create --name vw vaultwarden/server:alpine
docker cp vw:/vaultwarden .
docker cp vw:/web-vault .
docker rm vw
```

如果你想要不同平台的二进制文件(例如，你只在 x86-64 机器上安装了 Docker，但你想在 Raspberry Pi 上运行 vaultwarden)，请将 `--platform` 选项添加到 `docker pull`命令：
```
docker pull --platform linux/arm/v7 vaultwarden/server:alpine
# 像上面一样运行剩余的命令。
# 请注意，`docker create` 命令可能会打印如下消息：
# 警告：请求的镜像平台 (linux/arm/v7) 与检测到的主机平台 (linux/amd64) 不匹配
# 并且没有请求特定的平台
# 这是意料之中的，无需担心。
```

## Extracting binaries without Docker installed

如果你不能或不想安装 Docker，你可以使用 [docker-image-extract](https://github.com/jjlin/docker-image-extract) 脚本来拉取和提取 Docker 镜像.例如，要拉取和提取 x86-64 镜像：
```
$ mkdir vw-image
$ cd vw-image
$ wget https://raw.githubusercontent.com/jjlin/docker-image-extract/main/docker-image-extract
$ chmod +x docker-image-extract
$ ./docker-image-extract vaultwarden/server:alpine
Getting API token...
Getting image manifest for vaultwarden/server:alpine...
Downloading layer 801bfaa63ef2094d770c809815b9e2b9c1194728e5e754ef7bc764030e140cea...
Extracting layer...
Downloading layer c6d331ed95271d8005dea195449ab4ef943017dc97ab134a4426faf441ae4fa6...
Extracting layer...
Downloading layer bfd9ec32f740ca8c86ccde057595d29a31eb093aafd7619fcdd4b956c7bf95e3...
Extracting layer...
Downloading layer e9bfb5d92e4629b1dcb4a13a470c90f51b9edde4e184d8520afc589728b8b675...
Extracting layer...
Downloading layer 5757963c858ce72bc4a1874f4971d326d21d2a844f03063a3c99e312150adf95...
Extracting layer...
Downloading layer f705bf64e4315fea1830cc137d1deda194e825da03bd7822e41ac52457bc83e7...
Extracting layer...
Downloading layer 909b5deb38cbce9f83598918bf7f38b7c2194d385456cf7ef15eff47f8a63108...
Extracting layer...
Downloading layer 8516f4cd818630cd60fa18254b072f8d9c3748bdb56f6e2527dc1c204e8e017c...
Extracting layer...
Image contents extracted into ./output.
$ ls -ld output/{vaultwarden,web-vault}
-rwx------ 1 user user 22054608 Feb  6 21:46 output/vaultwarden
drwx------ 8 user user     4096 Feb  6 21:46 output/web-vault/
```

如果您想要 ARMv7 映像，您目前必须通过摘要下载它。

转到 https://hub.docker.com/r/vaultwarden/server/tags?name=alpine 并找到 `alpine` 标签的条目。
单击`linux/arm/v7`映像的部分摘要：
![#](https://cdn.jsdelivr.net/gh/Smart-Chou/webphoto@latest/note/Hsz8vJ4.png)

这应该会将您带到一个显示完整摘要的页面：

![#](https://cdn.jsdelivr.net/gh/Smart-Chou/webphoto@latest/note/T5WdwtS.png)

复制完整摘要，并将上面的 `docker-image-extract vaultwarden/server:alpine` 命令替换为
`docker-image-extract Vaultwarden/server:<full_digest>`。

例如：

```
$ ./docker-image-extract vaultwarden/server:sha256:ef129de113bec3409b6370c37a6e5573a1dacc051a3aae2a8a3339323ae63623
```