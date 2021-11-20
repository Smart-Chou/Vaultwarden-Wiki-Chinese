!>  :poop: !> 

我们的构建基于 MariaDB 客户端库，因为这是 Debian 提供的。
对最新 Oracle MySQLv8 版本的支持需要额外注意。
如果您坚持使用 MySQLv8 而不是 MariaDB，则使用旧密码散列方法而不是默认方法创建用户！

!>  :poop: !> 

!>  Alpine 目前**不支持**，在 amd64 上 Alpine 支持 sqlite 和 postgresql，在 armv7 上只支持 sqlite。

---

要使用 MariaDB (MySQL) 后端，您可以使用 [官方 Docker 映像](https://hub.docker.com/r/vaultwarden/server) 或构建您自己的二进制文件 [启用 MySQL](https:/ /github.com/dani-garcia/vaultwarden/wiki/Building-binary#mysql-backend）。

要运行二进制文件或容器，请确保设置了 ```DATABASE_URL``` 环境变量（即```DATABASE_URL='mysql://<user>:<password>@mysql/vaultwarden'``）。

**连接字符串语法：**
```ini
DATABASE_URL=mysql://[[user]:[password]@]host[:port][/database]
```
如果您的密码包含特殊字符，则需要使用百分比编码。

| ! | # | $ | % | & | ' | ( | ) | * | + | , | / | : | ; | = | ? | @ | [ | ] |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| %21 | %23 | %24 | %25 | %26 | %27 | %28 | %29 | %2A | %2B | %2C | %2F | %3A | %3B | %3D | %3F | %40 | %5B | %5D |

完整的代码列表可以在[百分比编码的维基百科页面](https://en.wikipedia.org/wiki/Percent-encoding#Percent-encoding_reserved_characters)上找到

## 使用 Docker 的示例

```bash
# 启动一个mysql容器
docker run --name mysql --net <some-docker-network>\
 -e MYSQL_ROOT_PASSWORD=<my-secret-pw>\
 -e MYSQL_DATABASE=vaultwarden\
 -e MYSQL_USER=<vaultwarden_user>\
 -e MYSQL_PASSWORD=<vaultwarden_pw> -d mysql:5.7

# 使用 MySQL Env Vars 设置启动 vaultwarden。
docker run -d --name vaultwarden --net <some-docker-network>\
 -v $(pwd)/vw-data/:/data/ -v <Path to ssl certs>:/ssl/\
 -p 443:80 -e ROCKET_TLS='{certs="/ssl/<your ssl cert>",key="/ssl/<your ssl key>"}'\
 -e RUST_BACKTRACE=1 -e DATABASE_URL='mysql://<vaultwarden_user>:<vaultwarden_pw>@mysql/vaultwarden'\
 -e ADMIN_TOKEN=<some_random_token_as_per_above_explanation>\
 -e ENABLE_DB_WAL='false' <you vaultwarden image name>
```

### 使用非 Docker MySQL 服务器的示例

```
Server IP/Port 192.168.1.10:3306 UN: dbuser / PW: yourpassword / DB: vaultwarden
mysql://dbuser:yourpassword@192.168.1.10:3306/vaultwarden
```

### 使用 docker-compose 的示例

```yaml
version: "3.7"
services:
 mariadb:
  image: "mariadb"
  container_name: "mariadb"
  hostname: "mariadb"
  restart: always
  env_file:
   - ".env"
  volumes:
   - "mariadb_vol:/var/lib/mysql"
   - "/etc/localtime:/etc/localtime:ro"
  environment:
   - "MYSQL_ROOT_PASSWORD=<my-secret-pw>"
   - "MYSQL_PASSWORD=<vaultwarden_pw>"
   - "MYSQL_DATABASE=vaultwarden"
   - "MYSQL_USER=<vaultwarden_user>"

 vaultwarden:
  image: "vaultwarden/server:latest"
  container_name: "vaultwarden"
  hostname: "vaultwarden"
  restart: always
  env_file:
   - ".env"
  volumes:
   - "vaultwarden_vol:/data/"
  environment:
## 在 mysql URL 周围使用单括号时出现问题，就像在普通 docker 示例中一样 
   - "DATABASE_URL=mysql://<vaultwarden_user>:<vaultwarden_pw>@mariadb/vaultwarden"
   - "ADMIN_TOKEN=<some_random_token_as_per_above_explanation>"
   - "RUST_BACKTRACE=1"
  ports:
   - "80:80"

volumes:
 vaultwarden_vol:
 mariadb_vol:
```

### 创建数据库和用户

1. 为 Vaultwarden 创建一个新的（空的）数据库（确保 Charset 和 Collat​​e 是正确的！）：
```sql
CREATE DATABASE vaultwarden CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

2a. 创建一个新的数据库用户并授予数据库权限（MariaDB，v8 之前的 MySQL 版本）：
```sql
CREATE USER 'vaultwarden'@'localhost' IDENTIFIED BY 'yourpassword';
GRANT ALL ON `vaultwarden`.* TO 'vaultwarden'@'localhost';
FLUSH PRIVILEGES;
```

2b. 如果您使用 MySQL v8.x，您需要像这样创建用户：
```sql
-- Use this on MySQLv8 installations
CREATE USER 'vaultwarden'@'localhost' IDENTIFIED WITH mysql_native_password BY 'yourpassword';
GRANT ALL ON `vaultwarden`.* TO 'vaultwarden'@'localhost';
FLUSH PRIVILEGES;
```
如果您已经创建了用户并想要更改密码类型：
```sql
-- Change password type from caching_sha2_password to native
ALTER USER 'vaultwarden'@'localhost' IDENTIFIED WITH mysql_native_password BY 'yourpassword';
```

您可能想尝试一组受限制的赠款：
```sql
GRANT ALTER, CREATE, DELETE, DROP, INDEX, INSERT, SELECT, UPDATE ON `vaultwarden`.* TO 'vaultwarden'@'localhost';
FLUSH PRIVILEGES;
```

### 从 SQLite 迁移到 MySQL

此 [问题评论](https://github.com/dani-garcia/vaultwarden/issues/497#issuecomment-511827057) 中描述了一种从 SQLite 迁移到 MySQL 的简单方法。下面重复这些步骤。请注意，您使用它需要您自担风险，强烈建议您备份安装和数据！

1.首先按照上面的步骤1和2
2. 配置 Vaultwarden 并启动它，以便diesel 可以运行迁移并正确设置架构。不要做别的。
3. 停止保管员。
4. 使用以下命令转储现有的 SQLite 数据库。仔细检查您的 sqlite 数据库的名称，默认应为 db.sqlite。<br>
**注意：** 您需要在 Linux 系统上安装 sqlite3 命令。<br>
我们需要从 sqlite 转储的输出中删除一些查询，例如创建表等。我们将在此处执行此操作。<br><br>
您可以使用这种单线：

```bash
sqlite3 db.sqlite3 .dump | grep "^INSERT INTO" | grep -v "__diesel_schema_migrations" > sqlitedump.sql ; echo -ne "SET FOREIGN_KEY_CHECKS=0;\n$(cat sqlitedump.sql)" > mysqldump.sql
```
或以下紧随其后：
```bash
sqlite3 db.sqlite3 .dump | grep "^INSERT INTO" | grep -v "__diesel_schema_migrations" > sqlitedump.sql
echo "SET FOREIGN_KEY_CHECKS=0;" > mysqldump.sql
cat sqlitedump.sql >> mysqldump.sql
```
5. 加载您的 MySQL 转储：
```bash
mysql --force --password --user=vaultwarden --database=vaultwarden < mysqldump.sql
```
6. 再次启动保管库。

*注意：使用 ```--show-warnings``` 加载您的 MySQL 转储将突出显示日期时间字段在导入期间被截断，**似乎**没问题。*
```
Note (Code 1265): Data truncated for column 'created_at' at row 1
Note (Code 1265): Data truncated for column 'updated_at' at row 1
```

*注1：然后错误加载数据mysqldump.sql加载错误*
```
error (1064): Syntax error near '"users" VALUES('9b5c2d13-8c4f-47e9-bd94-f0d7036ff581'*********)
```
fix:
```bash
sed -i 's#\"#\#g' mysqldump.sql
```
```bash
mysql --password --user=vaultwarden
use vaultwarden
source /vw-data/mysqldump.sql
exit
```

*注 2：如果 SQLite 数据库从以前的旧版本迁移而来，MariaDB 可能会抱怨不匹配的值计数，例如：*
```
ERROR 1136 (21S01) at line ###: Column count doesn't match value count at row 1
```
版本跳转可能添加了新的数据库列。首先使用 SQLite 后端升级 vaultwarden 以在 SQLite 数据库上运行迁移，切换到 MariaDB 后端，然后重复上述迁移步骤。或者，查找自您安装的版本以来添加迁移的提交，并使用 `sqlite3` 手动运行迁移