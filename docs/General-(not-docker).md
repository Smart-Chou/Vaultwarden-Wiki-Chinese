## 备份

要包含在备份中的内容：

- 启动Vaultwarden时使用的环境文件
- `data` 目录
- Vaultwarden 数据库
  - SQLite 数据库默认存放在`data`目录下
  - 使用 MariaDB/PostgreSQL/MySQL 的数据库备份功能创建备份

确保记录存储备份的过程和位置！

＃＃ 恢复

- 安装 Vaultwarden
-（不适用于 SQLite）从备份中恢复数据库
- 恢复环境文件
- 将您的 `data` 目录恢复到正确的位置

## 平台特定

### FreeBSD 端口

| Item        | Location |
|-            |-         |
| Environment | `/usr/local/etc/rc.conf.d/vaultwarden` |
| Data        | `/usr/local/www/vaultwarden/data` |

### MariaDB / MySQL

见例如[MariaDB - 备份和恢复概述](https://mariadb.com/kb/en/backup-and-restore-overview/)