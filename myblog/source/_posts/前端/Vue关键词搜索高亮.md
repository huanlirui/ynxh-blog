---
cover: /img/vue.jpg
---

转自：https://wintc.top/ 正好有此需求。借鉴博主的实现思路进行实操。以下为原文

有时候给页面内容添加一个关键词搜索功能，如果搜索结果能够像浏览器搜索一样高亮显示，那找起来就会很明显体验会好很多。本文就介绍一下关键词搜索高亮的实现方案。

实现效果大概如下：

关键字搜索高亮

可在线预览：http://wintc.top/laboratory/#/search-highlight。

一、实现原理
实现原理很简单：
使用正则匹配出文本内容中的所有关键词，在关键词外包一层内联标签，比如 span 或者 font，通过 innerHtml 渲染文本。使用 CSS 控制插入的内联元素样式，并且记录下当前搜索到的结果是第几个，使用不同的样式来展示。

比如文本内容是“江畔何人初见月？江月何年初照人？”，而关键词是“月”，那替换后的字符串可能变为：

江畔何人初见<font style="background: #ff9632">月</font>？江<font style="background: #ffff00">月</font>何年初照人？
其中匹配的“月”被替换成了<font>月</font>，并且设置 font 标签的背景色，使得搜索到的第一个“月”（当前关键词）背景变为橘黄色，而第二个“月”背景变为黄色。

本文基于 Vue 实现了一个组件，并且将组件发布到了 npm 上，如果你不想自己写组件，可以直接安装使用：vue-search-highlight。接下来会介绍一下组件 vue-search-highlight 的使用，然后给出 Vue 中的搜索高亮的代码实现。

二、vue-search-highlight 组件使用
组件需要传入文本 content 以及关键词 keyword，组件会渲染出一个包含 content 并且关键词被 font 元素替换的 div 元素。
组件功能如下：

关键词高亮
关键词匹配总数以及当前关键词的索引（即是第几个搜索结果）
查找上一个、下一个功能，调用相应函数即可跳到上一个或者下一个
使用方法：

安装组件，使用 npm 或者 yarn
// 如果使用 yarn
yarn add vue-search-highlight
// 如果使用 npm
npm install vue-search-highlight​
引入
vue-search-highlight 本身是一个组件，在需要搜索高亮的页面里引入后，像正常的组件一样使用即可。
import SearchHighlight from 'vue-search-highlight'

// 注册为子组件
components: {
'search-highlight': SearchHighlight
},​
props
props 说明 备注
content 需要展示的文本，搜索即在这个文本中进行。
keyword 关键词
highlightStyle 关键词高亮的 CSS 样式 非必传，参照浏览器搜索，默认设置背景为黄色#ffff00
currentStyle 当前关键词高亮的 CSS 样式 非必传，参照浏览器搜索，默认设置背景为橘黄色#ff9632
events
组件有两个重要的数据，即搜索匹配数量和当前关键词索引，会通过$emit 弹射出来，如果需要展示搜索索引和匹配总数（比如 3 / 16），你可以监听组件的这两个事件：
事件名 返回值
＠current-change 返回值：当前关键词索引。
关键词改变的时候，如果搜索到内容，会返回 1，搜索不到则返回 0。
＠mactch-count-change 返回值：文本匹配关键词总数。
methods
你可以通过 ref 引用组件，直接调用组件内部的一些方法：
方法名 参数 说明
searchNext 无 下一个关键词滚动到可视区域
searchLast 无 上一个关键词滚动到可视区域
scrollTo index 滚动到第 index（从 1 开始）个关键词　
使用示例：

<search-highlight
class="search-highlight"
ref="search"
@current-change="currentChange"
@mactch-count-change="matchCountChange"
:content="content"
:keyword="keyword">
</search-highlight>

<script>
import SearchHighlight from 'vue-search-highlight'

