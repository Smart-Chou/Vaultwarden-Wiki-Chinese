此页面是第三方 Vaultwarden 软件包的索引。由于这些软件包不是由 Vaultwarden 维护或控制的，因此它们可能落后于官方版本，有时甚至明显滞后。如果你依赖这些包，你可能想要[启用观看](https://docs.github.com/en/github/managing-subscriptions-and-notifications-on-github/viewing-your-subscriptions#configuring- your-watch-settings-for-an-individual-repository) 用于新的 vaultwarden 版本，并让维护者知道软件包是否没有保持最新。

## Arch Linux

可在 [官方存储库](https://archlinux.org/packages/community/x86_64/vaultwarden/) 以及 [web vault](https://archlinux.org/packages/community/any/vaultwarden-网络/）。

## Debian

基于 docker 的工具链可用于构建 debian 包：https://github.com/greizgh/bitwarden_rs-debian
它捆绑了服务器和网络保险库。

## CentOS7 / RHEL7

RPM 存储库由 @MrMEEE 在此处打包：https://copr.fedorainfracloud.org/coprs/mrmeee/bitwarden_rs/ ...这还包括附加包中的网络界面。

安装说明：https://github.com/MrMEEE/bitwarden_rs_rpm/blob/master/README.md

RPM 的任何问题都可以在这里报告：https://github.com/MrMEEE/bitwarden_rs_rpm/issues

## CentOS 8 / RHEL 8

构建 RPM 并使用 Docker 将其推送到 COPR 的 repo 的一个分支。

https://github.com/alexpdp7/bitwarden_rs/tree/rpm/packages/centos8
https://copr.fedorainfracloud.org/coprs/koalillo/bitwarden_rs/

## Nix (OS)

Bitwarden_rs 在 Nix 中包含 4 个包（一个用于 mysql、sqlite 和 postgresql，一个用于 Vault）。对于 NixOS，还有一个模块 (services.bitwarden_rs)，因此 bitwarden_rs 也可以以 NixOS 声明方式进行配置。

## Cloudron

[Cloudron](https://cloudron.io) 是一个可以帮助您在服务器上运行 Web 应用程序的平台。
使用 Cloudron，您可以轻松地从 [应用程序库](https://cloudron.io/store/com.github.bitwardenrs.html) 在自定义域上安装 Bitwarden_rs
该应用程序包与上游 Web Vault 捆绑在一起，安装后无需任何进一步配置即可开始使用。 Cloudron 团队会跟踪发布并提供自动更新。

可以在 https://git.cloudron.io/cloudron/vaultwarden-app 找到包代码和问题跟踪器
 
## Home Assistant

[家庭助理](https://www.home-assistant.io/) 是一个开源的家庭自动化平台。 https://github.com/hassio-addons/addon-bitwarden 提供了一个 bitwarden_rs 社区插件。

## 为 Ubuntu 20.04 构建脚本

Dinger1986 已经创建了一个脚本来在 Ubuntu 20.04 上从源代码安装 bitwarden_rs，参见
https://github.com/dinger1986/bitwardenrs_install_script

## FreeBSD

在 [FreeBSD 端口树](https://www.freshports.org/security/vaultwarden/) 中可用，并作为 FreeBSD pkg 存储库中的二进制包提供：`pkg install vaultwarden`

## 多个 RPM 和 DEB 发行版

openSUSE 构建服务项目，支持“CentOS、Debian、Fedora、RHEL、SUSE、Ubuntu”。

您可以直接下载软件包或使用可用的存储库。

**警告：** 目前这些包包含预构建的二进制文件，无法使用此构建服务构建 rust-nightly 包。

[Vaultwarden](https://build.opensuse.org/package/show/home:Masgalor:Vaultwarden/vaultwarden)
[vaultwarden-webvault](https://build.opensuse.org/package/show/home:Masgalor:Vaultwarden/vaultwarden-webvault)