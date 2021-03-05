---
cover: /img/react.jpg
---

移动端中，用户访问了一个列表页，上拉浏览列表页的过程中，随着滚动高度逐渐增加，数据也将采用触底分页加载的形式逐步增加，列表页浏览到某个位置，用户看到了感兴趣的项目，点击查看其详情，进入详情页，从详情页退回列表页时，需要停留在离开列表页时的浏览位置上

类似的数据或场景还有已填写但未提交的表单、管理系统中可切换和可关闭的功能标签等，这类数据随着用户交互逐渐变化或增长，这里理解为状态，在交互过程中，因为某些原因需要临时离开交互场景，则需要对状态进行保存
umi项目的实践
安装

npm install umi-plugin-keep-alive --save
# or
yarn add umi-plugin-keep-alive
从 umi 中导出 KeepAlive，包裹在需要被缓存的组件上

import { useState } from 'react'
import { KeepAlive } from 'umi'

function Counter() {
  const [count, setCount] = useState(0)

  return (
    <div>
      <p>count: {count}</p>
      <button onClick={() => setCount(count => count + 1)}>add</button>
    </div>
  )
}

export default function() {
  const [show, setShow] = useState(true)

  return (
    <div>
      <h1>Page index</h1>
      {show && (
        <KeepAlive>
          <Counter />
        </KeepAlive>
      )}
      <button onClick={() => setShow(show => !show)}>toggle</button>
    </div>
  )
}


注意！一定要包裹住整个组件，可将当前页面取名后再包裹一层进行封装。否则usestate中的值依然无法缓存