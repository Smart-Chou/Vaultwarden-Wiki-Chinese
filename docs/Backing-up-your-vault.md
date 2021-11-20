## 概述

Vaultwarden 数据应定期备份，最好通过自动化流程(例如，cron 作业)进行备份。理想情况下，至少应远程存储一份副本(例如，云存储或另一台计算机)。避免依赖文件系统或 VM 快照作为备份方法，因为这些是更复杂的操作，可能会出现更多问题，并且在这种情况下恢复对于典型用户来说可能很困难或不可能。在您的备份上添加额外的加密层通常是个好主意(特别是如果您的备份还包括像 [禁用admin token](Disable-admin-token) 之类的配置数据)，但您可能会选择跳过此步骤，如果您' 确信您的主密码(以及您的其他用户的密码，如果有)是强密码。

## 备份数据

默认情况下，Vaultwarden 将其所有数据存储在名为`data`的目录下(与`vaultwarden`可执行文件位于同一目录中)。可以通过设置 [DATA_FOLDER](Changing-persistent-data-location) 环境变量来更改此位置。如果您使用 SQLite 运行 Vaultwarden(这是最常见的设置)，那么 SQL 数据库只是数据文件夹中的一个文件。如果使用 MySQL 或 PostgreSQL 运行，则必须单独转储这些数据——这超出了本文的范围，但网络搜索会找到许多其他涵盖此主题的教程。

当使用默认的 SQLite 后端运行时，Vaultwarden `data` 目录具有以下结构：

```
data
├──attachments # 每个附件都作为一个单独的文件存储在这个目录下。
│ └── <uuid> #(如果没有创建附件，附件目录将不存在。)
│ └── <random_id>
├── config.json # 存储管理页面配置；仅在之前已启用管理页面时才存在。
├── db.sqlite3 # 主 SQLite 数据库文件。
├── db.sqlite3-shm # SQLite 共享内存文件(不总是存在)。
├── db.sqlite3-wal # SQLite 预写日志文件(不总是存在)。
├── icon_cache # 站点图标(favicon)缓存在这个目录下。
│ ├── <域名>.png
│ ├── example.com.png
│ ├── example.net.png
│ └── example.org.png
├── rsa_key.der # `rsa_key.*` 文件用于对认证令牌进行签名。
├── rsa_key.pem
├── rsa_key.pub.der
└── sent # 每个发送附件都作为一个单独的文件存储在这个目录下。
    └── <uuid> #(如果没有创建发送附件，则发送目录将不存在。)
        └── <random_id>
```

使用 MySQL 或 PostgreSQL 后端运行时，目录结构相同，只是没有 SQLite 文件。您仍然需要备份`data`目录中的文件，以及您的 MySQL 或 PostgreSQL 表的转储。

接下来将更详细地讨论每组文件。

### SQLite 数据库文件

_**需要备份。**_

SQLite 数据库文件(`db.sqlite3`)存储几乎所有重要的保管库数据/状态(数据库条目、用户/组织/设备元数据等)，主要的例外是附件，它们作为单独的文件存储在文件系统上.

