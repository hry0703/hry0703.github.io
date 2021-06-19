---
title: reactHooks
date: 2020-05-03 11:11:45
categories: 
tags: react
---

副作用：React 中主要指那些没有发生在数据向视图（M-V）转换过程中的逻辑，如 Ajax 请求、访问原生 DOM 元素、本地持久化缓存、绑定/解绑事件、添加/取消订阅、设置定时器、记录日志等。

Hooks 的优越性：
1. 函数式编程：No class, No super, No this。对于不了解 OOP 的 React 初学者更友好。
2. 有状态逻辑易复用：可以通过 Custom Hook（后面讲解）重构，而不用修改组件结构。
3. 易拆分：状态管理和副作用管理松耦合，原子性强。很容易将一些相关联的逻辑拆分成更小的函数。
4. 可逐步引入：Hooks 向后兼容，与现有代码可并行工作，因此我们可以逐步采用它们。
5. 副作用分组：很多副作用逻辑分散在类组件生命周期函数中。而 Hooks 可以将每个副作用的设置和清理封装在一个函数中。
6. 副作用分离：副作用操作都在页面渲染之后。

## useState
useState用来声明状态变量。接收的参数是我们的状态初始值（initial state），它返回了一个数组，这个数组的第[0]项是当前当前的状态值，第[1]项是可以改变状态值的方法函数。
```javascript
  const [count, setCount] = useState(0);
  // 如果不用es6解构写法就得这么写：
  let _useState = useState(0);
  let count = _useState[0];
  let setCount = _useState[1];
```

**useState是按照书写顺序执行的**，因此为了保证执行顺序一致。useState不能书写在if else 等判断逻辑中
```javascript
  // 错误写法
  let flag = false
  if(!flag){
    const [loading,setLoading] = useState(false)
    flag = true
  }  
  const [datalist,setDataList] = useState([])
```
**useState「粒度」问题**
实际工作中，一个类组件的 this.state 中往往有十几项，用 Hooks 改写的话要写十几个 useState
根据官方文档，总结下来，有几点：
1. 建议将 state 分割为多个 useState。粒度更细，更易于管理，更好复用。
2. 可能一起改变的 state 可合并成一个useState（ 比如Dom元素的 top left）。
3. 当 state 逻辑趋于复杂，建议使用 reducer 或 Custom Hook 管理（后面介绍）。
4. 当组件的 state 很多的时候，为了提高代码的可读性，也可以把逻辑相关的一些 state 合并为一个 useState（ 比如分页参数 ）。但这些 state 并不是一起改变的，所以当其中一个 state 改变，调用对应的 setFunction 的时候。你需要做对象合并(不合并就丢了)：

```javascript
const [ pageData, setPageDate ] = useState({ pageSize: 20, current: 1, total:0, })
const onPageChange = current => {
  // 常规操作
  setPageDate( Object.assign( {}, pageData, { current } ) ）
  // 官方建议
  setPageDate(currentPageData => ({ ...currentPageData, current}));
}
```

## useEffect
useEffect在第一次渲染之后和每次更新之后都会执行。也可以通过第二个参数来控制是否执行，**React 保证了每次运行 effect 的同时，DOM 都已经更新完毕。**
effects 的心理模型跟 componentDidMount 和其他的生命周期是不同的。**应该用 effects 的方式去思考**，而且比起回应生命周期事件，它的心理模型更接近於**執行同步化**
```javascript
  useEffect(() => {
    return ()=>{
      // 组件卸载之前执行 需要做副作用的清理，保证引起不必要内存泄漏。比如，手动绑定事件，订阅，定时器等。
    } 
  },[]);
```
```javascript
// child组件
import React, { useState,useEffect } from 'react';
export default function Child (){
  const [count, setCount] = useState(0)
  useEffect(() => {
    console.log(`mount:${count}`);
    return ()=>{
      console.log(`unmount:${count}`);  
    } 
  });
  return (
    <>
    <p>I'm a lifecycle demo</p>
    <button onClick={()=>{setCount(count + 1)}}>add</button>
    <div>
      {count}
    </div>
    </>
  );
}

// home组件
import React, { useState,useEffect } from 'react';
import './Home.scss';
import Child from '../Child/Child'
export default function Home (){
  // 建立一个状态来切换 LifecycleDemo 的显示和隐藏
  const [mounted, setMounted] = useState(true);
  // 该函数将卸载并重新挂载 LifecycleDemo
  // 在控制台可以看到  unmount 被打印 
  const toggle = () => setMounted(!mounted);
  return (
    <>
      <button onClick={toggle}>Show/Hide LifecycleDemo</button>
      {mounted && <Child/>}
    </>
  );
}
```
**控制useEffect 的执行** 
useEffect(fn, []),第二个参数是个数组，传递的是该useEffect的依赖项。
当传递进去的依赖项变动时，该useEffect会重新执行。
注意：
1. 不传第二个参数，则其他不相关数据变动引起的组件重新渲染时也会重新执行该useEffect；
2. 第二个参数为空数组，表示没有依赖项，该useEffect在组件首次渲染时执行和组件卸载时执行清除副作用之后就不会再执行，（诚实传递依赖项，不要漏写，以免引起未知问题）；

