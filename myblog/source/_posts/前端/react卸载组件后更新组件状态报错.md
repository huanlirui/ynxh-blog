解决react hook组件卸载数据处于加载中渲染组件状态Can't perform a React state update on an unmounted component. This is a no-op, but it indicates a memory leak in your application. To fix, cancel all subscriptions and asynchronous tasks in a useEffect cleanup function.


报错提示：无法对未装载的组件执行反应状态更新。这是一个不准的操作，但它表示应用程序内存泄漏。若要修复，请取消useffect清理函数中的所有订阅和异步任务

原因：组件卸载了，但是仍处于渲染数据状态（如：setState，useState），一般写定时器时候会有出现。其他情况也会，只要组件卸载但仍在更新数据时机

解决如下：

这个错误不大会影响实际使用，解决非常简单。

先讲下hook函数组件的解决：

第一种定时器情况：

```
const [update, setUpdate] = useState(1);
 useEffect(() => {
    const creatInt = setInterval(() => {    //假设这里写了定时器来更新update
      setUpdate(c => c + 1);
    }, 2000);
return () => {
      clearInterval(creatInt);   //（重点）这里清除掉定时器  
    };
  }, []);
```

第二种useSate情况：
```
useEffect(() => {
    let isUnmount = false;      //这里插入isUnmount
    const fetchDetail = async () => {
      const res = await getDetail(detailId);
      if (res.code === 0 && !isUnmount) {  //加上判断isUnmount才去更新数据渲染组件
        setDetail(res.data);
      }
    };
    fetchDetail();
    return () => isUnmount = true;   //最好return一个isUnmount
  }, [detail]);
```

说明：useEffect相当于class组件的3个生命周期，其中return ()=>{ }  相当于 componentWillUnMount

class类组件：原理同hook在 componentWillUnMount 中设置一个 值true 、false来判断是否渲染组件