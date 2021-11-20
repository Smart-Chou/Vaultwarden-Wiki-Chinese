为方便起见，Vaultwarden 映像还将托管用于 Vault Web 界面的静态文件。您可以通过设置 WEB_VAULT_ENABLED 变量来完全禁用此静态文件托管。

```sh
docker run -d --name vaultwarden \
  -e WEB_VAULT_ENABLED=false \
  -v /vw-data/:/data/ \
  -p 80:80 \
  vaultwarden/server:latest
```

或者，您可以覆盖 Vault 文件并提供您自己的静态文件来托管。您可以通过在容器中的`/web-vault` 目录上挂载包含文件的路径来实现。只要确保该目录至少包含 `index.html` 文件。

```sh
docker run -d --name vaultwarden \
  -v /path/to/static/files_directory:/web-vault \
  -v /vw-data/:/data/ \
  -p 80:80 \
  vaultwarden/server:latest
```

请注意，您还可以通过提供带有路径的`WEB_VAULT_FOLDER`环境变量来更改 vaultwarden 查找静态文件的路径。