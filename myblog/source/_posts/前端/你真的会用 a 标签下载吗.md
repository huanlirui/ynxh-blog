# 后端接口动态生成csv格式的数据，返回前端二进制数据，唤醒浏览器进行下载

记录一下最近写的下载方法,在非同源请情况下可以将资源当成二进制的 blob 先拿到手，再进行`<a>` 的下载处理。

如果后端业务上有问题会提前抛错，这个时候需要终止下载，给予错误提示。
代码如下：

```ts
export const downloadExcelTemplate = async (_url: string, _fileName?: string) => {
  try {
    ElMessage({
      type: "info",
      message: "即将开始下载"
    });
    const res: any = await request.get({
      url: _url,
      responseType: "blob",
      headers: {
        "Content-Type": "application/x-www-form-urlencoded"
      }
    });

    if (res && res.data) {
      if (res.data.type == "application/json") {
        await fileExportResponseError(res);
        return;
      }
      ElMessage({
        type: "success",
        message: res.msg || "即将开始下载"
      });
      const url = window.URL.createObjectURL(new Blob([res.data]));
      // 创建一个a标签
      const link = document.createElement("a");
      link.href = url;
      let fileName = "";
      if (_fileName) {
        fileName = _fileName;
      } else {
        // 从Content-Disposition中提取文件名
        const contentDisposition = res.headers["content-disposition"];
        const fileNameMatch = contentDisposition?.match(/filename\*=utf-8''([^;]+)/);
        const name = fileNameMatch ? fileNameMatch[1] : "模板";
        fileName = decodeURIComponent(name);
      }

      link.setAttribute("download", fileName);
      // 模拟点击下载
      link.click();
      // 释放URL对象
      window.URL.revokeObjectURL(url);
    } else {
      ElMessage({
        type: "warning",
        message: res.msg || "下载失败"
      });
    }
  } catch (error) {
    console.log("error :>> ", error);
    ElMessage({
      type: "warning",
      message: "下载失败"
    });
  }
};
```

## 我真正遇到的问题，在跨域的情况下，无法正常进行下载，一直会下载一个错误的 txt 格式文件

后端要设置暴露 content-disposition ，否则浏览器无法正常下载 xls 格式的文件

```java
response.setHeader("Access-Control-Expose-Headers", "Content-Disposition")
```

在排查问题的过程中，刚好也进行了这块知识的复习和深入。

以下文章转载自掘金 <https://juejin.cn/post/7246747232997720120>

## a 标签 download

这应该是最常见，最受广大人民群众喜闻乐见的一种下载方式了，搭配上 `download` 属性， 就能让浏览器将链接的 URL 视为下载资源，而不是导航到该资源。

