---
cover: /img/vue.jpg
---

# vue 中，无缓存/缓存场景下组件定时器的销毁问题

1. 无缓存场景

```
  data() {
        return {
            timer: null
        }
    },
    created() {
        this.timer = setInterval(..., 1000);
    },
    beforeDestroy() {
        clearInterval(this.timer);
    }
```

但是这种方式需要在 data 中存储 timer 变量，看着一点不优雅。
我们希望最好的方式是不额外添加任何的杂物，同时可以实现自动清除定时器的功能。
在这里我们可以使用生命周期钩子去做这一操作。

> vm.\$once( event, callback ), 用于监听一个自定义事件，触发一次然后销毁。

我们可以监听到 beforeDestroy，用于清除定时器，
看似$once已经很优雅地解决了一切，但这里需要注意，vm.$once 重复监听，不会覆盖，即这段代码执行了多少次，他就会触发多少次。

所以如果页面存在多次清除和开启定时器的操作时，
比如登录页面的发送验证码按钮，有 60s 的等待间隔。

这种场景下我们最好在定时器第一次初始化的时候去绑定监听，而不是把绑定监听和初始化定时器的逻辑写到一个函数里然后调用。

```
  created() {
        const timer = setInterval(..., 1000);
        this.$once('hook:beforeDestroy', () => {
            clearInterval(timer);
        })
    },
    beforeDestroy() {
        clearInterval(timer);
    }
```

2. keep-alive 场景
   > 当我们用<keep-alive> 包裹组件时，路由切换，会缓存不活动的组件，而不是销毁它们。

说白了就是 vue 帮我们做了一个组件的缓存。

想象一下，我们的项目使用了<keep-alive>包裹路由，有多个页面都用到了上文中的组件，

```
App.vue
<keep-alive>
    <router-view />
</keep-alive>
```

```
Page1.vue
<template>
    <child />
</template>
```

```
Page2.vue
<template>
    <child />
</template>
```

```
child.vue
  <template>
    ...
  </template>
  <script>
    export default {
      name: "child",
      data() {
        return {
          ...
        }
      },
      created() {
          const timer = setInterval(..., 1000);
          this.$once('hook:beforeDestroy', () => {
              clearInterval(timer);
          })
      },
      beforeDestroy() {
          clearInterval(timer);
      }
    }
</script>
```

那么当我们切换页面时，上一个页面的组件并没有销毁，自然而然地就没有触发监听，所以定时器依然存在。

### 解决思路：

1. 由于子组件内部无法使用路由守卫（beforeRouteEnter，beforeRouteLeave），
   所以我们改用 watch 监听\$route，每次路由切换到其他页面时，销毁定时器，页面返回时再重新创建。
2. 在 data()中添加 timer。作为页面进入或离开的条件判断，
   每次路由变化，如果 timer 存在，表示要离开当前页面，如果 timer 为 null，表示要回到当前页面。

### 解决方案：

```
 watch: {
      $route() {
        if(timer) {
            clearInterval(this.timer);
            this.timer = null;
        } else {
            this.timer = setInterval(...);
        }
      }
    }
```

这里我们再考虑一下，每次路由变化时，有以下情况，

1. 切换到其他页面
2. 进入当前页面
   2-1. 初始化
   2-2. 由其他页面返回
   所以我们需要在组件创建时便执行一次，于是我们又添加了 immediate: true

```
  watch: {
      $route: {
        immediate: true, // 页面首次进入时触发
        handler: function () {
            if(timer) {
                clearInterval(this.timer);
                this.timer = null;
            } else {
                this.timer = setInterval(...);
            }
        }
      }
    },
```

上文我们提到，vm.$once会重复触发，于是这里我们不能再通过$once 监听了，直接在 beforeDestroy 中销毁即可，
最后代码如下，

```
  data() {
      return {
        timer: null,
        ...
      }
    },
    methods: {
      initInterval() {
        this.timer = setInterval(..., 1000);
      },
      destroyInterval() {
        clearInterval(this.timer);
        this.timer = null;
      }
    },
    beforeDestroy() {
      this.destroyInterval();
    },
    watch: {
      $route: {
        immediate: true,
        handler: function () {
          if (this.timer) {
            this.destroyInterval();
          } else {
            this.initInterval();
          }
        }
      }
    },
```

### 总结

1. this.\$once 用于监听一个自定义事件，触发一次然后销毁，但是注意只监听一次即可，不要重复监听。
2. 在缓存场景下，考虑到多页面内调用组件时，可以利用 watch 和 data()中的条件变量来对定时器进行操作，页面进入时创建，页面离开时销毁。
