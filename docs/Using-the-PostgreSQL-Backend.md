要使用 PostgreSQL 后端，您可以使用 [官方 Docker 映像](https://hub.docker.com/r/vaultwarden/server) 或构建您自己的二进制文件 [启用 PostgreSQL](Building-binary#postgresql-后端)。

要运行二进制文件或容器，请确保设置了 `DATABASE_URL` 环境变量(即 `DATABASE_URL='postgresql://<user>:<password>@postgresql/bitwarden'`)

**连接字符串语法：**

```ini
DATABASE_URL=postgresql://[[user]:[password]@]host[:port][/database]
```

一个示例 docker run 环境变量是：

```
-e 'DATABASE_URL=postgresql://postgresadmin:strongpassword@postgres:5432/vaultwarden'
```

如果您的密码包含特殊字符，则需要使用百分比编码。

| ! | # | $ | % | & | ' | ( | ) | * | + | , | / | : | ; | = | ? | @ | [ | ] |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| %21 | %23 | %24 | %25 | %26 | %27 | %28 | %29 | %2A | %2B | %2C | %2F | %3A | %3B | %3D | %3F | %40 | %5B | %5D |

完整的代码列表可以在[百分比编码的维基百科页面](https://en.wikipedia.org/wiki/Percent-encoding#Percent-encoding_reserved_characters)上找到

**从 SQLite 迁移到 PostgreSQL**

有一种从 SQLite 迁移到 PostgreSQL 或 MySQL 的简单方法，但请注意，您 ** 使用它的风险由您自己承担，强烈建议您备份安装和数据！**。这是**不受支持的**，尚未经过稳健测试。

1. 为 Vaultwarden 创建一个新的(空)数据库：
```sql
CREATE DATABASE vaultwarden;
```

2.新建数据库用户并授予数据库权限：
```sql
CREATE USER vaultwarden WITH ENCRYPTED PASSWORD 'yourpassword';
GRANT all privileges ON database vaultwarden TO vaultwarden;
```

3. 配置 Vaultwarden 并启动它，以便diesel 可以运行迁移并正确设置架构。不要做别的。
4. 停止保管员。
5. 安装[pgloader](http://pgloader.io/)
6. [禁用WAL](Running-without-WAL-enabled#_1-在旧数据库上禁用-wal) SQLite 数据库。
7.创建文件bitwarden.load，内容如下：
```
load database
     from sqlite:///where/you/keep/your/vaultwarden/db.sqlite3 
     into postgresql://yourpgsqluser:yourpgsqlpassword@yourpgsqlserver:yourpgsqlport/yourpgsqldatabase
     WITH data only, include no drop, reset sequences
     EXCLUDING TABLE NAMES LIKE '__diesel_schema_migrations'
     ALTER SCHEMA 'bitwarden' RENAME TO 'public'
;
```

8. 运行命令 
```
pgloader bitwarden.load
``` 

你可能会看到一些警告，但迁移应该成功完成
9. 再次启动 Vaultwarden。