如果 `download` 再指定个 `filename` ，那么就可以在下载文件时，将其作为预填充的文件名。不过名字中的 `/` 和 `\` 会被转化为下划线 `_`，而且文件系统可能会阻止文件名中的一些字符，因此浏览器会在必要时适当调整文件名。

### 封装下载方法

贴份儿我常用的下载方法：

```ts
const downloadByUrl = (url: string, filename: string) => { if (!url) throw new Error('当前没有下载链接'); const a = document.createElement("a"); a.style.display = "none"; a.href = url; a.download = filename; // 使用target="_blank"时，添加rel="noopener noreferrer" 堵住钓鱼安全漏洞 防止新页面window指向之前的页面 a.rel = "noopener noreferrer"; document.body.append(a); a.click(); setTimeout(() => { a.remove(); }, 1000); };
```

#### Firefox 不能一次点击多次下载

这里有个兼容性问题：在火狐浏览器中，当一个按钮同时下载多个文件（调用多次）时，只能下载第一个文件。所以，我们可以利用 `<a>` 标签的 `target` 属性，将其设置成 `_blank` 让火狐在一个新标签页中继续下载。

```ts
// 检查浏览器型号和版本 const useBrowser = () => { const ua = navigator.userAgent.toLowerCase(); const re = /(msie|firefox|chrome|opera|version).*?([\d.]+)/; const m = ua.match(re); const Sys = { browser: m[1].replace(/version/, "'safari"), version: m[2] }; return Sys; };
```

添加一个浏览器判断：

```ts
const downloadByUrl = (url: string, filename: string) => { // 略...... // 火狐兼容 if (useBrowser().browser === "firefox") { a.target = "_blank"; } document.body.append(a); }
```

### download 使用注意点

`<a>` 标签虽好，但还有一些值得注意的点：

#### 1. 同源 URL 的限制

> download 只在同源 URL 或 `blob:` 、 `data:` 协议起作用

也就是说跨域是下载不了的......（这种说法不全对，除非后端配置 Content-Disposition 为 attachment，后面会讲）

首先，非同源 URL 会进行导航操作。其次，如果非要下载，那么正如上面的文档所说，可以先将其转换为 `blob:` 或 `data:` 再进行下载，至于如何转换会在 `Blob` 章节中详细介绍。

#### 2. 不能携带 Header

使用 `<a>` 标签下载是带不了 `Header` 的，因此不能通过添加请求表头的形式来鉴权，但是可以将 `sessionid` 或 `token` 字段拼接到 URL 末尾来达到鉴权的目的。这里我们给出另一个解决方案：

1. 先发送请求获取 `blob` 文件流，这样就能在请求时进行鉴权；
2. 鉴权通过后再执行下载操作。

这样是不是就能很好的同时解决问题 1 和问题 2 带来的两个痛点了呢，而且下载的文件名也能自定义了 😃

顺便提一下，`location.href` 和 `window.open` 也存在[同样的问题](https://juejin.cn/post/7156427561302982670#heading-1 "https://juejin.cn/post/7156427561302982670#heading-1")。

#### 3. download 与 Content-Disposition 的优先级

这里需要关注一个响应标头 `Content-Disposition`。在同源情况下（非同源 download 直接无效，没有可比性），它会影响 `<a>`的 `download` 从而可能产生不同的下载行为，先看一个真实下载链接的 `Response Headers`：

![Snipaste_2023-06-20_18-19-21.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c1a3ed67530b48aaa2a00af74e6bf0dc~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=677&h=146&s=12046&e=png&b=ffffff)

如图所示，`Content-Disposition` 的 value 值为 `attachment;filename=aaaa.bb`。请记住，此时 **Content-Disposition 的 filename 优先级会大于 `<a>` download 的优先级**。也就是说，当两者都指定了 `filename` 时，会优先使用 `Content-Disposition` 中的文件名。

接下来我们看看这个响应标头到底是什么。

## Content-Disposition

> 在常规的 HTTP 应答中，Content-Disposition 响应标头指示回复的内容该以何种形式展示，是以内联的形式（即网页或者页面的一部分），还是以附件的形式下载并保存到本地。

与 `Content-Type` 不同，后者用来指示资源的 MIME 类型，比如资源是图片（`image/png`）还是一段 JSON（`application/json`），而 `Content-Disposition` 则是用来指明该资源是直接展示在页面上的，还是应该当成附件下载保存到本地的。

当它作为 HTTP 消息主题的标头时，有以下三种写法：

```txt
Content-Disposition: inline Content-Disposition: attachment Content-Disposition: attachment; filename="filename.jpg"
```

### inline

默认值，即指明资源是直接展示在页面上的。 但是在同源 URL 情况下，`<a>` 元素的 `download` 属性优先级比 `inline` 大，浏览器优先使用 `download` 属性来处理下载（Firefox 早期版本除外）。

### attachment

即指明资源应该被下载到本地，大多数浏览器会呈现一个 “保存为” 的对话框，如果此时后面还有 `filename`，那么它将其优于 `download` 属性成为下载的预填充文件名。

**它的优先级最高**。

此时，无论是  **a 标签跳转**、**location.href 导航**、**window.open 打开新页面**、**直接在地址栏上输入 URL**  等，其结果：

都可以实现下载！

都可以实现下载！

都可以实现下载！

### `<a>`标签 VS Content-Disposition

介绍完 `Content-Disposition`，我们做一个横向比对的归纳一下：

- `download` VS `inline/attachment`

  优先级：`attachment` > `download` > `inline`

- `download` 的值 VS `attachment` 的值（指 filename）

  优先级：`attachment` 的值 > `download` 的值

## Blob 转换

前文介绍到，在非同源请情况下可以将资源当成二进制的 blob 先拿到手，再进行 `<a>` 的下载处理。接下来，我们介绍两种 blob 的操作：

### 方法 1. 用作 URL（blob:）

[URL.createObjectURL](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FAPI%2FURL%2FcreateObjectURL_static "https://developer.mozilla.org/zh-CN/docs/Web/API/URL/createObjectURL_static") 可以给 `File` 或 `Blob` 生成一个 URL，形式为 `blob:<origin>/<uuid>`，此时浏览器内部就会为每个这样的 URL 存储一个 URL → Blob 的映射。因此，此类 URL 很短，但可以访问 Blob。

那这就好办多了，写成代码就三行：

```js
import downloadByUrl from "@/utils/download"; const download = async () => { const blob = await fetchFile(); // 生成访问 blob 的 URL const url = URL.createObjectURL(blob); // 调用刚刚封装的 a 标签下载方法 downloadByUrl(url, "表格文件.xlsx"); // 删除映射，释放内存 URL.revokeObjectURL(url); };
```

不过它有个副作用。虽然这里有  `Blob`  的映射，但  `Blob`  本身只保存在内存中的。浏览器无法释放它。

在文档退出时（unload），该映射会被自动清除，因此  `Blob`  也相应被释放了。但是，如果应用程序寿命很长，那这个释放就不会很快发生。

**因此，如果我们创建一个 URL，那么即使我们不再需要该  `Blob`  了，它也会被挂在内存中。**

不过，[URL.revokeObjectURL](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FAPI%2FURL%2FrevokeObjectURL_static "https://developer.mozilla.org/zh-CN/docs/Web/API/URL/revokeObjectURL_static") 可以从内部映射中移除引用，允许  `Blob`  被删除并释放内存。所以，在即时下载完资源后，不要忘记立即调用 URL.revokeObjectURL。

### 方法 2. 转换为 base64（data:）

作为 `URL.createObjectURL`  的一个替代方法，我们也可以将  Blob  转换为 base64-编码的字符串。这种编码将二进制数据表示为一个由 0 到 64 的 ASCII 码组成的字符串，非常安全且“可读”。

更重要的是 —— 我们可以在 “data-url” 中使用此编码。[“data-url”](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh%2Fdocs%2FWeb%2Fhttp%2FData_URIs "https://developer.mozilla.org/zh/docs/Web/http/Data_URIs")  的形式为  `data:[<mediatype>][;base64],<data>`。我们可以在任何地方使用这种 url，和使用“常规” url 一样。

[FileReader](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FAPI%2FFileReader "https://developer.mozilla.org/zh-CN/docs/Web/API/FileReader") 是一个对象，其**唯一目的**就是从 Blob 对象中读取数据，我们可以使用它的 [readAsDataURL](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FAPI%2FFileReader%2FreadAsDataURL "https://developer.mozilla.org/zh-CN/docs/Web/API/FileReader/readAsDataURL") 方法将 Blob 读取为 base64。请看以下示例：

```js
import downloadByUrl from "@/utils/download"; const download = async () => { const blob = await fetchFile(); // 声明一个 fileReader const fileReader = new FileReader(); // 将 blob 读取成 base64 fileReader.readAsDataURL(blob); // 读取成功后 下载资源 fileReader.onload = function () { downloadByUrl(fileReader.result); }; };
```

在上述例子中，我们先实例化了一个 `fileReader`，用它来读取 blob。

一旦读取完成，就可以从 fileReader 的 `result` 属性中拿到一个`data: URL` 格式的 Base64 字符串。

最后，我们给 fileReader 注册了一个 `onload` 事件，在读取操作完成后开始下载。

### 两种方法总结与对比

`URL.createObjectURL(blob)` 可以直接访问，无需“编码/解码”，但需要记得撤销（revoke）；

而 `Data URL` 无需撤销（revoke）任何操作，但对大的  `Blob`  进行编码时，性能和内存会有损耗。

总而言之，这两种从  `Blob`  创建 URL 的方法都可以用。但通常  `URL.createObjectURL(blob)`  更简单快捷。

### responseType

最后，我们回头说一下请求的注意点：如果你的项目使用的是 `XHR` （比如 axios）而不是 `fetch`， 那么请记得在请求时添加上 [responseType](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FAPI%2FXMLHttpRequest%2FresponseType "https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest/responseType") 为 'blob'。

```js
export const fetchFile = async params => {
  return axios.get(api, { params, responseType: "blob" });
};
```

`responseType` 不是 axios 中的属性，而是 `XMLHttpRequest` 中的属性，它用于指定响应中包含的数据类型，当为 "blob" 时，表明 `Response` 是一个包含二进制数据的  `Blob` 对象。

除了 `blob` 之外，responseType 还有 `arraybuffer`、`json`、`text`等其他枚举字符串值。

## Content-Type 与 MIME 嗅探

看到这里，你可能会觉得很麻烦：什么乱七八槽的，我做需求的时候不需要搞这些花里胡哨的东西啊，直接一个 `window.open()` 或者 `location.href` 就可以直接下载了，也没有看到链接上有写什么 `Content-Dispoistion` 呀！

![132U330G-0.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1eb142c6daf543edaee4f0e480e743e8~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=248&h=185&s=10767&e=gif&f=2&b=f8f6f6)

恭喜你，遇到了一个好后端~

的确，在有些情况下，即使不指定 `Content-Dispoistion` 依然能够以最简单的方式直接下载。比如，这个下载链接：

![excel.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/24cf5ba9a2054736b61fd935adc2e2a9~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=915&h=469&s=40798&e=png&b=ffffff)

当使用 `window.open()` 或 `location.href` 发起一个 `Doc` 类型的请求时（**注意，不是 `Fetch/XHR` 类型**），浏览器便会自动下载 `.xlsx` 格式的 `excel` 文件。

这是因为 `Content-Type` 被设置成了 `application/vnd.openxmlformats-officedocument.spreadsheetml.sheet`，表示返回的内容是扩展名为 `.xlsx` 且文档类型为 `Microsoft Excel (OpenXML)` 的资源。

除了 Excel 还有 Word、PPT，它们都有自己特定的 MIME 类型：

| 扩展名  | 文档类型                       | MIME 类型                                                                 |
| ------- | ------------------------------ | ------------------------------------------------------------------------- |
| `.doc`  | Microsoft Word                 | application/msword                                                        |
| `.docx` | Microsoft Word (OpenXML)       | application/vnd.openxmlformats-officedocument.wordprocessingml.document   |
| `.xls`  | Microsoft Excel                | application/vnd.ms-excel                                                  |
| `.xlsx` | Microsoft Excel (OpenXML)      | application/vnd.openxmlformats-officedocument.spreadsheetml.sheet         |
| `.ppt`  | Microsoft PowerPoint           | application/vnd.ms-powerpoint                                             |
| `.pptx` | Microsoft PowerPoint (OpenXML) | application/vnd.openxmlformats-officedocument.presentationml.presentation |

正是这些类型，才触发了浏览器的下载行为。当然，不同的浏览器行为可能不一样。

那如果连 `Content-Type` 都没有设置呢？

> 在缺失 MIME 类型或客户端认为文件设置了错误的 MIME 类型时，浏览器可能会通过查看资源来进行  **MIME 嗅探**。
>
> 每一个浏览器在不同的情况下会执行不同的操作。（例如，Safari 会在发送的 MIME 类型不合适时查看文件的扩展名。）由于某些 MIME 类型可能代表可执行内容，会存在一些安全问题。

也就是说，此时浏览器便会自行猜测并设置 MIME 类型。但是这样做是不规范且不安全的，为了防止这种行为，后端可以设置  [`X-Content-Type-Options`](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FHTTP%2FHeaders%2FX-Content-Type-Options "https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/X-Content-Type-Options")  为  `nosniff`。

至于为什么需要避免浏览器嗅探，感兴趣的同学可以查看 👉 [为何浏览器不应该猜测 MIME 类型](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FLearn%2FServer-side%2FConfiguring_server_MIME_types%23%25E4%25B8%25BA%25E4%25BD%2595%25E6%25B5%258F%25E8%25A7%2588%25E5%2599%25A8%25E4%25B8%258D%25E5%25BA%2594%25E8%25AF%25A5%25E7%258C%259C%25E6%25B5%258B_mime_%25E7%25B1%25BB%25E5%259E%258B "https://developer.mozilla.org/zh-CN/docs/Learn/Server-side/Configuring_server_MIME_types#%E4%B8%BA%E4%BD%95%E6%B5%8F%E8%A7%88%E5%99%A8%E4%B8%8D%E5%BA%94%E8%AF%A5%E7%8C%9C%E6%B5%8B_mime_%E7%B1%BB%E5%9E%8B")。

## FAQ：a download 失效排查原因

原因一：因为跨域，真的失效了；

原因二：生效了，但被 Dispoistion 的 filename 给覆盖了；

解决思路：参考总结 👇

## 总结

1. `Content-Disposition` 为 `attachment`，无论哪种方式，都能直接下载；
   - 优点：方便；
   - 缺点：如果指定了 filename，优先级最高，前端无法直接修改文件名，除非转 `blob:` 或 `data:` 再利用 a download 下载并修改
2. `Content-Disposition` 没有设置（默认 `inline`）
   - 同源直接使用 `<a> download` 下载；
   - 跨域就先获取 `blob`，用 `createObjectURL` 或 `readAsDataURL` 读取链接，再用 `<a> download` 下载。
3. 有些特殊格式的资源（比如 Excel 表格），只需正确设置对应的 `Content-Type`，即可通过 `Doc` 类型的 HTTP 请求直接触发浏览器的下载行为。

## 参考资料

- [`<a>`锚元素 | MDN](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FHTML%2FElement%2Fa "https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/a")
- [Content-Disposition | MDN](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FHTTP%2FHeaders%2FContent-Disposition "https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Disposition")
- [PDF 预览和下载你是怎么实现的](https://juejin.cn/post/7207078219215732794#heading-14 "https://juejin.cn/post/7207078219215732794#heading-14")
- [Blob | 现代 JavaScript 教程](https://link.juejin.cn/?target=https%3A%2F%2Fzh.javascript.info%2Fblob "https://zh.javascript.info/blob")
- [URL.createObjectURL | MDN](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FAPI%2FURL%2FcreateObjectURL_static "https://developer.mozilla.org/zh-CN/docs/Web/API/URL/createObjectURL_static")
- [URL.revokeObjectURL | MDN](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FAPI%2FURL%2FrevokeObjectURL_static "https://developer.mozilla.org/zh-CN/docs/Web/API/URL/revokeObjectURL_static")
- [Data URL | MDN](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FHTTP%2FBasics_of_HTTP%2FData_URLs "https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics_of_HTTP/Data_URLs")
- [FileReader | MDN](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FAPI%2FFileReader "https://developer.mozilla.org/zh-CN/docs/Web/API/FileReader")
- [XMLHttpRequest.responseType | MDN](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FAPI%2FXMLHttpRequest%2FresponseType "https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest/responseType")
- [HTTP 的几种类型](https://link.juejin.cn/?target=https%3A%2F%2Fwww.cnblogs.com%2Fjann8%2Fp%2F17472129.html "https://www.cnblogs.com/jann8/p/17472129.html")
- [常见 MIME 类型表](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FHTTP%2FBasics_of_HTTP%2FMIME_types%2FCommon_types "https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics_of_HTTP/MIME_types/Common_types")
- [MIME types 列表](https://link.juejin.cn/?target=https%3A%2F%2Fwww.iana.org%2Fassignments%2Fmedia-types%2Fmedia-types.xhtml "https://www.iana.org/assignments/media-types/media-types.xhtml")
- [MIME 嗅探](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FHTTP%2FBasics_of_HTTP%2FMIME_types%23mime_%25E5%2597%2585%25E6%258E%25A2 "https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics_of_HTTP/MIME_types#mime_%E5%97%85%E6%8E%A2")
- [X-Content-Type-Options](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FHTTP%2FHeaders%2FX-Content-Type-Options "https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/X-Content-Type-Options")
