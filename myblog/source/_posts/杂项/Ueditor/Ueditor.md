---
cover: /img/vue.jpg
---

# vue + 百度 ueditor

## 前言

今天的前端开发，很多小伙伴都在使用 Vue，React 这种组件化的前端框架。这就导致在这些“现代”框架中集成 UEditor 变得很不平滑。所以才会有介绍如何在 Vue 项目中集成 UEditor。

> 本人使用的是 jeecg-boot 快速开发框架。我将 ueditor 集成在了项目中，以此记录过程

## Installation

```
npm i vue-ueditor-wrap
# 或者
yarn add vue-ueditor-wrap
```

## 下载

> 下载最新编译的 UEditor。官网目前最新的版本是 1.4.3.3，[官网下载地址](http://ueditor.baidu.com/website/download.html) 由于存在诸多 BUG，且官方不再积极维护。建议使用 CSDN 博主「郑昊川」修复并发布的
> [修复版本](https://github.com/HaoChuan9421/vue-ueditor-wrap/tree/master/assets/downloads)

将下载的压缩包解压并重命名为 ueidotr（只需要选择一个你需要的版本,比如 utf8-php）,放入你项目的<span style="color:red"> 静态文件 </span> 目录下。（static、public）

> 如果你使用的是 vue-cli 3.x，可以把 ueidotr 文件夹放入项目的 public 目录下。

## 引入 VueUeditorWrap 组件

```javascript
import VueUeditorWrap from "vue-ueditor-wrap"; // ES6 Module
```

## 注册组件

```javascript
components: {
  VueUeditorWrap;
}
// 或者在 main.js 里将它注册为全局组件
Vue.component("vue-ueditor-wrap", VueUeditorWrap);
```

## v-model 绑定数据 并导入配置信息

```javascript
<VueUeditorWrap v-model="message" :config="editorConfig" @ready="ready" />
```

```javascript
data() {
    return {
      message: '',
      // 简单配置
      editorConfig: {
        // 编辑器不自动被内容撑高
        autoHeightEnabled: true,
        // 初始容器高度
        initialFrameHeight: 500,
        // 初始容器宽度
        initialFrameWidth: '100%',
        // 上传文件接口, 报错属于正常，若需要验证可使用(也是盗大神的)http://35.201.165.105:8000/controller.php
        // 调试完毕打包上线则切换回/static/UEditor/php/controller.php即可，不用做其他处理
        serverUrl:'http://35.201.165.105:8000/controller.php',
        // 你的ueditor资源存放的路径,相对于打包后的index.html
        UEDITOR_HOME_URL: '/ueditor/'
      }
    }
  },
```

> 这里有大神提供了一个后端测试地址。http://35.201.165.105:8000/controller.php 仅供测试，不可在生产环节中使用

> 注意：
> <span style="color:red"> serverUrl</span> 是后端配置的地址。</br> > <span style="color:red"> EDITOR_HOME_URL </span>是你的 ueditor 资源存放的路径,相对于打包后的 index.html。 具体请看你的 webpack 设置，静态文件的路径。不可使用相对路径。

## 如何获取 UEditor 实例？

```javascript
methods: {
  ready (editorInstance) {
    console.log(`编辑器实例${editorInstance.key}: `, editorInstance)
  }
}
```

## 常见错误

* ![错误](./Ueditor/error1.png)

> 这是 UEDITOR_HOME_URL 参数配置错误导致的。在 vue cli 2.x 生成的项目中使用本组件，默认值是 '/static/UEditor/'，在 vue cli 3.x 生成的项目中，默认值是 process.env.BASE_URL + 'UEditor/' 。但这并不能满足所有情况。例如你的项目不是部署在网站根目录下，如"http://www.example.com/my-app/"，你可能需要设置为"/my-app/static/UEditor/"。是否使用了相对路径、路由是否使用 history 模式、服务器配置是否正确等等都有可能会产生影响。总而言之：无论本地开发和部署到服务器，你所指定的 UEditor 资源文件是需要真实存在的，vue-ueditor-wrap 也会在 JS 加载失败时通过 console 输出它试图去加载的资源文件的完整路径，你可以借此分析如何填写。当需要区分环境时，你可以通过判断 process.env.NODE_ENV 来分别设置。

* ![错误](./Ueditor/error2.png)

> 上传图片、文件等功能是需要与后台配合的，而你没有给 config 属性传递正确的 serverUrl ，大神提供了http://35.201.165.105:8000/controller.php 的临时接口，你可以用于测试，但切忌在生产环境使用！！！ 关于如何搭建上传接口，可以参考[官方文档](http://fex.baidu.com/ueditor/#server-deploy)。

* 图片跨域上传失败
   > UEditor 的单图上传是通过 Form 表单 + iframe 的方式实现的，但由于同源策略的限制，父页面无法访问跨域 iframe 的文档内容，所以会出现单图片跨域上传失败的问题。请后端参考[官方文档](http://fex.baidu.com/ueditor/#server-deploy)进行跨域的配置


>参考文章(https://blog.csdn.net/haochuan9421/article/details/81975966)