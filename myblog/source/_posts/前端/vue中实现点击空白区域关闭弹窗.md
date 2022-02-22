---
cover: /img/vue.jpg
---

1. 第一种做法
   首页在外层容器里面取一个名字为 main，即 ref="main",当 bankSwitch 为 true 的时候，弹窗出现

```

<div class="selectedBorder" ref="main">
	<div class="bankItem" v-if="bankSwitch == true">
		你好我是弹窗里面的内容部分
  	</div>
</div>
```

所触发的事件如下：
首页，先在全局创建一个点击事件:bodyCloseMenus

事件作用：当点击 main 容器的时候（this.refs.main && !this.refs.main.contains(e.target)），并且弹窗出现的时候（self.bankSwitch == true），点击空白区域将弹窗关闭（self.bankSwitch = false）

最后，在页面注销前，将点击事件给移除

```
mounted() {
    document.addEventListener("click", this.bodyCloseMenus);
  },
  methods:{
  bodyCloseMenus(e) {
      let self = this;
      if (this.$refs.main && !this.$refs.main.contains(e.target)) {
        if (self.bankSwitch == true){
          self.bankSwitch = false;
        }
      }
    },
beforeDestroy() {
    document.removeEventListener("click", this.bodyCloseMenus);
  },
```

2.第二种做法
首页在外层容器里面定义一个阻止冒泡事件，即@click.stop,当 bankSwitch 为 true 的时候，弹窗出现

```
<div class="selectedBorder" @click.stop>
	<div class="bankItem" v-if="bankSwitch == true">
		你好我是弹窗里面的内容部分
  	</div>
</div>
```

所触发的事件如下：

首页，先在全局创建一个点击事件:bodyCloseMenus

事件作用：当弹窗出现的时候（self.bankSwitch == true），点击空白区域将弹窗关闭（self.bankSwitch = false）

最后，在页面注销前，将点击事件给移除

```
mounted() {
    document.addEventListener("click", this.bodyCloseMenus);
  },
  methods:{
  bodyCloseMenus(e) {
      let self = this;
        if (self.bankSwitch == true){
          self.bankSwitch = false;
        }
    },
beforeDestroy() {
    document.removeEventListener("click", this.bodyCloseMenus);
  },

```
