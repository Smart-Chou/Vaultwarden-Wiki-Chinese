## 配置方法

在 Vaultwarden 中，您可以通过环境变量或 [[admin page|Enabling-admin-page]]（将设置写入数据目录下的 `config.json` 文件）执行配置。需要注意的是，`config.json` 中的每个设置都会覆盖相应的环境变量设置（如果存在）。例如，如果你设置了环境变量`DOMAIN=https://bitwarden.example.com`，但是你的`config.json`包含`"domain": "https://vw.example.com"`，那么Vaultwarden 将根据配置文件 (`https://vw.example.com`) 中的内容生成各种链接。

一个常见的混淆源是启用管理页面（创建 `config.json` 文件），通过管理页面更改一些设置（在 `config.json` 中设置相应的值），然后尝试更改这些设置通过环境变量（这不起作用，因为`config.json` 覆盖了环境变量）。为了避免这种混淆，强烈建议坚持使用一种或另一种配置方法；也就是说，完全通过环境变量配置，或者完全通过`config.json`（无论是使用管理页面还是直接编辑`config.json`）。

请注意，管理页面的“只读配置”部分下的配置设置只能通过环境变量进行设置，因此您必须重新启动 Vaultwarden 才能对其进行更改。

## 从文件加载环境变量

如果你想将环境变量保存在一个文件中（通常命名为`.env`），你可以按如下方式加载它们：

- 使用独立的 Vaultwarden，将 `.env` 放在当前工作目录中。 Vaultwarden 将尝试在启动时加载此文件。请注意，Vaultwarden 仅查找名为“.env”的 env 文件；它不知道如何找到具有其他名称的 env 文件。
- 使用 Docker，通过使用 `docker run --env-file <env-file> ...`（让 Docker 加载 env 文件）或 `docker run -v /path/to/.env:/.env` （让 Vaultwarden 从容器内部加载 `.env` 文件）。如果您使用`--env-file`，请注意Docker 不会取消引用值，因此请确保使用`key=val` 而不是`key="val"` 或`key='val'`。
- 使用 Docker Compose，通过使用 [`env_file`](https://docs.docker.com/compose/environment-variables/#the-env_file-configuration-option) 指令。

## 配置选项

您可以在以下位置找到可以设置的环境变量列表

<https://github.com/dani-garcia/vaultwarden/blob/main/.env.template>

如果您启用 [[admin page|Enabling-admin-page]]，也会显示完整的配置选项列表。

如有任何错误或遗漏，真实来源为

<https://github.com/dani-garcia/vaultwarden/blob/main/src/config.rs>（搜索`make_config！{`）

或者，如果您的（基于 Chromium 的）浏览器支持文本片段，则可以使用此直接链接：

<https://github.com/dani-garcia/vaultwarden/blob/main/src/config.rs#LC290:~:text=make_config!%20%7B,-folders>

## 设置域URL

确保将 `DOMAIN` 环境变量（或配置文件中的 `domain`）设置为用于访问 Vaultwarden 实例的 URL。如果您不这样做，则各种功能很可能会神秘地中断。一些例子：

- `https://bitwarden.example.com`
- `https://bitwarden.example.com:8443`（非默认端口）
- `https://host.example.com/bitwarden` (尽可能避免 URL 重写技巧)