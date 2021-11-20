WebSocket 通知用于通知浏览器和桌面 Bitwarden 客户端发生了一些感兴趣的事件，例如密码数据库中的条目被修改或删除时。收到通知后，客户端可以采取适当的操作，例如重新获取修改后的条目，或从其本地数据库副本中删除已删除的条目。在此通知方案中，Bitwarden 客户端与 Bitwarden 服务器(在本例中为 vaultwarden)建立持久的 WebSocket 连接。每当服务器有事件要报告时，它就会通过此持久连接将其发送给客户端。

请注意，WebSocket 通知不适用于移动 (Android/iOS) Bitwarden 客户端。这些客户端使用原生推送通知服务 ([FCM](https://firebase.google.com/docs/cloud-messaging) for Android，[APNs](https://developer.apple.com/go/?id=push-notifications) 适用于 iOS)。 Vaultwarden 目前不支持向移动客户端推送通知。

要启用 WebSockets 通知，需要外部反向代理，并且必须对其进行配置以执行以下操作：

- 将 `/notifications/hub` 端点路由到 WebSocket 服务器，默认在端口 `3012`，确保传递 `Connection` 和 `Upgrade` 标头。 (注意端口可以用`WEBSOCKET_PORT`变量改变)
- 将其他所有内容(包括`/notifications/hub/negotiate`)路由到标准 Rocket 服务器，默认在端口`80`。
- 如果使用 Docker，您可能需要使用 `-p` 标志映射两个端口

示例配置包含在 [代理示例](Proxy-examples) 中。

然后你需要通过将`WEBSOCKET_ENABLED`变量设置为`true`来在Vaultwarden端启用WebSockets协商：

```sh
docker run -d --name vaultwarden \
  -e WEBSOCKET_ENABLED=true \
  -v /vw-data/:/data/ \
  -p 80:80 \
  -p 3012:3012 \
  vaultwarden/server:latest
```

注意：此解决方法的原因是 Rocket 缺乏对 WebSockets 的支持(尽管 [这是一个计划中的功能](https://github.com/SergioBenitez/Rocket/issues/90))，这迫使我们启动一个辅助服务器在一个单独的端口上。