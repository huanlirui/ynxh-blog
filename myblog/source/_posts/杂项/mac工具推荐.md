> 许多工程师都喜欢使用 **Mac** 进行开发工作，笔者也是如此，所以整理了一下个人觉得好用的软件和工具，以及相关的设置并分享出来，也欢迎在评论区交流想法👏。

MacOS上有许多优秀的生产力软件和工具，加上更贴近 Linux 的使用习惯，赢得许多开发者的青睐，Windows PC 虽然无所不能，出于将工作和娱乐分开的目的，本人的 PC 已经完全变成了**Steam启动器和原神启动器**了（ 原神，启动!🤓）

> **TL；DR（Too Long，Don‘t read - 太长不读**）

文章篇幅较长，直接列一个省流概括，， 如果有精力还是建议阅读下全文，每个软件我都给出推荐理由和介绍，都是个人体验过并最终留下来长期使用的。

___

**Arc** - 浏览器， **Warp** - 终端， **Raycast** - 启动器，**Orbstack** - 容器，**Setapp** - 软件订阅，**CleanShotX** - 截屏，**OBS** - 录屏推流，**Gifox** - Gif图制作， **1Password** - 密码管理器，**Bartender** - 菜单栏管理，**Downie** - 视频下载，**IINA / Infuse** - 视频播放器，**iRightMouse** - 鼠标右键增强，**PopClip / Bob** - 鼠标工具

## 浏览器

因为浏览器在日常使用的频率非常高，且内容较多( 包括浏览器拓展和油猴脚本),故单独列举出来。

### Arc

> 目前已使用半年多，不少人说不久还是会换回 chrome 或者其他，但并没有，我相信他们都只是习惯了使用Chrome而不想折腾，但其实如果你稍微留意，会发现很多知名开发者都一直在使用 **Arc** 且评价相当不错，如 `Anthony fu` ，`t3dotgg` 等，油管教学博主有相当高的比例都在使用 `Arc`, 它绝不是一个好看的玩具，它值得你去尝试，合适不合适需要体验后才有发言权。
>
> 实际上我都认真尝试过几乎市面上所有的浏览器，尽管各有所长，但本质并没有区别，唯独 Arc 颠覆了我很多以往对浏览器的传统认知。如果你对`Chrome` 不满意， 也许你该试着尝试`Arc`，它不仅能完美替代，还能做到许多只有 Arc 才能做到的事情。 毫不夸张地说， **Arc 是个人觉得目前 Mac 上最好的浏览器，同时也非常期待今年年底推出的 Win 版本 Arc**。

