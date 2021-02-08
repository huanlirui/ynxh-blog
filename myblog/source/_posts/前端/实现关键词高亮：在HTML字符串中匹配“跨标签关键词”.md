---
cover: /img/javascript.jpg
---

正好有此需求。借鉴博主的实现思路进行实操。以下为原文

转自：https://wintc.top/  
很久之前（好像刚好是一年前）写过一个 Vue 组件，匹配文本内容中的关键词高亮，类似浏览器 ctrl+f 搜索结果。实现方案是，将文本字符串中的关键字搜索出来，然后使用特殊的标签（比如 font 标签）包裹关键词替换匹配内容，最后得到一个 HTML 字符串，渲染该字符串并在 font 标签上使用 CSS 样式即可实现高亮的效果。

当时的实现过于简单，没有支持接收 HTML 字符串作为内容进行关键词匹配。这两天有同学问到，就又思考了这个问题，发现并不是那么麻烦，写了几行代码解决了一下。

一、匹配关键字：HTML 字符串与文本字符串对比

1. 纯文本字符串的处理
   对于纯文本字符串，如：“江畔何人初见月？江月何年初照人？ ”，假如我们想匹配“江月”这个关键字，则匹配结果可处理为：

江畔何人初见月？<font style="background: #ff9632">江月</font>何年初照人？
这样“江月”两个字被 font 标签包裹，在 font 标签上应用特殊的背景样式以达到关键字高亮的效果。

2. 对 HTML 字符串的处理
   对于上述例子，如果内容字符串是一个 HTML 文本：

江畔何人初见<b>月</b>？江<b>月</b>何年初照人？
对于同样的关键词“江月”，怎样处理它呢？因为关键词中的字在不同的标签内，所以只能分别用 font 标签进行替换：

江畔何人初见<b>月</b>？<font style="background: #ff9632">江</font><b><font style="background: #ff9632">月</font></b>何年初照人？
这是比较简单的情况，实际情况下关键字则可能跨多级、多层标签。

二、跨标签匹配关键词
跨标签解析关键词，其实就是对于匹配到的关键词，提取出各标签中对应的子片段，然后用 font 之类的标签包裹，再将高亮样式用于 font 标签即可。

对于整个 HTML 内容而言，渲染出来的文本由各类标签内的文本节点组成。因为关键词匹配的内容会跨标签，所以需要将各文本节点有序取出，并将节点内容拼接起来进行匹配。拼接时记下节点文本在拼接串中的起止位置，以便关键词匹配到拼接串的某位置时截取文本片段并使用 font 标签包裹。

1. 深度优先遍历 DOM 树取出文本节点
   深度优先可以采用循环或者递归的方式遍历，这里采用循环实现，按取出某个元素下所有文本节点（利用 nodeType 判断文本节点）：

function getTextNodeList (dom) {
const nodeList = [...dom.childNodes]
const textNodes = []
while (nodeList.length) {
const node = nodeList.shift()
if (node.nodeType === node.TEXT_NODE) {
textNodes.push(node)
} else {
nodeList.unshift(...node.childNodes)
}
}
return textNodes
} 2. 取出所有文本内容进行拼接
获取到了文本节点列表，可以取出所有文本内容并记录每个文本片段在拼接结果中的开始、结束索引：

function getTextInfoList (textNodes) {
let length = 0
const textList = textNodes.map(node => {
let startIdx = length, endIdx = length + node.wholeText.length
length = endIdx
return {
text: node.wholeText,
startIdx,
endIdx
}
})
return textList
},
拼接文本：

const content = textList.map(({ text }) => text).join('') 3. 匹配关键词
获得了拼接文本，可以利用拼接文本获取所有的拼接结果了。这里偷个懒直接用正则匹配吧，得把正则用到的一些特殊符号进行转义一下：

getMatchList (content, keyword) {
const characters = [...'\\[](){}?.+_^$:|'].reduce((r, c) => (r[c] = true, r), {})
  keyword = keyword.split('').map(s => characters[s] ? `\\${s}` : s).join('[\\s\\n]_')
const reg = new RegExp(keyword, 'gmi')
const matchList = []
let match = reg.exec(content)
while (match) {
matchList.push(match)
match = reg.exec(content)
}
return matchList
},
关键词字符转义处理后，字符与字符之间中间插入了正则中的空白符和换行符(\s\n)，以在匹配时忽略一些看不见的字符。上述代码循环使用使用 RegExp.prototype.exec 匹配拼接字符串中的所有关键词，每一次匹配的结果中都包含了本次匹配到的文本、匹配索引等，一个简单的例子:

注：String.prototype.matchAll 可以一次查出所有匹配结果，但是浏览器兼容性不好，matchAll 的匹配示例：

