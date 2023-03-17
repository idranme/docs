# 跨平台

Koishi 说自己是一个「跨平台」的框架，但是这个「跨平台」是指什么呢？这并不单单是指 Koishi 支持多个运行平台，而是指 Koishi 能够同时接入多个聊天平台，并提供最原生的体验：

- Koishi 提供了统一的接口，你可以在任何聊天平台上享受完整的 Koishi 生态
- Koishi 应用可以同时接入多个聊天平台，用户可以在任意平台上与你的机器人进行交互
- Koishi 原生地支持了跨平台账号绑定，这使得用户可以带着全部数据进行无感迁移

现在就让我们来说说这件事。

## 基础概念

**平台 (Platform)** 是指聊天平台，比如 QQ、Discord 等。同一平台内的用户间具有相互发送消息的能力，而不同平台的用户间则没有。对于 Rocket Chat 这一类可自建的聊天平台而言，每个独立的自建服务器都视为不同的平台。

**机器人 (Bot)** 是指由 Koishi 操控的平台用户。这里的用户不一定是真实用户，也可以是部分平台专门提供的机器人用户。其他用户通过与机器人进行交互来体验 Koishi 的各项功能。

**适配器 (Adapter)** 是指实现了平台协议，能够让机器人接入平台的插件。通常来说一个适配器实例对应了一个机器人用户，同时启用多个适配器就实现了多个机器人的同时接入。

**消息 (Message)** 是字面意义上的消息。通常是文本或富文本格式的，有时也会包含图片、语音等媒体资源。在 Koishi 中，消息通过消息元素进行统一编码。

**频道 (Channel)** 是消息的集合。一个频道包含了具备时间、逻辑顺序的一系列消息。频道又分为私聊频道和群聊频道，其中私聊频道有且仅有两人参与，而群聊频道可以有任意多人参与。

**群组 (Guild)** 是平台用户的集合。一个群组通常会同时包含一组用户和频道，并通过权限机制让其中的部分用户进行管理。在部分平台中，群组和群聊频道的概念恰好是重合的 (例如 QQ)：一个群组内有且仅有一个群聊频道。私聊频道不属于任何群组。

## 受理人机制

默认情况下，同一个 Koishi 应用接入的多个机器人账号在同一个频道内，只有一个机器人会响应用户的消息。这是为了防止消息重复发送和循环触发等问题。这个负责响应消息的机器人被称为该频道的「受理人」。默认情况下，第一个收到该频道的消息的机器人会自动成为受理人。

当受理人账号离线时，即便频道内仍有其他机器人，它们也不会响应你的消息。此时如果想要切换受理人，可以安装 [admin](../../plugins/common/admin.md) 插件，并使用 `@bot assign`，其中 @bot 是你所希望的受理人。

反过来，如果你希望你的 Koishi 实例不主动响应某个频道的消息，可以通过 `assign -r` 的方式来移除该频道的受理人。其他频道不会受到影响。

