## Vaultwarden

Vaultwarden 是一个用 Rust 编写的非官方 Bitwarden 服务器实现。它与[官方Bitwarden客户端](https://bitwarden.com/download/)兼容，非常适合不需要运行官方资源密集型服务的自托管部署。

Vaultwarden 面向个人、家庭和小型组织。开发主要对大型组织有用的功能(例如，单点登录、目录同步等)不是优先事项，但欢迎实现此类功能的高质量 PR。

## 支持的功能

Vaultwarden 实现了大多数功能所需的 Bitwarden API，包括：

- 网页界面(相当于 https://vault.bitwarden.com/)
- 个人金库支持
- [组织](https://bitwarden.com/help/article/getting-started-organizations/) 保险库支持
- [密码分享](https://bitwarden.com/help/article/share-to-a-collection/)和[访问控制](https://bitwarden.com/help/article/user-types-access-control/)
- [收藏](https://bitwarden.com/help/article/about-collections/)
- [文件附件](https://bitwarden.com/help/article/attachments/)
- [文件夹](https://bitwarden.com/help/article/folders/)
- [收藏夹](https://bitwarden.com/help/article/favorites/)
- [网站图标](https://bitwarden.com/help/article/website-icons/)
- [Bitwarden Authenticator (TOTP)](https://bitwarden.com/help/article/authenticator-keys/)
- [Bitwarden 发送](https://bitwarden.com/help/article/about-send/)
- [紧急访问](https://bitwarden.com/help/article/emergency-access/)
- [实时同步](https://bitwarden.com/blog/post/live-sync/)(仅限 WebSocket)用于桌面/浏览器客户端/扩展
- [垃圾箱](https://bitwarden.com/help/article/managing-items/#items-in-the-trash)(软删除)
- [主密码重新提示](https://bitwarden.com/help/article/managing-items/#protect-individual-items)
- 通过[email](https://bitwarden.com/help/article/setup-two-step-login-email/)、[Duo](https://help.bitwarden.in/two-step-login/two-step-login-via-duo)、[YubiKey](https://bitwarden.com/help/article/setup-two-step-login-yubikey/) 和 [FIDO2 WebAuthn](https://bitwarden.com/help/article/setup-two-step-login-fido/)两步登录
- [目录连接器](https://bitwarden.com/help/article/directory-sync/) 
  支持(基本实现，无组支持)<br/>仅支持 [v2.9.2](https://github.com/bitwarden/directory-connector/releases/tag/v2.9.2) 及更低版本，v2.9.3 及更高版本不支持使用不同的登录方式然而。
- 某些企业政策：
  - [两步登录](https://bitwarden.com/help/article/policies/#two-step-login)
  - [主密码](https://bitwarden.com/help/article/policies/#master-password)
  - [密码生成器](https://bitwarden.com/help/article/policies/#password-generator)
  - [个人所有权](https://bitwarden.com/help/article/policies/#personal-ownership)
  - [禁用发送](https://bitwarden.com/help/article/policies/#disable-send)
  - [发送选项](https://bitwarden.com/help/article/policies/#send-options)
  - [单一组织](https://bitwarden.com/help/article/policies/#single-organization)

## 缺少功能

问题 [#246](https://github.com/dani-garcia/vaultwarden/issues/246) 包含功能请求的完整列表，包括 Vaultwarden 中缺少的官方服务器的功能，以及特定于保管库管理员。

为了简化与官方服务器的比较，本节总结了官方服务器中实现的功能，这些功能目前在Vaultwarden中不可用。

可以在时间允许的情况下添加的功能(始终欢迎贡献)：

- [Bitwarden公共API](https://bitwarden.com/help/article/public-api/)
- [事件日志](https://bitwarden.com/help/article/event-logs/)
- [实时同步](https://bitwarden.com/blog/post/live-sync/)(推送通知)适用于移动客户端(Android/iOS)
- [管理员密码重置](https://bitwarden.com/help/article/admin-reset/)
- 某些企业政策：
  - [主密码重置](https://bitwarden.com/help/article/policies/#master-password-reset)

除非做出贡献，否则可能不会添加的功能：

- [单点登录 (SSO)](https://bitwarden.com/help/article/about-sso/)
- [群组](https://bitwarden.com/help/article/about-groups/)
- [自定义角色](https://bitwarden.com/help/article/user-types-access-control/#custom-role)

## 保持联系

要提出问题、提供建议、请求新功能或获得配置或安装软件的帮助，请[使用论坛](https://vaultwarden.discourse.group/)。

如果您发现 Vaultwarden 本身存在任何错误或崩溃，请[创建问题](https://github.com/dani-garcia/vaultwarden/issues/)。但是，请确保没有任何类似的问题！

如果您喜欢聊天，我们通常会在 Matrix 的 [#vaultwarden:matrix.org](https://matrix.to/#/#vaultwarden:matrix.org) 房间闲逛。欢迎加入我们！
