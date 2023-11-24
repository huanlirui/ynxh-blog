路由的hash和History模式

为了达到改变视图的同时不会向后端发出请求这一目的，浏览器当前提供了以下两种支持：
hash模式：即地址栏 URL 中的 # 符号
比如这个 URL：<http://www.abc.com/#/hello，> hash 的值为 #/hello
它的特点在于：hash 虽然出现在 URL 中，但不会被包括在 HTTP 请求中，对后端完全没有影响，因此改变 hash 不会重新加载页面。

history模式：利用了 HTML5 History Interface 中新增的 pushState() 和 replaceState() 方法。（需要特定浏览器支持）
这两个方法应用于浏览器的历史记录栈，在当前已有的 back()、forward()、go() 方法的基础之上，这两个方法提供了对历史记录进行修改的功能。当这两个方法执行修改时，只能改变当前地址栏的 URL，但浏览器不会向后端发送请求，也不会触发popstate事件的执行

因此可以说，hash 模式和 history 模式都属于浏览器自身的特性，Vue-Router 只是利用了这两个特性（通过调用浏览器提供的接口）来实现前端路由.
vue中的router有两种模式：hash模式（默认）、history模式（需配置mode: 'history'）

vue中的hash模式
即地址栏 URL 中的 # 符号,这个#就是hash符号，中文名哈希符或锚点
比如这个 URL：<http://www.baidu.com/#/home，hash> 的值为 #/home
它的特点在于：hash 虽然出现在 URL 中，但不会被包括在 HTTP 请求中，对后端完全没有影响，因此改变 hash 不会重新加载页面。

路由的哈希模式其实是利用了window.onhashchange事件，也就是说你的url中的哈希值（#后面的值）如果有变化，就会自动调用hashchange的监听事件，在hashchange的监听事件内可以得到改变后的url，这样能够找到对应页面进行加载

```javascript
window.addEventListener('hashchange', () => {
   // 把改变后的url地址栏的url赋值给data的响应式数据current，调用router-view去加载对应的页面
   this.data.current = window.location.hash.substr(1)
})
```

vue中history模式
HTML5 History Interface 中新增的两个神器 pushState() 和 replaceState() 方法（需要特定浏览器支持），用来完成 URL 跳转而无须重新加载页面，不过这种模式还需要后台配置支持。因为我们的应用是个单页客户端应用，如果后台没有正确的配置，就需要前端自己配置404页面。

pushState() 和 replaceState() 这两个神器的作用就是可以将url替换并且不刷新页面，好比挂羊头卖狗肉，http并没有去请求服务器该路径下的资源，一旦刷新就会暴露这个实际不存在的“羊头”，显示404（因为浏览器一旦刷新，就是去真正请求服务器资源）

那么如何去解决history模式下刷新报404的弊端呢，这就需要服务器端做点手脚，将不存在的路径请求重定向到入口文件（index.html），前后端联手，齐心协力做好“挂羊头卖狗肉”的完美特效

pushState方法、replaceState方法，只能导致history对象发生变化，从而改变当前地址栏的 URL，但浏览器不会向后端发送请求，也不会触发popstate事件的执行

popstate事件的执行是在点击浏览器的前进后退按钮的时候，才会被触发

```javascript
window.addEventListener('popstate', () => {
  this.data.current = window.location.pathname
})
```

使用场景
一般场景下，hash 和 history 都可以，除非你更在意颜值，# 符号夹杂在 URL 里看起来确实有些不太美丽。

如果不想要很丑的 hash，我们可以用路由的 history 模式，这种模式充分利用 history.pushState API 来完成URL 跳转而无须重新加载页面。 Vue-router 另外，根据 Mozilla Develop Network 的介绍，调用 history.pushState() 相比于直接修改 hash，存在以下优势:

pushState() 设置的新 URL 可以是与当前 URL 同源的任意 URL；而 hash 只可修改 # 后面的部分，因此只能设置与当前 URL 同文档的 URL
pushState() 设置的新 URL 可以与当前 URL 一模一样，这样也会把记录添加到栈中；而 hash 设置的新值必须与原来不一样才会触发动作将记录添加到栈中
pushState() 通过 stateObject 参数可以添加任意类型的数据到记录中；而 hash 只可添加短字符串
pushState() 可额外设置 title 属性供后续使用

总结
传统的路由指的是：当用户访问一个url时，对应的服务器会接收这个请求，然后解析url中的路径，从而执行对应的处理逻辑。这样就完成了一次路由分发

而前端路由是不涉及服务器的，是前端利用hash或者HTML5的history API来实现的，一般用于不同内容的展示和切换

# 另一番解释

在了解路由模式前，我们要先清楚，vue-roter 的实现原理是怎样的，什么是单页面应用，特点是什么，这样更容易加深对路由的理解。

SPA 单页面及应用方式:单一页面应用程序，只有一个完整的页面；它在第一次加载页面时,就将唯一完整的 html 页面和所有其余页面组件一起下载下来，这样它在切换页面时，不会加载整个页面，而是只更新某个指定的容器中内容。

单页面应用(SPA)的核心之一是: 更新视图而不重新请求页面。

路由器对象底层实现的三大步骤即(1)监视地址栏变化；(2)查找当前路径对应的页面组件；(3)将找到的页面组件替换到 router-vieW 的位置。

vue-router 在实现单页面前端路由时，提供了两种方式：Hash 模式和 History 模式；vue2 是根据 mode 参数来决定采用哪一种方式，vue3 则是 history 参数，下面我们将围绕这个属性进行进一步了解。

## hash

vue-router 默认 hash 模式 —— 使用 URL 的 hash 来模拟一个完整的 URL，于是当 URL 改变时，页面不会重新加载。  hash（#）是 URL 的锚点，代表的是网页中的一个位置，单单改变 # 后的部分，浏览器只会滚动到相应位置，不会重新加载网页，也就是说 # 是用来指导浏览器动作的，对服务器端完全无用，HTTP 请求中也不会不包括 # ，同时每一次改变 # 后的部分，都会在浏览器的访问历史中增加一个记录，使用 "后退" 按钮，就可以回到上一个位置，所以说 hash 模式通过锚点值的改变，根据不同的值，渲染指定 DOM 位置的不同数据。

> "#" 符号本身以及它后面的字符称之为 hash，可通过 window.location.hash 属性读取。

特点

hash 虽然出现在URL中，但不会被包括在 HTTP 请求中。它是用来指导浏览器动作的，对服务器端完全无用，因此，改变 hash 不会重新加载页面
可以为 hash 的改变添加监听事件：

javascript复制代码   window.addEventListener("hashchange", fncEvent, false)

每一次改变 hash（window.location.hash），都会在浏览器的访问历史中增加一个记录
url 带一个 # 号。

## history

history 是路由的另一种模式，由于 hash 模式会在 url 中带#，如果不想要带 #的话，我们可以使用路由的 history 模式，只需要在响应的 router 配置规则时,加上即可，vue 的路由默认是 hash 模式。
利用了HTML5 History Interface中新增的 pushState() 和 replaceState() 方法。
这两个方法应用于浏览器的历史记录栈，在当前已有的 back、forward、go 的基础之上，它们提供了对历史记录进行修改的功能。只是当它们执行修改时，虽然改变了当前的 URL，但浏览器不会立即向后端发送请求。