::: tip
某个频道处于无受理人状态与该频道被过滤器排除的区别在于：前者仍然可以被通过 @机器人 的方式触发指令，而后者无论如何都无法触发指令。具体请参考 [触发前缀](./command.md#触发前缀) 章节。
:::

## 账号绑定

Koishi 支持账号绑定，即一个 Koishi 账号可以同时对应多个平台用户。完成绑定后，你无论在哪个平台上与机器人交互，数据都会被共享。

Koishi 官方目前提供了两个插件实现账号绑定，我们将逐一介绍。

### 获取平台账号信息

在我们开始之前，为了以防有人不知道自己的平台账号是什么，我们首先建议启用 inspect 插件。这个插件的功能非常简单：返回当前的平台、用户、频道、消息 ID。使用要绑定的平台账号向机器人发送 `inspect` (这里不要使用沙盒，不然只能获得沙盒用户的数据)：

<chat-panel>
<chat-message nickname="Alice">inspect</chat-message>
<chat-message nickname="Koishi">
<p>平台名：discord</p>
<p>消息 ID：1085992290352373951</p>
<p>频道 ID：835804172850561094</p>
<p>群组 ID：811975252883800125</p>
<p>用户 ID：811972350065115208</p>
<p>自身 ID：952190117479600159</p>
</chat-message>
</chat-panel>

之后只需要用到「平台名」和「用户 ID」这两个部分。我们把它们记录一下。

### 控制台登录 (管理员)

[auth](../../plugins/console/auth.md) 插件允许用户在控制台登录 Koishi 账号并管理自己的用户信息。我们首先介绍这个插件是因为它还自带了一个管理员账号，对于刚刚搭建好 Koishi 的用户来说，这是最方便的提权方式。

接着，再次进入「插件配置」界面，并点击 auth 插件。这里我们会看到有一个「管理员设置」：

![plugin-auth](/manual/console/plugin-auth.light.webp) {.light-only}

![plugin-auth](/manual/console/plugin-auth.dark.webp) {.dark-only}

填写你自己准备好的密码，然后点击「启用插件」。此时会弹出一个登录框，选择「用户密码登录」，填写你刚刚配置好的用户名 (如果你没改就是默认值 `admin`) 和密码，点击「登录」即可进入个人页面。

![login-password](/manual/console/login-password.light.webp) {.light-only}

![login-password](/manual/console/login-password.dark.webp) {.dark-only}

### 控制台登录 (普通用户)

如果你是机器人的普通用户，而机器人也开放了控制台。那么你也通过控制台来绑定自己的平台账号。进入控制台，你会发现只有寥寥几个页面是能访问的。这是因为你尚未登录。

点击左下角的「登录」按钮，选择「平台账户登录」，并输入你的平台名和用户 ID。点击「获取验证码」，并把页面中出现的验证码通过上述账号私聊发送给机器人，即可完成登录。

![login-platform](/manual/console/login-platform.light.webp) {.light-only}

![login-platform](/manual/console/login-platform.dark.webp) {.dark-only}

### 在控制台中绑定

登录完成后，你会被重定向到个人页面。在这个页面中你可以修改自己的用户名和密码。普通用户第一次登录只能使用平台账户登录，而设置了用户名和密码后，点击右上角的「应用更改」，未来就可以使用用户密码登录了。

![profile](/manual/console/profile.light.webp) {.light-only}

![profile](/manual/console/profile.dark.webp) {.dark-only}

除了修改用户名密码外，你还可以在这个页面中绑定平台用户。点击「平台账号绑定」右侧的「添加」按钮，并使用你要绑定的账号完成一遍类似平台账户登录的流程，就大功告成了。

如果要解除绑定，点击对应平台账号右侧的「解绑」即可。

### 通过指令绑定

[bind](../../plugins/common/bind.md) 插件通过指令也实现了账号绑定。使用要绑定的平台账号向机器人发送 `bind`：

<chat-panel>
<chat-message nickname="Alice">bind</chat-message>
<chat-message nickname="Koishi">
<p>指令 bind 可用于在多个平台间绑定用户数据。绑定过程中，原始平台的用户数据将完全保留，而目标平台的用户数据将被原始平台的数据所覆盖。</p>
<p>请确认当前平台是你的目标平台，并在 5 分钟内使用你的账号在原始平台内向机器人发送以下文本：</p>
<p>Koishi/123456</p>
</chat-message>
</chat-panel>

跟随提示，使用原始平台账号向机器人发送 `Koishi/123456`。如果你的第一条消息是私聊消息，那么此时绑定已经完成。如果你的第一条消息是群聊消息，则机器人会再次让你进行一遍确认：

<chat-panel>
<chat-message nickname="Alice">Koishi/123456</chat-message>
<chat-message nickname="Koishi">
<p>令牌核验成功！下面将进行第二步操作。</p>
<p>请在 5 分钟内使用你的账号在目标平台内向机器人发送以下文本：</p>
<p>Koishi/654321</p>
<p>注意：当前平台是你的原始平台，这里的用户数据将覆盖目标平台的数据。</p>
</chat-message>
</chat-panel>

再次跟随提示，使用目标平台账号向机器人发送 `Koishi/654321`，即可完成绑定。

绑定完成后，你可以随时在目标平台向机器人发送 `bind -r` 来解除绑定状态。