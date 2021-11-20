!>  :skull:
    **使用这些命令风险自负！**
    **在做任何可能破坏整个保险库的事情之前，请务必创建备份！**

## 一般的
Vaultwarden 最初设计时仅使用 SQLite，但当时 MariaDB (MySQL) 和 PostgreSQL 也被添加到组合中。
对于 SQLite，您不必运行单独的服务器或容器，而对于其他两个，您确实需要运行一些额外的东西。

现在，如果您开始使用 MariaDB 并想回到 SQLite，该怎么办？
嗯，这是可能的，但是使用以下步骤可能会有一些我们不知道的怪癖。如果您遇到任何奇怪的问题并需要帮助，甚至解决它，请在此处打开新讨论：https://github.com/dani-garcia/vaultwarden/discussions 以帮助您和其他人。

# 如何从 MariaDB 迁移到 SQLite

确保您对 SQLite 和 MariaDB 使用相同版本的 Vaultwarden（Docker 或自定义构建），不要在这些步骤之间更新 Docker 映像。
要迁移到 SQLite，我们首先需要有一个 SQLite 数据库文件，我们可以用它来传输数据。
要创建此文件，您需要停止当前的 Vaultwarden 实例，并将其配置为使用 SQLite。
例如，您可以通过将“DATABASE_URL”从“DATABASE_URL=mysql://<vaultwarden_user>:<vaultwarden_pw>@mariadb/vaultwarden”更改为“DATABASE_URL=/data/db.sqlite3”来实现。 （ `/data` 是您使用的 `-v` 卷的 Docker 容器内的内部路径）。

更改配置后，启动 Vaultwarden，它应该通过检查以“Executing migration script ....”开头的行的日志来显示它执行了一些迁移。

现在再次停止 Vaultwarden，以便您可以开始迁移。
您需要用于 MariaDB 的数据库主机和凭据才能继续。

现在运行以下单行程序并将 `<dbhost>`、`<dbuser>` 和 `<database>` 调整为您用于 MariaDB 连接的内容。

```bash
mysqldump \
  --host=<dbhost> \
  --user=<dbuser> --password \
  --skip-create-options \
  --compatible=ansi \
  --skip-extended-insert \
  --compact \
  --single-transaction \
  --no-create-db \
  --no-create-info \
  --hex-blob \
  --skip-quote-names <database> \
  | grep "^INSERT INTO" | grep -v "__diesel_schema_migrations" \
  | sed 's#\\"#"#gm' \
  | sed -sE "s#,0x([^,]*)#,X'\L\1'#gm" \
   > mysql-to-sqlite.sql
```

系统会提示您输入密码，输入密码并按回车键。

这应该会生成一个文件“mysql-to-sqlite.sql”来保存你的数据库。
现在查找 db.sqlite3 文件 Vaultwarden 在您第一次使用 SQLite 作为数据库启动 Vaultwarden 时在上一步中创建的文件。
复制或移动 `mysql-to-sqlite.sql`，以便 `db.sqlite3` 和导出都在同一目录中。
现在您可以执行以下操作

```bash
sqlite3 db_new.sqlite3 < mysql-to-sqlite.sql
```

这应该已经用转储填充了 SQLite 数据库，您现在可以使用 SQLite 而不是 MySQL 再次启动 Vaultwarden。
