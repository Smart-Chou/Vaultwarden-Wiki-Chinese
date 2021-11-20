`vaultwarden` 提供单个 Docker 镜像 ([`vaultwarden/server`](https://hub.docker.com/r/vaultwarden/server))，统一支持 SQLite、MySQL 和 PostgreSQL 数据库后端，从版本开始1.17.0。在该版本之前，每个数据库后端都有单独的镜像(请参阅 [历史镜像](#历史镜像))。

`vaultwarden/server` 镜像也是多架构的，这意味着它在一个镜像名称下支持多个 CPU 架构。假设您正在运行一种受支持的架构，只需拉动 `vaultwarden/server` 应该会自动为您的环境生成适当的特定于架构的映像，ARMv6 板可能除外，例如 Raspberry Pi 1 和 Zero(请参阅 [moby /moby#41017](https://github.com/moby/moby/issues/41017))。运行 Docker 20.10.0 及更高版本的 ARMv6 用户可以像往常一样简单地拉取 `vaultwarden/server` 多架构映像。运行早期 Docker 版本的 ARMv6 用户必须在镜像标签中指定 `arm32v6`，例如`最新的arm32v6`。

SQLite 后端是使用/测试最广泛的后端，建议大多数用户使用，除非有特定需要使用不同的数据库后端。

### 镜像标签

`Vaultwarden/server` 镜像有几个标签，每个标签代表镜像的某些变体或属性(例如，特定版本)。

- `latest` -- 跟踪最新发布的版本(即用版本号标记)。建议大多数用户使用此标签，因为它通常是最稳定的。

- `testing` -- 跟踪源存储库的最新提交。建议希望尽早访问最新功能、增强功能或错误修复的用户使用此标签。测试版一般比较稳定，但偶尔会出现问题。

- `x.y.z`(例如，`1.16.0`)——代表一个特定的发布版本。

- `alpine` -- 除了少数例外，这个镜像在功能上与 `latest` 相同，但基于 Alpine 而不是基于 Debian，从而导致镜像更薄。因此，`latest`与`alpine`主要是偏好问题。但是，请注意，`alpine` 标签目前仅支持 `amd64` 和 `arm32v7` 架构，并且仅支持 SQLite 和 PostgreSQL 数据库后端。

- `x.y.z-alpine`(例如，`1.16.0-alpine`)——类似于`alpine`，但代表一个特定的发布版本。

- `latest-arm32v6` -- 与 `latest` 相同，但明确表示 `arm32v6` 镜像。当运行早于 20.10.0 的 Docker 版本时，这是 ARMv6 板(例如 Raspberry Pi 1 和 Zero)用户所必需的。这些旧版本存在一个问题 ([moby/moby#41017](https://github.com/moby/moby/issues/41017))，导致 `arm32v7` 镜像被拉取，这将不起作用。该问题已在 Docker 20.10.0 及更高版本中修复，因此运行这些版本的用户可以像往常一样使用`最新`。

- `testing-arm32v6` -- 与 `testing` 相同，但明确表示 `arm32v6` 镜像。

- `x.y.z-arm32v6`(例如，`1.16.0-arm32v6`)——类似于`latest-arm32v6`，但代表一个特定的发布版本。

## 镜像更新

有时，上游 Bitwarden 项目(即 Bitwarden Inc.)对需要对服务器实现进行匹配更改的客户端进行向后不兼容的更改。 Vaultwarden 通常会及时推出新版本来处理这些更改。

但是，由于上游控制客户端的发布，并且移动应用程序和浏览器扩展通常会自行自动更新，因此 vaultwarden 用户必须及时了解最新的 vaultwarden 版本。否则，不兼容的客户端和服务器版本可能会导致突然损坏或不当行为。

网络保险库是唯一的例外；由于它与 vaultwarden 映像捆绑在一起，因此 Web Vault 版本始终与 vaultwarden 服务器版本正确匹配。如果您只使用 Web Vault 作为客户端(不太可能)，那么您无需担心这些兼容性问题。

## 历史镜像

在 1.17.0 版中添加 multidb 支持之前，MySQL 和 PostgreSQL 支持仅包含在单独的特定于数据库的映像中。您仍然可以在 Docker Hub 中找到这些，并且它们现在仍在更新。但是，特定于数据库的映像将在将来被删除，因此您应该过渡到使用统一的`Vaultwarden/server`映像。

- [`bitwardenrs/server-mysql`](https://hub.docker.com/r/bitwardenrs/server-mysql) - 基于 Debian 的 `vaultwarden` 镜像，仅支持 MySQL(不支持 SQLite 或 PostgreSQL)。
- [`bitwardenrs/server-postgresql`](https://hub.docker.com/r/bitwardenrs/server-postgresql) - 基于 Debian 的 `vaultwarden` 镜像，仅支持 PostgreSQL(不支持 SQLite 或 MySQL)。

## 历史标签

在 1.16.0 版本中添加多架构镜像支持之前，所有特定于架构的镜像都有单独的特定于架构的标签。截至 2021 年 1 月 14 日，这些标签已被删除，因为许多用户仍然由于遵循过时的教程或未阅读发行说明而最终删除了这些旧标签。

- `raspberry` - Armv7hf 映像应该在 Raspberry Pi 2 或更新版本上运行，并且可能在任何其他兼容的板上运行。此映像不会在 Raspberry Pi 1 或 Raspberry Pi Zero 上运行，因为它们使用 armv6 CPU。

- `armv6` - Raspberry Pi 1 和 Raspberry Pi Zero 的 Armv6 映像。

- `aarch64` - Aarch64 映像，应该在 ARMv8 设备上运行，例如 Raspberry Pi 3 或其他可能基于 ARMv8 的设备。

  **注意** 这还需要在您的设备上安装 aarch64 发行版，因此例如，如果您在 Raspberry Pi 3 上使用 Raspbian，您仍然需要使用 `raspberry` 标签，因为 Raspbian 是一个 `armv7hf` 发行版。

## 报告的兼容性表

如果您在表中没有的硬件上运行映像，请在此处添加您的详细信息。

| Hardware used        | OS           | Docker architecture reported    | Image used          | Status | Notes |
|----------------------|--------------|---------------------------------|---------------------|--------|-------|
| Regular 64bit server | Ubuntu 18.04 | x86_64                          | `vaultwarden/server` | OK     |       |
| O-Droid HC2          | Armbian      | arm7l (arm32)                   | `registry.lollipopcloud.solutions/arm32v7/bitwarden` (see notes) | OK | Unofficial image built from upstream sources ; `vaultwarden/server:raspberry` is the official equivalent image |
| Raspberry Pi Zero W  | Raspbian (4.14.98+) | linux/arm (armv6l)       | `vaultwarden/server:armv6` | OK |     |
| Raspberry Pi Zero W  | Raspbian (4.19.66+) | linux/arm (armv6l)       | `vaultwarden/server:latest` (Multiarch) | OK | Only when using the docker experimental feature 'docker pull --platform=linux/arm/v6'. Otherwise the wrong image will be selected (https://github.com/dani-garcia/vaultwarden/issues/1064) |
| Raspberry Pi 1 B     | Raspbian (4.19.97+) | linux/arm (armv6l)       | `vaultwarden/server:armv6` | OK |     |
| Raspberry Pi 3 B     | Raspbian (4.14.98-v7+) | linux/arm (armv7l)    | `vaultwarden/server:raspberry` | OK |     |
| Raspberry Pi 4       | Raspbian (4.19.118-v7l+) | linux/arm (armv7l)  | `vaultwarden/server:raspberry` | OK | 4go version, rev 1.1   |
| Synology             | DSM (DSM 6.2.1-23824 Update 6) | Docker-x64-17.05.0-0367 | `vaultwarden/server:latest` | OK |
| Synology             | DSM (DSM 6.2.2-24922 Update 4) | Docker-x64-18.09.0-0506 | `vaultwarden/server:1.13.0-alpine` | OK |
| Regular 64bit server | Unraid 6.8.0 | 19.03.5                         | `vaultwarden/server:latest` | OK |     |