4. 关键词使用 font 标签替换
   根据关键词匹配结果索引，以及每个文本节点的起止索引，可以计算出每个关键词匹配了哪几个文本节点，其中对于开始和结束的文本节点，可能只是部分匹配到，而中间的文本节点的所有内容都是匹配到的。

比如对于 HTML 文本：

<span>江畔何人初见<b>月</b>？江月何年初照人？</span>
其 DOM 树对应的的文本节点有 3 个：

文本节点

假如关键字是“何人初见月？”，那此时，对于第一个文本节点匹配了后半部分，第二个文本节点完全匹配，第三个文本节点匹配了第一个字符。三个节点中匹配的部分需要分别用 font 标签替换：

<span>江畔<font>何人初见</font><b><font>月</font></b><font>？</font>江月何年初照人？</span>
默认情况下，连续的文字会在同一个文本节点中，而对于匹配了部分内容的文本节点，就需要将它一分为二，可以利用 Text.splitText()API 来分割文本节点，API 接收一个索引值，从索引位置将文本节点后半部分切割并返回包含后半部分内容的新文本节点。上述例子中匹配的是 3 个节点，拆分后就会得到 5 个文本节点：

中间三个文本节点即是需要被替换的节点，使用 replaceChild 就可以直接将文本节点替换为 font 标签。

对于整个 HTML 字符串，同一个关键词可能同时有多处匹配结果，因此要对所有匹配结果进行上述处理。使用前几步获取的 textNodes、textList、matchList，代码实现如下：

function replaceMatchResult (textNodes, textList, matchList) {
// 对于每一个匹配结果，可能分散在多个标签中，找出这些标签，截取匹配片段并用 font 标签替换出
for (let i = matchList.length - 1; i >= 0; i--) {
const match = matchList[i]
const matchStart = match.index, matchEnd = matchStart + match[0].length // 匹配结果在拼接字符串中的起止索引
// 遍历文本信息列表，查找匹配的文本节点
for (let textIdx = 0; textIdx < textList.length; textIdx++) {
const { text, startIdx, endIdx } = textList[textIdx] // 文本内容、文本在拼接串中开始、结束索引
if (endIdx < matchStart) continue // 匹配的文本节点还在后面
if (startIdx >= matchEnd) break // 匹配文本节点已经处理完了
let textNode = textNodes[textIdx] // 这个节点中的部分或全部内容匹配到了关键词，将匹配部分截取出来进行替换
const nodeMatchStartIdx = Math.max(0, matchStart - startIdx) // 匹配内容在文本节点内容中的开始索引
const nodeMatchLength = Math.min(endIdx, matchEnd) - startIdx - nodeMatchStartIdx // 文本节点内容匹配关键词的长度
if (nodeMatchStartIdx > 0) textNode = textNode.splitText(nodeMatchStartIdx) // textNode 取后半部分
if (nodeMatchLength < textNode.wholeText.length) textNode.splitText(nodeMatchLength)
const font = document.createElement('font')
font.innerText = text.substr(nodeMatchStartIdx, nodeMatchLength)
textNode.parentNode.replaceChild(font, textNode)
}
}
}
代码里对匹配结果遍历时，采用的是倒序遍历，原因是遍历过程对 textNodes 存在副作用：在遍历中会对 textNodes 中的文本节点进行切割。假设同一个文本节点中有多处匹配，会进行多次分割，而 textNodes 里引用的是原文本节点即前半部分，因此从后往前遍历会确保未处理的匹配文本节点的完整。

同时代码中省去了 font 节点的样式设置，这个可以根据自己的逻辑来设置。

三、完整代码调用
上述步骤描述了 HTML 字符串跨标签匹配关键词的所有流程实现，下面是完整的代码调用示例：

function replaceKeywords (htmlString, keyword) {
if (!keyword) return htmlString
const div = document.createElement('div')
div.innerHTML = htmlString
const textNodes = getTextNodeList(div)
const textList = getTextInfoList(textNodes)
const content = textList.map(({ text }) => text).join('')
const matchList = getMatchList(content, keyword)
replaceMatchResult(textNodes, textList, matchList)
return div.innerHTML
}
输入一个 HTML 字符串和关键词，将 HTML 串中的关键词用 font 标签包裹后返回。

四、总结
上述实现方案中有一些简单的细节省去了，比如设置 font 标签的样式、隐藏的 dom 匹配时忽略等。

font 标签样式设置看使用场景吧，如果是长 HTML 字符串匹配建议是不要直接设置 style 属性，而是操作样式表来达到目的。可以给 font 标签设置特殊的属性，然后使用属性选择器来设置样式。比如可以给 font 设置 highlight="${i}"属性，来针对匹配的关键词应用不同的样式。操作样式表可以给 style 标签设置 innerText 或者调用 CSSStyleSheet.insertRule()和 CSSStyleSheet.deleteRule()。

demo: https://wintc.top/laboratory/#/search-highlight

github 查看源码：https://github.com/Lushenggang/vue-search-highlight