**与 componentDidMount 和 componentDidUpdate 的区别**
使用 useEffect 调度的副作用不会阻塞浏览器更新屏幕。这使得应用感觉上具有响应式。大多数副作用不需要同步发生。如果需要同步进行，（比如测量布局），有一个单独的 useLayoutEffect Hook， API 和 useEffect 相同。

**知识点**：调用 useState 的更新函数时，可以传一个箭头函数，这个函数的参数是当前最新的 state， 返回值是要设置的 state,可以配合useEffect使用，来减少useEffect的依赖项。
```javascript
import React, { useState, useEffect } from 'react';
function App() {
  // 秒表开关
  const [isOn, setIsOn] = useState(false);
  // 计数
  const [timer, setTimer] = useState(0);
  useEffect(() => {
    let interval;
    //开关打开的时候才执行
    if (isOn) {
      // 通过定时器增加计数
      interval = setInterval(
        () => setTimer(timer + 1), // 这里每次需要获取定时器每次的最新值，所以这种写法，useEffect必须添加timer为依赖项
        // () => setTimer(val => val + 1),// 用箭头函数的方式，参数就是最新的定时器值，这种写法可以减少依赖项，较优
        1000,
      );
    }
    // 需要清除定时器
    // 不清理会如何？codesandbox中尝试，页面直接卡死
    return () => clearInterval(interval);
  },[isOn,timer]); 

  return (
    <>
      <p>{timer}</p>
      {!isOn && (
        <button type="button" onClick={() => setIsOn(true)}>
          Start
        </button>
      )}
      {isOn && (
        <button type="button" onClick={() => setIsOn(false)}>
          Stop
        </button>
      )}
    </>
  );
}
export default App;
```
**实际项目中axios的使用** 
useEffect 不能接收 async 作为执行函数。useEffect 接收的函数，要么返回一个能清除副作用的函数，要么就不返回任何内容。而 async 返回的是 promise。
useEffect 调用的函数如果依赖 state 或者 props。最好在执行函数中定义。这样依赖容易追踪。
```javascript
import React, { useState, useEffect } from 'react';
import axios from 'axios';
function App() {
  const [data, setData] = useState([]);
  useEffect(() => {
    // 更优雅的方式
    const fetchData = async () => {
      const result = await axios(
        'https://hn.algolia.com/api/v1/search?query=redux',
      );
      setData(result.data);
    };
    fetchData();
  }, []);
  return (
    <ul>
      {data.hits.map(item => (
        <li key={item.objectID}>
          <a href={item.url}>{item.title}</a>
        </li>
      ))}
    </ul>
  );
}
export default App;
```

## useContext
多级嵌套组件之间共享状态，可以使用useContext。
```javascript
import React, { useContext} from 'react';
const data = {
  age:118,
  action:function(){
    console.log('this is action')
  },
}
const dataContext = React.createContext(data.age);  // 1.
export default function ContextCo (){
  return (                                          // 2.
    <dataContext.Provider value={data.age}>         
      <Son />
    </dataContext.Provider>
  );
}
function Son(){
  return (
    <GraSon>
    </GraSon>
  )
}
function GraSon(){
  const data = useContext(dataContext);            // 3.
  console.log(data)   //  118
  return (
    <>
      this is GraSon
    </>
  )
}
```

## useRef
useRef 不仅仅是用来管理 DOM ref 的(场景一)，同时也会用来存放组件实例的属性（场景三）。
useRef 返回一个可变的ref对象，其.current属性被初始化为传入的参数（initialValue）。**返回的 ref 对象在组件的整个生命周期内保持不变,是相同的引用**（createRef 每次渲染都会返回一个新的引用）,因此 **useRef 的内容发生变化时,它不会通知您**。更改.current属性不会导致重新呈现。 因为他一直是一个引用。

