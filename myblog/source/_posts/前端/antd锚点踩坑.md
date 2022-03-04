

最近使用了antd vue 版本的锚点功能，发现几个坑点。记录一下！

场景：
使用antd 抽屉页 ，在抽屉页中顶部固定锚点跳转连接

内容部分使用id来给予锚点标识



坑：

由于抽屉页，弹窗页的dom层级均是root根节点以外。滚动容器也不是默认的window。因此锚点不会生效。

必须手动指定滚动容器。

解决方案：
方案并不完美，但确实解决了我的需求问题。

锚点组件提供了一个属性getContainer 需要返回一个 htmlElement  用于绑定滚动容器，生效区间。

vue2中template内无法使用箭头函数，直接调用document 或者 jq的this.$el 来进行htmlElement 的返回

（上述这句话可能不严谨，但是我实验下来确实如此，有没有大神教教我怎么写？）

只好考虑定义一个方法来返回节点
:getContainer = this.getContainer

```
getContainer() {
      let htmlElement
      try {
        htmlElement = this.$el.querySelector('.customerDrawer .ant-drawer-wrapper-body')
      } catch (error) {
        htmlElement = document.querySelector('.customerDrawer .ant-drawer-wrapper-body')
      }
    return htmlElement
}
```

这里使用try catch是因为 this.$el 可能还是undefine 

