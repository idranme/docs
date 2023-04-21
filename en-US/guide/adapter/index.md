# 使用适配器

::: danger 注意
此页文档正在施工，其中的内容可能不是最新。
:::
:::

如果要问我 Koishi 作为一个机器人框架最大的特色是什么，那我一定会回答是其多账户和跨平台机制。

使用多个机器人是 **负载均衡** 的一个重要部分。使用多个机器人是 **负载均衡** 的一个重要部分。一方面，使用多个机器人可以有效地将每个机器人每天发送的信息数量限制在一个范围之内，从而降低因为风控到导致的账号问题出现的概率，同时即使出现了封号等问题，也能通过切换账号来妥善解决；另一方面，使用多个机器人可以将机器人的用户群分离，因为有助于通过配置的区别实现更好的颗粒化控制。

而跨平台则能够让你的机器人服务更大的用户群体。与此同时，如果你担心某个平台突然停止运营，或机器人被封禁，你完全可以丝滑地过渡到另一个平台上。更进一步，Koishi 允许用户进行 **跨平台的账号绑定**。即便你运营一个 TRPG 游戏或者有着深度定制的用户系统，跨平台也并不会增加用户迁移的成本。与此同时，如果你担心某个平台突然停止运营，或机器人被封禁，你完全可以丝滑地过渡到另一个平台上。更进一步，Koishi 允许用户进行 **跨平台的账号绑定**。即便你运营一个 TRPG 游戏或者有着深度定制的用户系统，跨平台也并不会增加用户迁移的成本。

## 使用多机器人

使用多个机器人有多种方法：

- 使用多台服务器运行机器人程序
- 在一台服务器的多个进程中运行机器人程序
- 在同一个进程运行多个机器人程序

当然，这三种方法并不是对立的，你完全可以同时使用上述三种方法中的两种或者更多。但是这里需要指出的是，如果使用前两种方法，由于这些机器人的运行程序本身是分离的，并不需要做特殊处理，同时你将可能面临数据竞争等问题。当然，这三种方法并不是对立的，你完全可以同时使用上述三种方法中的两种或者更多。但是这里需要指出的是，如果使用前两种方法，由于这些机器人的运行程序本身是分离的，并不需要做特殊处理，同时你将可能面临数据竞争等问题。而对于第三种方法，机器人管理程序可以对每个账号进行妥善的管理，并且能够通过复用连接的形式获得更高的性能。因此，本章节将着重介绍同一进程的多机器人开发。因此，本章节将着重介绍同一进程的多机器人开发。

在 [快速上手]() 一章中我们已经给出了一个简单的例子：

::: tabs code
```yaml
plugins:
  adapter-discord:
    token: QwErTyUiOpAsDfGhJkLzXcVbNm

  adapter-onebot:1:
    protocol: ws
    selfId: "114514"
    endpoint: ws://127.0.0.1:8080
  adapter-onebot:2:
    protocol: http
    selfId: "1919810"
    endpoint: http://127.0.0.1:5700
```
```ts
app.plugin('adapter-discord', {
  token: 'QwErTyUiOpAsDfGhJkLzXcVbNm',
})

app.plugin('adapter-onebot', {
  bots: [{
    protocol: 'ws',
    selfId: '123456789',
    endpoint: 'ws://127.0.0.1:6700',
  }, {
    protocol: 'http',
    selfId: '234567890',
    endpoint: 'http://127.0.0.1:5700',
  }],
})
```
:::

让我们来简单地总结一下多机器人的配置方法：

1. 你需要提供一个 bots 配置项，它应该是一个数组，其中包含了每一个机器人的具体配置
2. 对适用于特定平台下每一个机器人的配置，你需要提供一个以平台名为名称的配置项（例如 onebot 等）