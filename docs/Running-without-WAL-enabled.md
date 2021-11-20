> WAL 是 SQLite 特定的设置。不适用于 MySQL 或 PostgreSQL；如果您使用这些后端之一，则`ENABLE_DB_WAL`配置选项无效。

默认情况下，在启动期间`vaultwarden` 将尝试为数据库启用 [WAL](https://sqlite.org/wal.html)。添加它可以提高性能，并且在过去有助于防止在某些情况下失败的请求。

## 关闭 WAL 的原因

一般来说，除非您绝对确定需要关闭 WAL，否则您应该启用它。但是，可能有一些有效的情况可以将其关闭。例如：

* 一些文件系统不支持 WAL - 这对于网络文件系统尤其如此。如果您使用这样的文件系统，该服务将无法启动并显示`Failed to turn on WAL`错误消息。
* 数据库需要 sqlite 版本 `3.7.0` 或更新版本，因此如果您出于任何原因(例如备份)需要使用其他无法更新的工具直接访问数据库，您可能需要禁用 WAL。
* [此处描述的缺点](https://sqlite.org/wal.html#advantages) 之一会影响您

## 如何关闭 WAL

### 0. 进行备份

这些更改通常是安全的并且可以在不丢失数据的情况下完成，但是在任何更改之前[备份您的数据](Backing-up-your-vault) 是强烈建议。

### 1. 在旧数据库上禁用 WAL

如果你有旧的 DB，它是在启用 WAL 的情况下使用的，你需要使用 sqlite 禁用它：

1. 停止`Vaultwarden`
2. 找到您的 [数据文件夹](Changing-persistent-data-location)。通常那里会有 `db.sqlite3` 文件，除非你指定了一些其他的名称来使用。
3. 使用 sqlite 打开文件：

```bash
sqlite3 db.sqlite3
```

4. 通过输入`PRAGMA journal_mode=delete;`并按回车键来禁用WAL：


```
sqlite> PRAGMA journal_mode=delete;
delete
```

5. 输入`.quit` 并按回车键退出sqlite 实用程序。 (注意开头的点)

### 2. 在 `Vaultwarden` 中禁用 WAL

要关闭 WAL，您需要将`ENABLE_DB_WAL`变量设置为`false`来启动`Vaultwarden`：

```bash
docker run -d --name vaultwarden \
  -e ENABLE_DB_WAL=false \
  -v /vw-data/:/data/ \
  -p 80:80 \
  vaultwarden/server:latest
```

确保始终以这个变量存在作为启动，即使没有它也会再次启用 WAL。 (如果发生这种情况，请从 [第一步](#1-disable-wal-on-old-db) 再次禁用它)

## 如何开启 WAL

一般来说，你只需启动 `vaultwarden` 而不将 `ENABLE_DB_WAL` 设置为 false，服务器会自动为你启用 WAL。您可以通过运行来验证这一点：

```bash
sqlite3 db.sqlite3 'PRAGMA journal_mode'
```

其中`db.sqlite3` 是`vaultwarden` 使用的数据库文件。它应该返回当前使用的模式，所以在我们的例子中它是 `wal`。禁用的 WAL 通常会报告`删除`，因为这是默认设置。