您通常应该在 SQLite CLI (`sqlite3`) 中使用 `.backup` 命令来备份数据库文件。此命令使用 [Online Backup API](https://www.sqlite.org/backup.html)，其中 SQLite 记录为 [最佳方式](https://www.sqlite.org/howtocorrupt.html#_backup_or_restore_while_a_transaction_is_active ) 备份可能正在使用的数据库文件。如果您可以确保备份运行时数据库不会被使用，您还可以使用其他方法，例如 `.dump` 命令，或者简单地复制所有 SQLite 数据库文件(包括 `-wal` 文件，如果存在)。

一个基本的备份命令看起来像这样，假设你的数据文件夹是 `data`(默认)：

```
sqlite3 data/db.sqlite3 ".backup '/path/to/backups/db-$(date '+%Y%m%d-%H%M').sqlite3'"
```

您还可以使用`VACUUM INTO`，它会压缩空白空间，但需要更多的处理时间：

```
sqlite3 data/db.sqlite3 "VACUUM INTO '/path/to/backups/db-$(date '+%Y%m%d-%H%M').sqlite3'"
```

假设此命令在 2021 年 1 月 1 日下午 12:34(当地时间)运行，这会将您的 SQLite 数据库文件备份到 `/path/to/backups/db-20210101-1234.sqlite3`。

您可以定期通过 cron 作业运行此命令(最好至少每天一次)。如果您通过 Docker 运行，请注意 Docker 映像不包含 `sqlite3` 二进制文件或 `cron` 守护程序，因此您通常会将这些安装在 Docker 主机本身上并在容器外运行 cron 作业。如果你出于某种原因真的想从容器内运行备份，你可以在[容器启动](Starting-a-Container#自定义容器启动)期间安装任何必要的包，或者使用您首选的`vaultwarden/server:<tag>` 镜像作为父镜像创建您自己的自定义 Docker 镜像。

如果您想将备份数据复制到云存储，[rclone](https://rclone.org/) 是与各种云存储系统接口的有用工具。 [restic](https://restic.net/) 是另一个不错的选择，尤其是当您有较大的附件并希望避免在每次备份时重新复制它们时。

### `attachments` 目录

_**需要备份。**_

[文件附件](https://bitwarden.com/help/article/attachments/) 是唯一没有存储在数据库表中的重要数据类，主要是因为它们可以是任意大的，而 SQL 数据库通常不是为了有效地处理大斑点。如果从未创建过文件附件，则该目录将不存在。

### `sends` 目录

_**备份可选。**_

与常规文件附件一样，[发送](https://bitwarden.com/help/article/about-send/) 文件附件不存储在数据库表中。 (但是，发送文本注释存储在数据库中。)

与常规附件不同，发送附件旨在是短暂的。因此，如果您想最小化备份的大小，您可以选择不备份此目录。另一方面，如果在还原过程中维护现有发送的正确功能更重要，那么您应该备份此目录。

如果从未创建过发送附件，则该目录将不存在。

### `config.json` 文件

_**推荐备份。**_

如果您使用管理页面来配置您的 vaultwarden 实例，并且没有以其他方式备份您的配置，那么您可能想要备份此文件，这样您就不必重新确定您的首选配置。

请记住，此文件确实包含一些可能被视为敏感的纯文本数据(管理员令牌、SMTP 凭据等)，因此如果您担心其他人可能能够访问这些数据，请务必加密这些数据(例如，上传到云存储时)。

### `rsa_key*` 文件

_**推荐备份。**_

这些文件用于对当前登录用户的 JWT(身份验证令牌)进行签名。删除它们只会让每个用户注销，强制他们再次登录。

### `icon_cache` 目录

_**备份可选。**_

图标缓存存储了[网站图标](https://bitwarden.com/help/article/website-icons/)，以便它们不需要从登录站点重复获取。除非您真的想避免重新获取大量图标缓存，否则它可能不值得备份。

## 恢复备份数据

确保 Vaultwarden 已停止，然后只需将 `data` 目录中的每个文件或目录替换为其备份版本。

在恢复使用 `.backup` 或 `VACUUM INTO` 创建的备份时，请确保首先删除任何现有的 `db.sqlite3-wal` 文件，因为这可能会在 SQLite 尝试恢复 `db.sqlite3` 时导致数据库损坏使用陈旧/不匹配的 WAL 文件。但是，如果您使用`db.sqlite3`及其匹配的`db.sqlite3-wal`文件的直接副本备份数据库，则必须将这两个文件成对还原。您不需要备份或恢复 `db.sqlite3-shm` 文件。

定期运行从备份恢复的过程是一个好主意，只是为了验证您的备份是否正常工作。执行此操作时，请确保移动或保留原始数据的副本，以防备份实际上无法正常工作。

## 例子

本节包含第三方备份示例的索引。在使用示例之前，您应该彻底查看示例并了解它的作用。

- <https://github.com/ttionya/vaultwarden-backup>
- <https://github.com/shivpatel/bitwarden_rs-local-backup>
- <https://github.com/shivpatel/bitwarden_rs_dropbox_backup>
- <https://gitlab.com/1O/bitwarden_rs-backup>
- <https://github.com/jjlin/vaultwarden-backup>
- <https://github.com/jmqm/vaultwarden_backup>
