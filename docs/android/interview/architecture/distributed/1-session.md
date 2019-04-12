# 分布式 Session

当一个带有会话表示的 `Http` 请求到 `Web` 服务器后，需求在请求中的处理过程中找到 `session` 数据。而问题就在于， `session` 是保存在单机上的。 假设我们有应用A和应用B，现在一位用户第一次访问网站， `session` 数据保存在 `应用A` 中。如果我们不做处理，怎么保障接下来的请求每次都请求到 `应用A` 呢? 如请求到了 `应用B` 中，就会发现没有这位用户的 `session` 数据，这绝对是不能容忍的。

解决方案有Session Stick，Session复制，Session集中管理，基于Cookie管理，下面一一说明。

## Session Stick

在单机情况， `session` 保存在单机上，请求也是到这台单机上，不会有问题。变成多台后，如果能保障每次请求都到同一台服务，那就和单机一样了。 这需要在负载均衡设备上修改。这就是 `Session Stick` ，这种方式也会有问题：

  - 如果某一台服务器宕机或重启，那么这台服务器上的 `session` 数据就丢失了。如果 `session` 数据中还有登录状态信息，那么用户需要重现登录。
  - 负载均衡要处理具体的 `session` 到服务器的映射。

## Session复制

`Session` 复制顾名思义，就是每台应用服务，都保存会话 `session` 数据，一般的应用容器都支持。与 `Session Stick` 相比， `sessioon` 复制对负载均衡 没有太多的要求。不过这个方案还是有缺点：

  - 同步 `session` 数据带来都网络开销。只要 `session` 数据变化，就需要同步到所有机器上，机器越多，网络开销越大。
  - 由于每台服务器都保存 `session` 数据，如果集群的 `session` 数据很多，比如 90万 人在访问网站，每台机器用于保存 `session` 数据的内容占用很严重。

这就是 **Session 复制**，这个方案是靠应用容器来完成，并不依赖应用，如果应用服务数量并不是很多，可以考虑。

## Session集中管理

这个也很好理解，再加一台服务，专门来管理 `session` 数据，每台应用服务都从专门的 `session` 管理服务中取会话 `session` 数据。可以使用数据库，NOSQL数据库等。 **和Session复制相比，减少了每台应用服务的内存使用，同步session带来的网络开销问题**。但还是有缺点：

  - 读写 `session` 引入了网络操作，相对于本机读写 `session` ，带来了延时和不稳定性。
  - 如 `Session` 集中服务有问题，会影响应用。

## 基于Cookie管理

最后一个是基于 `Cookie` 管理，我们把 `session` 数据存放在 `cookie` 中，然后请求过来后，从 `cookie` 中获取 `session` 数据。与集中管理相比，这个方案并不依赖外部 的存储系统，读写 `session` 数据带来的网络操作延时和不稳定性。但依然有缺点：

  - **Cookie有长度限制，这会影响session数据的长度**。
  - **安全性**：session数据本来存储在服务端的，而这个方案是让 `session` 数据转到外部网络或客户端中，所以会有安全性问题。不过可以对写入 Cookie 的 `session` 数据做加密。
  - **带宽消耗**：由于加了session数据，带宽当然也会增加一点。
  - **性能消耗**：每次Http请求和响应都带有Session数据，对于Web服务器来说，在同样的处理情况下，响应的结果输出越少，支持的并发请求越多。