如果你不了解`Arc`，欢迎阅读这篇介绍 Arc 的文章：[Arc浏览器 - 备受瞩目又充满想象力的产品 - 掘金](https://juejin.cn/post/7264168916291682341 "https://juejin.cn/post/7264168916291682341")

![Arc.jpg](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b2e1c72f187b4b399eb047bacad44e26~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1987&h=1231&s=202460&e=jpg&b=fbdccd)

### Arc 配置

> Arc 有很多优点， 但不一定适合所有人，特别有些默认配置还挺别扭的

**记录下个人配置当个备忘录，仅供参考，可以忽略😊**

- 关闭 `Litter Arc`
- 关闭 `iCloud` 同步
- 设置 `Tab` 自动归档时间为 **30** 天
- 修改个别快捷键

**Litter Arc** 其实相当于一个迷你窗口，默认是打开链接，和微信内置浏览器那种交互方式类似，我个人是比较反感的，更倾向于打开新的标签页🏷️，所以关闭了，但也有不少人喜欢的，仁者见仁了。

因为平时公司使用的电脑和个人闲余时间使用电脑不一样，但使用一样的**Apple ID**，故**iCloud** 也一样，这样在不同设备可以同步标签页的状态，但我就是想工作和生活分开，所以关闭 **iCloud** 同步。

**Arc** 的标签页是会自动归档的，也就是如果你不手动归档(关闭标签页)，超过一定时间**Arc** 会自动把标签页归档，保持你标签栏的整洁，同时也迫使你避免养成不要打开一堆标签页却从来不看的坏习惯，但是即便被归档了，你依然可以在归档里查找并复原回来，所以不需要过于担心标签页丢失的问题。

**Arc** 因为功能多且独特，有许多快捷键的频率是非常高的，如打开**开发者模式** - `Command + Shift + D` ，呼出快捷指令 `Command + T` 等等，如果你安装软件比较多的话，需要检测并注意下键位冲突的问题。

### 浏览器拓展

> **Arc** 是基于 **Chromium** 的， 自然可以使用 **Chrome Web Store** 的拓展

个人拓展的备份，有推荐的欢迎到评论区补充，篇幅有限不一一介绍了，直接贴图，个人比较推荐的是：

![Plugin..jpg](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a5f6915e19c1423a96c498109e5ceafa~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2318&h=1346&s=171581&e=jpg&b=fefefe)

1. Immersive Translate (沉浸式翻译)
2. Vimium
3. Octotree
4. GitHub Web IDE
5. Tampermonkey

**沉浸式翻译** 相比传统划词翻译，直接在原文的基础上插入译文，方便对照，可以说通过它的帮助下，阅读英文资料的速度是之前远不可比拟的。 **Vimium** 则是支持通过 Vim 键位 进行一些网页操作， 如前进后退，上下滚动，打开关闭网页等等，所有操作通过键盘完成，不需要鼠标，掌握以后大部份网页操作的效率提升显著也可预防腱鞘炎。 **Octotree** 则是在 Github repo 页面增加侧边栏，可以直接预览源码目录结构和跳转对应页面，体验非常丝滑，比原生的手动一个个点进去还是方便很多。 **GitHub Web IDE** 则是支持集成了一些Github 在线沙箱环境， 无需Clone 到本地，即可在线编辑源码和查看源代码，对于只是想学习或者粗看源码到话，非常合适。 **Tampermonkey** 是大名鼎鼎的油猴插件，它可以对特定网站运行特定的javascript脚本，所以几乎无所不能。如解除网页限制，自动刷网课，去除广告，免登陆复制...等等

### 油猴脚本

**Tampermonkey 可玩性非常高，脚本库非常丰富，它几乎无所不能**，奈何个人折腾不多，只是保留了几个脚本，如果有好用的脚本，欢迎在评论区补充 👏～

![tampermonkey.jpg](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/31541f4dcc354050a8c9ff3ae6032128~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2307&h=1312&s=193503&e=jpg&b=fafafa)

**自动无缝翻页** 当滚动到页面底部自动将第二页内容插入到底部，实现流畅且自动化的浏览体验。

**B站区域解锁** 由于某些原因，B站的番剧比较难引进，所以很多番剧都是港澳台独享，如果你没有科学手段的话，那么建议这个油猴脚本，让你无需游去对岸就能看最新的番剧。

**CSDN Greener** CSDN 阅读体验十分糟糕，如登录才运行复制，广告弹窗一大堆，SEO 污染搜索结果等等，当然也可以在搜索引擎直接 block 掉 CSDN (我就是这么做的)。

**蓝奏云网盘增强** **Google Drive** 和 **OneDrive**国内使用受限，百度网盘下载限速，阿里云资源不多，很多资源许多人都采取蓝奏云来分享，增强许多功能，如直接下载等。

**知乎增強** 增强知乎的阅读体验， 和 CSDN Greener 同理，总得有勇者屠恶龙。

## 开发相关

### Homebrew

> **Homebrew** 是一个命令行包管理器工具，像管理软件包一样集中管理你的第三方软件，支持 **Mac / Linux**

![Homebrew (1)..jpg](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/524a1267113b45d0a504a40ee9ad9496~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1200&h=630&s=33803&e=jpg&b=2e2b24)

### Warp

> **Mac** 最为流行的终端软件非 `Item2` 莫属， 但 **Warp** 横空出世了，尽管曾经我对该软件需要注册帐号来使用表达过质疑，但是相比提升的效率和更加好的开发体验，还是真香了， 可以说 **Warp** 就是一个 **现代终端** 应该有的样子。

![warp.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/df4a45d5fb4f4938961d20856fb41676~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=871&h=691&s=491373&e=png&b=31323e)

### Orbstack

> **OrbStack** 是运行 **Docker** 容器和 **Linux** 的快速、轻便且简单的方法。 **Docker Desktop** 替代方案以光速进行开发。Say goodbye to slow, clunky containers and VMs

![orbstack.jpeg](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ed9bc2112edc4614a902844e6af2c2ba~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2782&h=1924&s=303434&e=jpg&b=241f29) 在这个云原生时代，大部份开发者都离不开 **Docker**，容器化技术终结了之前程序员配了一个下午的环境只为打出一句 `Hello World` 的时代，无论多复杂的环境，直接 pull 下来就可以直接 run，加上微服务的广泛使用，容器技术确确实实给开发者带来了好处。

官方的 **Docker Compose** 非常耗费系统资源，即便是 **MacBook Pro M1 pro + 16g ram**, 运行 **2～3个** 容器， 系统也会非常卡顿，苹果的内存又比金子还贵😠，于是出现了完美替代品 - **OrbStack**。**OrbStack** 对 **CPU** 和磁盘的使用率低，对内存的需求少，而且是一款原生的 **Swift** 应用程序，可以无缝运行 **Docker** 容器和完整的 **Linux** 发行版，并提供强大的网络功能。个人可以完全免费使用！

### Surge

> **Surge**是适用于 **Mac 和 iOS** 的高级网络工具箱，满足您对网络的一切个性化，如流畅访问Github， ChatGPT 等，嗯，很常见的开发需求。

需要注意的是： Mac 和 IOS 版本是分开的，需要单独购买， IOS 版本附赠 Apple TV 版本的 TV OS版， 借助 Apple TV 可以实现一些比较 amazing 的功能（懂得都懂，不懂的我也不能说）。 ![surge.webp](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ce2ceb12260e419ebfae5d7f4e4da410~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2192&h=1504&s=93276&e=webp&a=1&b=f7f4f4)

### Dash

> **Dash** 是一个离线文档浏览器和代码片段管理器，开发者需要经常翻阅技术文档来查询某个API的用法，那么下载到 Dash 离线的观看体验更加良好。`可通过订阅 setapp 免费获取`

![CleanShot 2023-10-22 at 22.46.58.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1223c2170b094daa9f3d95ba05730a4c~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1668&h=1007&s=272177&e=png&a=1&b=fefefe)

### Telegram & Discord

许多框架和服务都提供 **Slack / Discord** 的联系方式， 通过 **Discord** 可以第一时间接收官方的一手消息，以及聆听社区的反馈。

**Discord** 的用户体验和开发体验都是非常优秀的， 比如你可以在自己的频道(Channel)植入bot和插件，api等等，如chatGPT机器人，群管理机器人等等，如果你有参与开源项目的打算（或者只是想开黑打游戏🎮），**Discord** 是非常推荐的～ ![discord.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/587336d6e5514149897ff57e06466c20~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1652&h=923&s=483577&e=png&a=1&b=292b2e)

### VS Code & Jetbrains

相信从事**web开发**的基本都使用这两，**VSCode** 作为一个编辑器的体验是完美的，而**Jetbrains旗下的IDE**则非常适合大型工程开发和重构调试等。 ![ide.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/039178fc9e554f7bacdc52487e2b008e~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1512&h=1112&s=380094&e=png&a=1&b=202224)

## 工具相关

### Raycast

> **Mac** 装机必备！！！ **Raycast** 是一个能大幅提升你日常效率的系统增强应用，通过Raycast，你可以快速启动某个应用或者进行一些操作， 只需要通过快捷键 `Command + 空格` 呼出，键入命令即可，你可以看做是Mac自带的聚焦搜索(**Spotlight**)的高阶版， 与之类似的还有 **Alfred** ,但是需要付费， 而 Raycast 是完全免费的！

可以说，装了一个 `Raycast`, 可以卸载N个软件了。 比如 **窗口管理**，`Raycast` 内置窗口管理且可以自定义快捷键， 效率up up！！ 除此之外，还有许多功能，完全可以根据你的需求拓展去使用，如果用不到也可以关闭。 ![raycast.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f8664cf73d1d4f9886df27aec8a25b11~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1112&h=742&s=250396&e=png&a=1&b=e7eaeb)

### Setapp

> **Setapp** 是 `Mac & iOS` 的正版软件分销平台， 通过订阅 **Setapp**，你可以使用上千种软件，虽然好用的就那么几个～
>
> **Note:** 如果长期使用的话，建议将常用的软件直接永久买断, 毕竟订阅一年也得**200+¥**， `因为 Setapp只会上架能被买断的软件，而不会上架订阅制的软件`。

以本人为例，通过家庭版订阅 **Setapp**， 每月 **20 ¥** 即可以使用 **Downie， Gifox，Clean Shot X，Clean My Mac， Popclip， Dash， Bartender， AirBuddy..** 等应用，还是挺实惠的～

家庭版每位成员可以在**1台Mac设备和1台ios设备**使用Setapp，未来也许会是2台Mac设备， 推荐Mac端，因为Ios的setapp真的没啥app，ios基本上有个**美区AppleID**，app store上基本都能安装到。 ![setapp.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/aafedc3b2dea49baa76a5b47ea32bddf~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1826&h=1140&s=431473&e=png&a=1&b=fbfbfb)

### PopClip

> **PopClip** 是一款鼠标工具小软件，通过该软件可以 **通过鼠标实现一键XX** 功能，如一键翻译，复制粘贴，搜索等等。有的时候想偷懒只想动鼠标，懒人必备～ `可通过订阅 setapp 免费获取`

![popclip.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/06ec2d8efb0f442f9dfb56445c48a6c0~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1130&h=713&s=356687&e=png&a=1&b=fcfcfc)

### Gifox

> Mac 最好用的 **Gif** 图制作软件 简单快捷的操作同时带键位记录， 非常合适录制一些gif图， `可通过订阅 setapp 免费获取`。

![gifox.jpg](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/414d9f5baf0345ebb8a3283680c3357a~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1367&h=1045&s=55496&e=jpg&b=faf6f6)

### Bartender

> 从2021款 MacBook 开始，引入了万恶的刘海屏，使得电脑菜单栏有一部份被遮挡.
>
> 通过 **Bartender** 可以缓解这个问题，根据需求自行管理菜单栏图标，不会再出现软件图标塞进刘海里点不到的尴尬场景😅！！！ `可通过订阅 setapp 免费获取`

![bartender.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c6425019d5c74d858e14ff61a1473f30~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1003&h=672&s=167922&e=png&a=1&b=e6eaed)

### 1Password

> 密码管理这块， 综合最佳还是老牌的 **1Password**, 家庭版拼车下来**一年40块**，省去靠大脑记密码还会弄丢的尴尬场景，输入密码只需要通过 `Command + /` 一键调出。

尽管目前 **passkey (通行密钥)** 等无密码登录方式正在逐渐走向现实，但如今依然离不开密码，所以暂时来说有一个密码管理器还是能提升许多效率的。 ![password.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/909c2a626f7a436e8a83e86543113f0f~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=662&h=220&s=42937&e=png&a=1&b=ebecee) 免费替代品推荐：**Chrome Password Manager / Bitwarden / iCloud Password & KeyChains**

### AirBuddy

> 苹果全家桶的生态互联是吸引许多人入手的原因，但 **Apple** 显然将中心集中在 **iPhone**，**AirBuddy** 可以让你在 **Mac** 管理其他苹果设备，查看可穿戴设备电量，链接动画等，蛮有意思的，价格也不贵，拼车大概7¥\*\*，`可通过订阅 setapp 免费获取`。

![airbuddy.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/54034c9f9bd34f33991c004e5188df78~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=390&h=365&s=50596&e=png&a=1&b=d9dedf)

### Input Source Pro

> `作为中国人，经常需要在不同应用间切换中英文，体验非常糟糕！！！` **Input Source Pro** 可以在你切换不同应用时候，记住之前应用所使用的输入法或者设定为使用某个输入法，这很好地保证丝滑的码字体验，**再也不会因为频繁切换中英文输入法而卡壳了，而且还是免费的**～

![inputSourcePro.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bc6c1ce2ba0b4ada92ab8e49b7bd6c53~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=892&h=760&s=186239&e=png&a=1&b=fefefe)

### CleanShot X

> 原本使用的是**开源免费**的 `Snipaste X`, 发现有更好用的 `Clean Shot X` 且在 **setapp** 里, 支持 **orc识图提取文字， 贴图， 滚动截图等**功能，是 **Mac** 上功能比较全面的截图软件，`可通过订阅 setapp 免费获取`

![cleanshotx.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/07e8eb2b244e4eaeae40ed71ea46f556~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=513&h=117&s=6076&e=png&a=1&b=161a24)

替代品推荐： **Snipaste** / **iShot** / **Shottr** / **Xnip**

### CleanMyMac X

> 算是**Mac** 上比较老牌且口碑不错的**系统清理软件**，提供软件卸载，垃圾清除，优化内存，监测硬件等功能， UI 精致美观，0使用门槛，`可通过订阅 setapp 免费获取`

![CleanMyMacX.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/41cdba1e5d41458ba549f4aca80f3317~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1165&h=714&s=853512&e=png&a=1&b=564e81)

### Live Wallpaper

> 相信使用 **Windows** 的大多数朋友都知道 **Steam的 壁纸引擎(WallPaper Engine)**，丰富的动态壁纸让你的桌面变得赏心悦目起来，这款国人开发的壁纸软件也是类似，尽管壁纸数量和壁纸引擎差别巨大，但依然有成千上万张高质量壁纸可供选择，还是不错的。

![livewallpaper.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/68a06d8541e446af814cfeaabf382db0~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1112&h=760&s=944084&e=png&a=1&b=e1eaee)

### IRightMouse Pro

> **超级右键** 是来自国产开发团队出品的其中一款精品Mac应用,增强系统右键功能，提升效率杠杠地！
>
> `Mac 系统原生的右键十分操蛋！如不能剪切，粘贴，新建文件等等`

![iRightMousePro.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8bca7c597e134c128ef14151c005b2e3~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=912&h=737&s=177829&e=png&a=1&b=eff2f3)

### Rectangle

> **Rectangle** 是 **Mac** 上开源免费且好用的窗口管理软件， 通过简单的快捷键将应用排列在屏幕不同的位置，相似的还有付费的 **Magnet**，

![rectangle.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/26877bacf0bd4837b613ceb3cd89f221~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=884&h=778&s=195540&e=png&a=1&b=eaeef0) 但是**Raycast**以及可以完美替代**Rectangle**了，少装一个软件和显示少菜单栏图标，所以卸载了～

### OpenAI translator

> MacOS 上基于**Open AI key** 的翻译软件，**chatGPT** 在翻译质量上，对传统的翻译软件都是降维打击的，该软件也支持通过 **Popclip / Bob** 等实现划词翻译。

本人在日常网页使用的是 **沉浸式翻译(Immersive Translate)** 浏览器插件，如果需要高质量的大文本量的翻译需求，则借助 **OpenAI translator**。 ![openai.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2198d64e206c4ba6a880231b5812c3e6~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=732&h=912&s=132058&e=png&a=1&b=fcfcfc)

## 影音相关

### Downie

> **Mac上 最好用的视频下载软件， 复制视频URL即可下载视频，支持合集下载，支持平台众多， 如Bilbil，Youtube，爱奇艺/土豆/优酷**， `支持 setapp 订阅`。

![downie4.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1f04255fa6f7457e95cdc9859e0d4d54~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=968&h=472&s=65734&e=png&a=1&b=d1d1d1)

### Infuse

> **Infuse** 是 **Apple** 生态下最全能的视频播放软件， `如果你在使用苹果全家桶，那么非常推荐 Infuse`。**Infuse 已经覆盖 iPhone、iPad、Apple TV、Mac 全系产品**，但是需要付费订阅或者永久买断使用， 目前价格是 68¥ / Year。

**Infuse**如果搭配 **云盘 / Nas / 他人维护的影音库** 使用， 完全可以看做是私人影音库， **Infuse** 能够对剧集和电影按照常见的类别进行自动归类，无论是在你的iPhone还是Mac或者是Ipad，还是客厅里的Apple TV，主打就是一个无缝切换和极致体验！

![infuse.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dfc8723778804f0292f3a62ef0406c27~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1680&h=1189&s=1807963&e=png&a=1&b=f6f3f3) 即便作为一款本地视频播放器，素质也是非常优秀，支持杜比视界和杜比音频等，

### IINA

> **IINA** 是 **Mac** 国人开发的一款免费开源且最好用的视频播放器， **Infuse** 很好， 但最开始并不是针对 **Mac** 而开发的， **IINA** 则是专门针对 Mac 开发。
>
> `如果你只是想要一个在 Mac 上播放本地视频的播放器，那么我会更加推荐 IINA， 而非 Infuse`

![iina.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d3223599437c4fdbb21ca04f4cb44c52~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1480&h=930&s=672242&e=png&a=1&b=395a8a)

### Spotify

全球最大的音乐&播客流媒体平台，**免费收听全球各地的歌曲**，在多端协同方面 Spotify 是没有对手的，缺点是中文冷门曲库较少以及免费功能被阉割。 ![spotify.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5784290f7e6c46e19dc7c61b25b2a092~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1590&h=1051&s=1174750&e=png&a=1&b=101010)

### 网易云音乐

因为比较喜欢看动漫，会听一些**动漫歌曲**和**翻唱**，**民谣**等，这块的体验目前没有替代。 ![netease.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7effe9cba10c46df822364ee354eed34~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1025&h=698&s=7688895&e=gif&f=165&b=f6f6f6)

## 笔记相关

> 笔记应用都是免费的，但是可以付费解锁一些功能， 如上传更大的附件，云同步，在线访问等

### Notion

**Notion** 作为引爆世界的明星产品自然不用多言，**All in One** 的典范。 ![notion.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0f23214bbc5f477a96f5d37889833bf0~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1200&h=630&s=65538&e=png&b=f7f6f3)

### Obsidian

有的人可能担心类Notion的数据放在云上不安全，想支持离线备份，那么 **Obsidian**值得一试 ![obsidian.webp](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8e6a42582a734a018b51b0dc222ff5fc~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2489&h=1237&s=18126&e=webp&b=ffffff)

### 语雀

如果 **Notion** 访问有困难，但又想做知识整理和分享， 国内目前比较好的语雀算一个。

## 苹果内置

> 整理了一些苹果内置的 APP， 不一定好用， 但因为是内置， 只要够用的话对我个人而言是没有必要替换的

### Mail

可绑定 **QQ 邮箱 和 Gmail** 等, 由于是苹果内置的，在 **iPhone / Mac / iPad** 使用都非常一致，就不折腾其他客户端了，如`Gmail/Ooutlook`等。

![mail.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/498e2d1621364a7f92741682cd5add22~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1569&h=895&s=359120&e=png&a=1&b=fdfdfd)

吐槽一下， **Mail** 经常收不到邮件，经常需要去网页端确认。。。

### iCloud

使用苹果全家桶很难离得开 **iCloud**，本人主力使用**美区Apple ID**，速度和隐私性都得到保障。

偶尔也需要和 PC 同步，同时也订阅 **OneDrive**， 不过 **OneDrive** 国内同步非常不稳定，无科学手段基本无法使用。

加上 **OneDrive** 价格更加优惠，**1T OneDrive** 通过 office 365 家庭版拼车一年才 40+ ， 国区 iCloud 2T 一个月就 68¥ 了。

### Apple One

**Apple 营收最多的产品是 iPhone**，但营收第二不是 **Mac** 也不是 **iPad**， 而是 **Apple Service** ，也就是苹果的订阅服务，其中最著名的则是 **Apple One**。

> **Apple One** 是苹果公司推出的一项服务，将 **Apple Music、Apple TV Plus、Apple Arcade、Apple News Plus、Apple Fitness Plus 和 iCloud** 存储整合在一起的捆绑包订阅服务。

通常根据 iCloud 的容量分为 中杯，大杯，超大杯，以及新加入的 iCloud 12TB 版本， 可以看到 **Apple One 支持 音乐，视频，游戏，新闻，云存储**等服务，如果需要且用得的到话，还是挺不错的。 ![appleone.webp](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ad069e4081394d4d8f3dec85009b54f8~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2880&h=1620&s=57106&e=webp&b=f5f5f5) 友情提示： Apple One 没有在国区上线， 如果需要订阅，可以转区或者注册其他地区的**Apple ID**， 如美区以及物美价廉的土耳其区等，`值得一提的是使用土区订阅苹果服务是完全合法合规的`，详情参考 - [国区使用土耳其iCloud服务，手把手保姆级上车教程](https://link.juejin.cn/?target=https%3A%2F%2Fbtsogiwudc.feishu.cn%2Fdocx%2FCgoJdHyWKopl3UxV12GcG3psnjf "https://btsogiwudc.feishu.cn/docx/CgoJdHyWKopl3UxV12GcG3psnjf")。

## 总结

还有很多已经买断的软件没有一一列举出来， 比如已经有更好的替代品就不再值得推荐了，或者用原生内置的凑合用也问题不大。 比如被许多人吐槽的 **访达(Finder)** , **Mac 中文输入法** 等等，总之，作为一个以编码为主的开发者，个人所使用过且推荐的暂且就这么多，后续有新的软件分享也会持续更新，希望对你有所收获。