```javascript
// 场景一：
import { useRef } from 'react';
export default function Home (){
  const inputEl = useRef(null);
  const onButtonClick = () => {
      // `current` 指向已挂载到 DOM 上的文本输入元素
      inputEl.current.focus();  // input框聚焦
  };
  return (
    <>
      <input ref={inputEl} type="text" />
      <button onClick={onButtonClick}>Focus the input</button>
    </>
  );
}
```
**useRef可以获取到实时状态**
```javascript
// 场景二：点击lert按钮时的count值，是点击alert动作触发时的count值快照，不是定时器触发后的实时值，
import React, { useState,useEffect } from 'react';
export default function Child (){
  const [count, setCount] = useState(0)
  const log = ()=>{
    setTimeout(()=>{
      console.log(count);
    },3000)
  }
  return (
    <>
      <p>计时器 </p>
      <button onClick={()=>{setCount(count + 1)}}>add</button>
      <button onClick={log}>print</button>  
      <div>
        {count}
      </div>
    </>
  );
}
// 上述场景如何取到定时器触发时实时的状态？ 需要使用uesRef
// 场景三：
import React, { useState, useEffect, useRef } from 'react';
export default function Child (){
  const [count, setCount] = useState(0)
  const latestCount = useRef(count)
  useEffect(()=>{
    latestCount.current = count 
  })
  const log = ()=>{
    setTimeout(()=>{
      console.log(latestCount.current);
    },3000)
  }
  return (
    <>
      <p>计时器 </p>
      <button onClick={()=>{setCount(count + 1)}}>add</button>
      <button onClick={log}>print</button>
      <div>
        {count}
      </div>
    </>
  );
}
```

## useMemo 
useMemo返回的是数据，用来缓存数据，当 组件内部某一个渲染的数据，需要通过计算而来，这个计算是依赖与特定的state、props数据，我们就用useMemo来缓存这个数据，以至于我们在修改她们没有依赖的数据源的情况下，多次调用这个计算函数，浪费计算资源。

```javascript
import React, { useState, useEffect, useRef, useMemo,useCallback } from 'react';
export default function ReactUseCallback (){
  const [gender,setGender] = useState('male')
  const [count, setCount] = useState(0)
  const latestCount = useRef(count)
  useEffect(()=>{
    latestCount.current = count 
  })
  const log = ()=>{
    setTimeout(()=>{
      console.log(latestCount.current);
    },3000)
  }
  function judgeGender(gender){
    console.log('这里是判断性别的方法')
    return gender === 'male' ? "男":"女"
  }
  // let showGender = judgeGender(gender)  // 这种写法，count修改时，judgeGender方法也会重复调用
  let showGender = useMemo(()=>{   // 依赖项gender不改变则不会重复执行judgeGender方法
    return judgeGender(gender)
  },[gender])
  const change = ()=>{
    console.log('我是父组件传递的方法')
  }
  return (
    <>
      <p>计时器 </p>
      <button onClick={()=>{setCount(count + 1)}}>add</button>
      <p>当前次数：{count}</p>
      <button onClick={log}>print</button>
      <button onClick={()=>setGender('female')}>修改性别</button>
      <p>这里显示性别：{showGender}</p>
    </>
  );
}
```

## useCallback
缓存一个函数，这个函数如果是由父组件传递给子组件，或者自定义hooks里面的函数【通常自定义hooks里面的函数，不会依赖于引用它的组件里面的数据】，这时候我们可以考虑缓存这个函数，好处：

1，不用每次重新声明新的函数，避免释放内存、分配内存的计算资源浪费
2，子组件不会因为这个函数的变动重新渲染。(和React.memo搭配使用)


## useReducer

## useImperativeHandle

## Custom Hooks
自定义 Hook 是一个 JavaScript 函数，其名称以 ”use” 开头，可以调用其他 Hook
**Custom Hooks 将解决有状态（stateful）逻辑共享的问题（相当于类组件中Hoc的功能）**
下面是判断设备是否离线的自定义hook:
```javascript
import React, { useState, useEffect } from 'react';
// 自定义 hook
function useOffline() {
  const [isOffline, setIsOffline] = useState(window.navigator.onLine);
  function onOffline() {
    setIsOffline(true);
  }
  function onOnline() {
    setIsOffline(false);
  }
  useEffect(() => {
    window.addEventListener('offline', onOffline);
    window.addEventListener('online', onOnline);
    return () => {
      window.removeEventListener('offline', onOffline);
      window.removeEventListener('online', onOnline);
    };
  }, []);
  return isOffline; // 只暴露一个 state
}

// 函数组件
function App() {
  const isOffline = useOffline();
  return (
    <>
       { 
         isOffline
         ? <div>网断已断开 ...</div>
         : <div>网络已连接 ...</div>
       }
    </>
  )
}
export default App;
```