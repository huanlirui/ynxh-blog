---
cover: /img/vue.jpg
---

转自：https://wintc.top/   正好有此需求。借鉴博主的实现思路进行实操。以下为原文

在做 Vue、React 项目的时候常会用 axios 请求库来与后端进行数据交互。我们通常采用一个用户凭证 token 来验证用户身份，服务器根据 token 进行判断当前用户是否有权限调用接口。

经常遇到的一个问题是，调用接口时 token 可能已经过期，此时调用接口会失败，需要重新登录后再调用接口。通常我们可能处理为，用户走完登录流程后再重新手动触发一次请求。这样的实现本身没什么问题，但是给用户的操作体验上有被中断的感觉，今天尝试解决了一下这个问题。

一、业务痛点
我的业务场景是，当一个接口（比如”用户发表评论“）返回 token 过期或者未验证的状态码时，调出登录弹窗，登录成功后再次发送评论请求。当然我们可以在评论接口处对状态码进行判断，token 失效则重新发送请求；如果有其它接口也类似处理。这个方法最大的问题在于对于每个接口都要单独处理，然而程序员最大的优点就是——懒。这个方法虽然可行，对于一个懒人来说并不怎么好用，那更好的办法的是什么呢？

二、业务实现——以登录过期，重新发送请求为例（Vue 实现）
axios 请求拦截器本身可以获取请求的地址、参数、超时时间等配置，请求的重发变得很简单；同时 axios 库的请求返回为一个 promise 对象，得益于 Promise 强大的嵌套异步处理，可以使得请求接口调用对请求重发“无感”——即调用接口的时候你可以默认它一定成功，因为失败的情况在拦截器已经重发了。

1. 使用拦截器进行请求重发
   function getToken () {
   return localStorage.getItem('token')
   }

function clearToken () {
localStorage.removeItem('token')
}

function setToken (token) {
localStorage.setItem('token', token)
}

function login () {
// 登录处理代码
}

const instance = axios.create({
baseURL: 'http://xxxx', // 请求域名前缀
timeout: 10 \* 1000
})
const requestError = error => Promise.reject(error)
const beforeRequest = config => {
config.headers['token'] = getToken() || undefined
return config
}
const beforeResponse = res => () => {
if (res.data.code == 401) { // 401 错误表示未登录授权或者 token 失效
clearToken()
return login().then(() => {
return instance.request(res.config) // 这里是核心代码，登录成功后重新请求！返回 Promise 对象。
})
}
return res
}
instance.interceptors.request.use(beforeRequest, requestError)
instance.interceptors.response.use(beforeResponse, requestError) 2.登录调用（login 函数）封装
上述代码里重新请求部分调用了 login 函数（login().then(callback)），login 函数体空置了，接下来将讨论其实现。login 预期返回一个 Proimise，登录成功则 Promise.then 注册的回调函数将被调用。

我们可以将登录弹窗封装为一个组件，然后通过手动挂载组件来函数式地调用登录流程。可以在登录组件内部提供一个函数以供外部 login 函数调用，比如 showModal，该函数直接返回一个 Promise，login 函数只要调用该函数并直接返回即可。

登录组件 Login.vue 封装：

<template>
  <div v-show="show">
      <!-- 登录框 -->
  </div>
</template>

<script>
export default {
  data () {
    return {
      username: '',
      password: '',
      resolve: null,
      reject: null,
      show: false
    }
  },
  methods: {
    showModal () {
      return new Promise((resolve, rejcet) => {
        this.show = true
        this.resolve = resolve
        this.reject = reject
      })
    },
    hideModal () {
      this.show = false
      this.reject = null
      this.resolve = null
      this.username = ''
      this.password = ''
    },
    success () {
      this.hideModal()
      this.resolve && this.resolve()
    },
    fail () {
      this.hideModal()
      this.reject && this.reject()
    }
  }
}
</script>

封装好了 Login.vue 组件，接下来实现 login 函数：

import Login from '@/components/Login.vue'
import Vue from 'vue'
// ...
let vm = null
function login() {
if (!vm) {
const LoginConstructor = Vue.extend(Login)
vm = new LoginConstructor().$mount()
}
return vm.showModal()
}
// ...
login 函数闭包了一个外部变量 vm，目的是通过 vm 引用已创建的 Login 组件实例，避免重复创建（即“单例”）。这样，完整的登录后重发请求流程即完成了。

三、总结
一般而言只要谈到异步，通常提到回调函数。JS 里的 Promise 是一个强大的回调函数管理器。本文的实现方案正是利用了 Promise 的特性，在接口无权限时，将“登录+重新请求”的过程封装为 Promise 并返回，实现接口调用过程中无权限登录统一处理。这样在接口调用的地方，可以不用单独处理无权限的情况，一定程度上避免用户操作被登录流程打断。
