## /data前缀：

默认情况下，所有持久数据都保存在 `/data` 下，您可以通过设置 `DATA_FOLDER` 环境变量来覆盖此路径：

```sh
docker run -d --name vaultwarden \
  -e DATA_FOLDER=/persistent \
  -v /vw-data/:/persistent/ \
  -p 80:80 \
  vaultwarden/server:latest
```

请注意，您需要相应地调整卷安装。

## 数据库名称和位置

默认为`$DATA_FOLDER/db.sqlite3`，你可以使用`DATABASE_URL`变量来更改专门针对数据库的路径：

```sh
docker run -d --name vaultwarden \
  -e DATABASE_URL=/database/vaultwarden.sqlite3 \
  -v /vw-data/:/data/ \
  -v /vw-database/:/database/ \
  -p 80:80 \
  vaultwarden/server:latest
```

请注意，如果数据库和其他持久数据不同，您需要记住为它们安装卷。

## 附件位置

默认为 `$DATA_FOLDER/attachments`，您可以使用 `ATTACHMENTS_FOLDER` 变量更改路径：

```sh
docker run -d --name vaultwarden \
  -e ATTACHMENTS_FOLDER=/attachments \
  -v /vw-data/:/data/ \
  -v /vw-attachments/:/attachments/ \
  -p 80:80 \
  vaultwarden/server:latest
```

请注意，您需要记住为附件和其他持久数据安装卷(如果它们不同)。

## 图标缓存

默认为 `$DATA_FOLDER/icon_cache`，您可以使用 `ICON_CACHE_FOLDER` 变量更改路径：

```sh
docker run -d --name vaultwarden \
  -e ICON_CACHE_FOLDER=/icon_cache \
  -v /vw-data/:/data/ \
  -v /icon_cache/ \
  -p 80:80 \
  vaultwarden/server:latest
```

请注意，在上面的示例中，我们没有在本地挂载卷，这意味着它不会在升级过程中持久化，除非您使用 --volumes-from 使用中间数据容器。这将影响性能，因为 Vaultwarden 将不得不在重新启动时重新下载图标，但可能会使您避免缓存中的陈旧图标，因为它们不会自动清除。