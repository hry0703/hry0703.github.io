---
title: reactAPI
date: 2020-05-03 11:14:37
categories: react
tags: react
---
## memo
memo 用来对**函数**组件进行缓存，父组件传的props未改变时，不会重新渲染该子组件，与class组件的PureComponent类似，
默认情况下其只会对复杂对象做浅层对比，可以传递一个对比函数作为第二个参数，来进行深层次的对比。
```javascript
  function MyComponent(props) {
    /* 使用 props 渲染 */
  }
  function areEqual(prevProps, nextProps) {
    /*
    如果把 nextProps 传入 render 方法的返回结果与
    将 prevProps 传入 render 方法的返回结果一致则返回 true，
    否则返回 false
    */
  }
  export default React.memo(MyComponent, areEqual);
```

<!-- more -->

举个例子：
```javascript
// 点击Re-render不会引起ReactMemo组件的重新渲染
// 子组件
import React, { memo } from 'react'
function ReactMemo(props){
  console.log('ReactMemo渲染了')
  return (
    <>
      <div>
        {`我是props的值:${props.flag}`}
      </div>
    </>
  )
}
export default memo(ReactMemo)
// 父组件
import React, { useState } from 'react';
import ReactMemo from '../ReactMemo/ReactMemo'
export default function Home (){
  const [random, setRandom] = useState(Math.random());
  const [mounted, setMounted] = useState(true);
  // 这个函数改变 random，并触发重新渲染
  const reRender = () => setRandom(Math.random());
  const toggle = () => setMounted(!mounted);
  return (
    <>
      <button onClick={reRender}>Re-render</button>
      <button onClick={toggle}>switch</button>
      <ReactMemo flag={mounted}></ReactMemo>
    </>
  );
}
```
子组件接收方法的情况下，需要配合useCallback 将函数缓存才能真正做到子组件的缓存