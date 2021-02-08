---
cover: /img/ios.jpeg
---
问题1：
输入框无法自动聚焦

问题2：
ios端日期格式显示问题invalid data

问题3：
ios长按图片会触发保存和分享功能。使用css属性来禁止。

问题4：
h5需要通过css属性来禁止全局的长按功能。
```
* {
  // -webkit-touch-callout:none; /系统默认菜单被禁用/
  // -webkit-user-select:none; /webkit浏览器/
  // -khtml-user-select:none; /早期浏览器/
  // -moz-user-select:none;/火狐/
  // -ms-user-select:none; /IE10/
  user-select: none;  //禁用移动端全局长按功能
  -webkit-touch-callout: none; //禁用ios移动端图片长按功能
}
input,
textarea {
  -webkit-user-select: auto;  //ios移动端兼容问题，必须对输入框特殊处理
}
```