export default {
  components: {
    SearchHighlight
  },
  data () {
    return {
      currentIdx: 0,
      matchCount: 0,
      keyword: '月',
      content: `
            春江花月夜
                [唐] 张若虚
      春江潮水连海平，海上明月共潮生。
      滟滟随波千万里，何处春江无月明！
      江流宛转绕芳甸，月照花林皆似霰；
      空里流霜不觉飞，汀上白沙看不见。
      江天一色无纤尘，皎皎空中孤月轮。
      江畔何人初见月？江月何年初照人？
      人生代代无穷已，江月年年望相似。
      不知江月待何人，但见长江送流水。
      白云一片去悠悠，青枫浦上不胜愁。
      谁家今夜扁舟子？何处相思明月楼？
      可怜楼上月徘徊，应照离人妆镜台。
      玉户帘中卷不去，捣衣砧上拂还来。
      此时相望不相闻，愿逐月华流照君。
      鸿雁长飞光不度，鱼龙潜跃水成文。
      昨夜闲潭梦落花，可怜春半不还家。
      江水流春去欲尽，江潭落月复西斜。
      斜月沉沉藏海雾，碣石潇湘无限路。
      不知乘月几人归，落月摇情满江树。`,
    }
  },
  methods: {
    searchNext () {
      this.$refs.search.searchNext()
    },
    searchLast () {
      this.$refs.search.searchLast()
    },
    matchCountChange (count) {
      this.matchCount = count
    },
    currentChange (idx) {
      this.currentIdx = idx
    },
    checkKeydown (event) {
      if (event.shiftKey) {
        this.searchLast()
      } else {
        this.searchNext()
      }
    }
  }
}
</script>

三、组件代码实现
vue-search-highlight 组件代码如下：

<template>
  <div class="search-highlight" v-html="contentShow">
  </div>
</template>

<script>
const CLASS_NAME = 'search-hightlight'

export default {
  props: {
    content: {
      type: String,
      default: ''
    },
    keyword: {
      type: String,
      default: ''
    },
    highlightStyle: {
      type: String,
      default: 'background: #ffff00'
    },
    currentStyle: {
      type: String,
      default: 'background: #ff9632'
    }
  },
  data () {
    return {
      lightIndex: 0,
      matchCount: 0
    }
  },

  computed: {
    contentShow () {
      if (!this.keyword) return this.content
      let reg = new RegExp(this.keyword, 'g')
      let stringList = this.content.split(reg)
      if (!stringList.length) return this.content
      let content = ''
      for (let i = 0; i < stringList.length - 1; i++) {
        let style = i === this.lightIndex ? this.currentStyle : this.highlightStyle
        content += `${stringList[i]}<font style="${style}" ${CLASS_NAME}>${this.keyword}</font>`
      }
      content += stringList[stringList.length - 1]
      return content
    }
  },
  watch: {
    keyword: {
      immediate: true,
      handler () {
        this.lightIndex = 0
        this.getMatchCount()
      }
    },
    lightIndex: {
      immediate: true,
      handler () {
        this.$emit('current-change', this.lightIndex)
      }
    },
    matchCount: {
      immediate: true,
      handler () {
        this.$emit('mactch-count-change', this.matchCount)
      }
    }
  },
  methods: {
    scrollTo (index) {
      this.$nextTick(() => {
        let list = this.$el.querySelectorAll(`font[${CLASS_NAME}]`)
        if (list[index]) {
          this.lightIndex = index
          list[index].scrollIntoView()
        }
      })
    },
    searchNext () {
      this.$nextTick(() => {
        let idx = this.lightIndex >= this.matchCount - 1 ? 0 : this.lightIndex + 1
        this.scrollTo(idx)
      })
    },
    searchLast () {
      this.$nextTick(() => {
        let idx = this.lightIndex <= 0 ? this.matchCount - 1 : this.lightIndex - 1
        this.scrollTo(idx)
      })
    },
    getMatchCount () {
      this.$nextTick(() => {
        let list = this.$el.querySelectorAll(`font[${CLASS_NAME}]`)
        this.matchCount = list.length
      })
    },
  }
}
</script>
