当你运行 vaultwarden 时，它会产生 `2 * <number of cpu cores>` worker 来处理请求。在某些系统上，这可能会导致 worker 数量减少，从而导致性能下降，因此 docker 映像中的默认值更改为产生 10 个线程。您可以通过设置`ROCKET_WORKERS`变量来覆盖此设置以增加或减少工作人员的数量。

在下面的例子中，我们从 20 个 workers 开始：

```sh
docker run -d --name vaultwarden \
  -e ROCKET_WORKERS=20 \
  -v /vw-data/:/data/ \
  -p 80:80 \
  vaultwarden/server:latest
```