传统上，Vaultwarden 仅限于驻留在子域的根，例如`https://vaultwarden.example.com`。

此限制源于后端和 Web 保管库，它们并非旨在容纳备用基础目录（请参阅 [bitwarden/server#277](/bitwarden/server/issues/277)）。移动/桌面应用程序和浏览器扩展实际上没有问题，使用带有路径的基本 URL。

在 vaultwarden 中，随着 [PR#868](https://github.com/dani-garcia/vaultwarden/pull/868)（后端）和 [PR#11](https://github.com/dani -garcia/bw_web_builds/pull/11)（网络保险库），您可以在备用基本目录中配置功能齐全的实例。

＃＃ 配置

只需配置您的域 URL 以包含基本目录。例如，假设您想通过 `https://vaultwarden.example.com/base-dir` 访问您的实例。 （请注意，如果需要，您还可以使用多级目录，例如 `https://vaultwarden.example.com/multi/level/base/dir`。）

1. 停止保管人。
2. 如果您通常使用管理页面配置 Vaultwarden，请编辑您的 `config.json` 如下所示：
    ```javascript
    {
      "domain": "https://vaultwarden.example.com/base-dir",
      // ... other values ...
    }
    ```
3. 如果您通常通过环境变量配置 Vaultwarden，请更新您的配置文件/脚本以将 `DOMAIN` 环境变量设置为基本 URL。例如：
   ```sh
   docker run -e DOMAIN="https://vaultwarden.example.com/base-dir" ...
   ```
4. 重新启动保管库。
5. 您现在应该可以通过 `https://vaultwarden.example.com/base-dir/` 访问网络保险库（注意尾部斜杠）。出于不完全清楚的原因，如果您使用`https://vaultwarden.example.com/base-dir`（没有尾部斜杠），您可能会遇到问题。
6. 将您的应用程序或浏览器扩展程序配置为使用 `https://vaultwarden.example.com/base-dir`。如果添加尾部斜杠，应用程序和扩展程序将在保存前自动将其删除。
7. 注意**5**。尾部斜线`/` 问题可以通过在路由位置字符串后附加`/` 来解决。例如，在 Nginx 中。

    ```
    location /my-base-path {
      # This config would cause `/` issue
    }
    
    location /my-base-path-2/ {
      # This config works perfectly
    }
    ```

## 反向代理

如果您将 vaultwarden 置于反向代理之后，请确保您的代理配置为将请求路径传递到 vaultwarden，因为 vaultwarden API 路由设置为期望基本目录。因此，如果对 `https://vaultwarden.example.com/base-dir/api/sync` 的请求命中了你的反向代理，然后代理到你的 vaultwarden 监听 `localhost:8080`，请求必须转到 `http ://localhost:8080/base-dir/api/sync`，而不是`http://localhost:8080/api/sync`。