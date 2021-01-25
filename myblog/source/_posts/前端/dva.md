---
toc:
enable: true
---

# dva 再回顾

### 前言
最近看了大神的前端代码，发现自己写得东西太混乱了，ui 组件中夹杂着数据处理，业务逻辑。代码可读性较差，且代码臃肿。看到大神用的 dva！果断跟！

学 react 时为了状态管理，看了 redux，只有一个感受：麻烦！！ 后来了解到 dva，只用 connect 一下，就能轻松把 model 和组件关联起来，公用状态。然而，当时的我仅仅领悟到了它的使用方法，未曾想过什么时候该用，也未曾深入了解 dva 仅有的几个 api 的作用和设计思想(主要是我看不懂 hhhh)。 这次，重新认识 dva!

### 首先认识一下 model 文件

dva 通过 model 的概念把一个领域的模型管理起来，包含同步更新 state 的 reducers，处理异步逻辑的 effects，订阅数据源的 subscriptions 。

> namespace 表示在全局 state 上的 key 我理解为每个 model 文件的 key。通过 namespace 可以拿到该 model 文件的 state

```javascript
export default {
  namespace: 'home',
  state: {
    data: {},
  },
  reducers: {
    save: (state, { payload }) => {
      state.data = payload;
    },
  },
  effects: {
    *getIndex(action, { call, put }) {},
  },
  subscriptions: {
    keyEvent({ dispatch }) {
      key('⌘+up, ctrl+up', () => {
        dispatch({ type: 'add' });
      });
    },
  },
};
```

## dva 中有 6 个 api

1. State

   每个模型就像组件一样，拥有一个状态库，保存了与之关联的状态，但是这个状态是全局的。可以在需要使用的时候，通过 dva 的 connect 方法，绑定 state 到某个视图组件 view

2. Action

   Action 是用来描述 UI 层事件的一个对象。它是改变 State 的唯一途径。无论是从 UI 事件、网络回调，还是 WebSocket 等数据源所获得的数据，最终都会通过 dispatch 函数调用一个 action，从而改变对应的数据。action 必须带有 type 属性指明具体的行为，其它字段可以自定义，如果要发起一个 action 需要使用 dispatch 函数；需要注意的是 dispatch 是在组件 connect Models 以后，通过 props 传入的。

dispatch()中的{}参数。即可理解为 Action

3.  dispatch

        在 dva 中，connect Model 的组件通过 props 可以访问到 dispatch，可以调用 Model 中的 Reducer 或者 Effects

    dipatch 可以看作是触发这个行为的方式

```javascript
dispatch({
  type: 'user/add', // 如果在 model 外调用，需要添加 namespace
  payload: {}, // 需要传递的信息
});
```

4. Reducer

   在 dva 中，reducers 聚合积累的结果是当前 model 的 state 对象。通过 actions 中传入的值，与当前 reducers 中的值进行运算获得新的值（也就是新的 state）。需要注意的是 Reducer 必须是纯函数，所以同样的输入必然得到同样的输出，它们不应该产生任何副作用。

```javascript
 reducers: {
    save: (state, { payload }) => {
      state.data = payload;
    },
  }
```

如上述例子所示，save 就是一个 reducer 函数，接受 2 个参数，1 个是当前的 state 对象，一个是将要改变的值。
我理解为 react 组件中的。setState 方法

5. Effect

   Effect 被称为副作用，在我们的应用中，最常见的就是异步操作。它来自于函数编程的概念，之所以叫副作用是因为它使得我们的函数变得不纯，同样的输入不一定获得同样的输出。

   > 这里为了将异步写法改为同步写法，参考了[generator 的相关概念](http://www.ruanyifeng.com/blog/2015/04/generator.html)

   通常在业务处理，数据处理时，通过 ui 组件直接 dispatch 该处的放方法。

```javascript
function* addAfter1Second(action, { put, call }) {
  yield call(delay, 1000);
  yield put({ type: 'add' });
}
```

   > Effect 是一个 Generator 函数，内部使用 yield 关键字，标识每一步的操作（不管是异步或同步）。我把函数前的*号和yield理解为：async 和 await

call 和 put

dva 提供多个 effect 函数内部的处理函数，比较常用的是 call 和 put。
> call：执行异步函数.
> put：发出一个 Action，类似于 dispatch

6. Subscription

    未完。待更新