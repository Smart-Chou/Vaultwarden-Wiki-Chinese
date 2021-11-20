
此页面主要面向对 Vaultwarden 开发感兴趣或有特定原因想要构建自己的二进制文件的人。

典型用户应该从基于 Alpine 的 Docker 镜像中[使用哪个容器镜像](Which-container-image-to-use)、[预建二进制文件](Pre-built-binaries)，或者寻找 [第三方包](Third-party-packages)。

### 依赖

* `Rust nightly`(强烈推荐使用 [rustup](https://rustup.rs))
* 在基于 Debian 的发行版上，一些通用软件包可确保构建正常进行，请安装以下内容：`build-essential`、`git`
* `OpenSSL`(应该在路径中可用，参见 [openssl crate docs](https://docs.rs/openssl/0.10.16/openssl/#automatic)) 在基于 Debian 的发行版上，您需要安装 `pkg-config` 和 `libssl-dev`
* 对于基于 Debian 的发行版上的 SQlite3 后端，您需要安装 `libsqlite3-dev`
* 对于基于 Debian 的发行版上的 MySQL 后端，您需要安装 `libmariadb-dev-compat` 和 `libmariadb-dev`
* 对于基于 Debian 的发行版上的 PostgreSQL，您需要安装 `libpq-dev` 和 `pkg-config`
* `NodeJS`(仅在编译 web-vault 时，通过系统的包管理器安装，使用 [预构建的二进制文件](https://nodejs.org/en/download/))或 [nodesource binary distribution](https://github.com/nodesource/distributions) _注意：web-vault 当前使用需要 NodeJS v11_ 的软件包基础(例如 node-sass < v4.12)

### 运行/编译

#### 所有后端

```
# 编译所有后端并运行
cargo run --features sqlite,mysql,postgresql --release
# 或者只编译所有后端(二进制文件位于 target/release/vaultwarden)
cargo build --features sqlite,mysql,postgresql --release
```

#### SQlite 后端

```
# 使用sqlite后端编译并运行
cargo run --features sqlite --release
# 或者直接用 sqlite 编译(二进制文件位于 target/release/vaultwarden)
cargo build --features sqlite --release
```

#### MySQL 后端

```
# 用mysql后端编译并运行
cargo run --features mysql --release
# 或者直接用 mysql 编译(二进制文件位于 target/release/vaultwarden)
cargo build --features mysql --release
```

#### PostgreSQL 后端

```
# 使用 postgresql 后端编译并运行
cargo run --features postgresql --release
# 或者只用 postgresql 编译(二进制文件位于 target/release/vaultwarden)
cargo build --features postgresql --release
```

运行时，服务器可以在 [http://localhost:8000](http://localhost:8000) 中访问。

~~_注意：之前的 _~~[~~_issue_~~](https://github.com/rust-lang/rust/issues/62896)~~_ 意味着由于 Rust 编译器和 LLVM 之间的不兼容，编译可能会因段错误而失败。可以使用旧版本的编译器作为解决方法，例如__`cargo +nightly-2019-08-27 build --features yourbackend --release`_~~

#### 安装网络保险库

Web Vault 的编译版本可以从 [dani-garcia/bw_web_builds](https://github.com/dani-garcia/bw_web_builds/releases) 下载。

如果您更喜欢手动编译它，请按照下列步骤操作：

_注意：构建 Vault 需要 ~1.5GB 的 RAM。在 RaspberryPI 等 1GB 或更少的系统上，请_[_启用交换_](https://www.tecmint.com/create-a-linux-swap-file/) _ 或在更强大的机器上构建它并从那里。如此多的内存仅用于构建它，使用 Vault 运行 Vaultwarden 只需要大约 10MB 的 RAM。_

* 在 [bitwarden/web](https://github.com/bitwarden/web) 克隆 git 存储库并签出最新的发布标签(例如 v2.1.1)：

```
# 克隆存储库
git clone https://github.com/bitwarden/web.git web-vault
cd web-vault
# 切换到最新的标签
git checkout "$(git tag --sort=v:refname | tail -n1)"
# 使用匹配的 jslib 提交
git submodule update --init --recursive
```

* 从 [dani-garcia/bw_web_builds](https://github.com/dani-garcia/bw_web_builds/tree/master/patches) 下载补丁文件并将其复制到`web-vault` 文件夹。 要选择要使用的版本，假设 Web Vault 是版本 `vX.Y.Z`：
  * 如果有版本为 `vX.Y.Z` 的补丁，请使用那个补丁
  * 否则，选择版本最大但仍然小于 `vX.Y.Z` 的那个
* 应用补丁

```
# 在`网络保险库`目录中
git apply vX.Y.Z.patch
```

* 然后，构建 Vault：

```
npm install
# 阅读下面的注释(我们确实将它用于我们的 docker 构建)。
# npm 审计修复
npm run dist
```

_注意：您可能会被要求运行 `npm audit fix` 来修复漏洞。这将自动尝试将软件包升级到较新版本，这可能不兼容并破坏网络保险库功能。 如果您知道自己在做什么，请自担风险使用它。顺便说一句，我们确实在我们自己的版本中使用了它！_

最后将`build`文件夹的内容复制到目标文件夹中：

* 如果你用 `cargo run --release` 运行，它是 `vaultwarden/web-vault`。
* 如果你直接运行编译好的二进制文件，它就在二进制文件的旁边，在 `vaultwarden/target/release/web-vault` 中。

### 配置

可用的配置选项记录在默认的 `.env` 文件中，可以通过取消对该文件中所需选项的注释或设置它们各自的环境变量来修改它们。有关可用的主要配置选项，请参阅此 wiki 的配置部分。

注意：环境变量会覆盖在 `.env` 文件中设置的值。

### 更多部署信息

* [配置你的反向代理](https://github.com/dani-garcia/vaultwarden/wiki/Proxy-examples)
* [通过 systemd 设置自动启动](https://github.com/dani-garcia/vaultwarden/wiki/Setup-as-a-systemd-service)

### 如何为 sqlite 后端重新创建数据库模式(对于开发人员)

安装带有货物的diesel-cli：

```
cargo install diesel_cli --no-default-features --features sqlite-bundled
```

确保数据库的正确路径在 `.env` 文件中。

如果要修改架构，请使用以下命令创建新迁移：

```
diesel migration generate <name>
```

修改 *.sql 文件，确保在 down.sql 文件中恢复任何更改。

应用迁移并保存生成的模式，如下所示：

```
diesel migration redo

# 这一步应该在使用diesel-cli > 1.3.0 时自动完成
# 柴油打印模式> src/db/sqlite/schema.rs
```

### 如何从 SQLite 后端迁移到 MySQL 后端(针对开发人员)

如果您想从 SQLite 迁移，请参阅[使用 MySQL 后端](https://github.com/dani-garcia/vaultwarden/wiki/Using-the-MySQL-Backend)。

### 如何从 SQLite 后端迁移到 PostgreSQL 后端(针对开发人员)

如果要从 SQLite 迁移，请参阅[使用 PostgreSQL 后端](https://github.com/dani-garcia/vaultwarden/wiki/Using-the-PostgreSQL-Backend)。
