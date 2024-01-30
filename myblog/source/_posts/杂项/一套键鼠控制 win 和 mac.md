## 局域网创建 Mac 和 Windows11 共享文件夹

## 需求分析

最近公司配了 win11 台式机，我主力机是 mac ，2 套键盘鼠标实在是用起来别扭，而且有几个痛点，我需要共享剪贴板。

### 开源工具

<https://github.com/debauchee/barrier>

### 使用步骤

1. 在 win 和 mac 上下载好对应版本的 barrier
2. 我是 mac 作为服务端，win 作为客户端
3. 确保 两台电脑在同一局域网内
4. 打开配置，取消Require client certificate 的勾选
5. 配置好后分别启用运行
6. 在服务端点击设置服务端，配置屏幕的位置。尝试移动鼠标能否成功跨屏互动。

> 如果遇到问题，打开控制台的看 log

如果要启用 Require client certificate ，请参考以下步骤（对我有用）：
> git issue 多翻翻看，这个软件一堆报错问题，翻下 issue 基本能够解决

Got it working on Sonoma. Same steps, nothing different.
My mac is the client and my windows is the server.

1. On mac, cd to the "~/Library/Application Support/barrier/SSL" location in terminal.
2. Paste openssl req -new -newkey rsa:2048 -days 3650 -nodes -x509 -keyout Barrier.pem -out Barrier.pem and click enter.
3. There should be a bunch of fields to fill out. Put whatever, it doesn't matter.
4. Restart Barrier on macOS. There should now be a SSL fingerprint menu. Make sure you see your key when you click on the dropdown.
5. Copy that Barrier.pem file to the ~\AppData\Local\Barrier\SSL location on your windows machine. The .pem file needs to be the same on both the client and server.
6. Restart Barrier on Windows.
