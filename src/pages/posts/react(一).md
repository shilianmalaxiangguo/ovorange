---
layout: '../../layouts/MarkdownPost.astro'
title: 'react(一)'
pubDate: 2024-03-15
description: 'react(一)'
author: '保安'
cover:
  url: '../img/cc.jpg'
  square: '../img/cc.jpg'
  twitter: '../img/cc.jpg'
  alt: 'cover'
tags: [ "react" ]
theme: 'dark'
featured: true
---

![封面|wide](/img/cc.jpg)[1]

## react(一)

### useState

像123、"mafeifei"、{name:"55kai"}这种叫数据，他们可以是各种数据类型，数据本身都是一个内存存储的不变的数据。

但是从一种数据变成另一种数据，这就是状态了(state)。

组件的核心就是状态。 **UI = f(data)**，这里的UI就可以指代组件。

```javascript
function App() {
    function initNum () {
        const num1 = 1 + 2;
        const num2 = 3 + 4;
        return num1 + num2;
    }
    const [num, setNum] = useState(initNum()) // useState可以传入一个函数，return你所需要的默认值
    // 但是这个函数只能是同步的

    return (
        <div onClick={() => setNum( num + 1)}>{num}</div>
    );
}

export default App;
```

### useEffect和useLayoutEffect
useLayoutEffect和useEffect的区别是，useLayoutEffect会在浏览器渲染之前执行，而useEffect会在浏览器渲染之后执行。

useLayoutEffect会在浏览器渲染之前执行，所以它会阻塞浏览器的渲染，所以我们要尽量避免使用useLayoutEffect。
- 请求数据在useEffect中执行
- 获取dom元素的大小和位置、添加滚动事件,resize等监听在useLayoutEffect中执行

```typescript jsx
import {useEffect, useState, useLayoutEffect, useRef} from "react";

async function queryData() {
    return await new Promise<number>((resolve) => {
        setTimeout(() => {
            resolve(998)
        }, 2e3)
    })
}

function App() {

    const [num, setNum] = useState(0)

    // useEffect(() => {
    //     queryData().then((num) => {
    //         setNum(num)
    //     })
    // }, [])

    useEffect(() => {
        console.log('effect')
        const timer = setInterval(() => {
            console.log('num', num)
        }, 1e3)

        return () => {
            console.log('clean up')
            clearInterval(timer)
        }
    }, [num])
    const divRef = useRef(null)
    const [size, setSize] = useState({x: 0, y: 0})
    const [scroll, setScroll] = useState(0)
    useLayoutEffect(() => {
        if (divRef.current) {
            setSize({
                x: divRef.current['offsetWidth'],
                y: divRef.current['offsetHeight']
            })
        }
        const handleScroll = () => {
            setScroll(window.scrollY)
        }
        window.addEventListener('scroll', handleScroll)

        return () => {
            window.removeEventListener('scroll', handleScroll)
        }
    },[])
    return (
        <div>
            <div onClick={() => setNum((prev) => prev + 1)}>click{num}</div>

            <div ref={divRef} style={{height: '120vh', width: '100%', background: 'lightblue'}}>
                mafeifei
                <div>
                    Div size: {size.x}px x {size.y}px
                </div>
                <div style={{margin: '40vw'}}>
                    Scroll position: {scroll}px
                </div>
            </div>

        </div>
    );
}

export default App
```

### useReducer
```typescript jsx
import {Reducer, useReducer} from "react";

interface Data {
    result: number
}

interface Action {
    type: 'add' | 'minus',
    num: number
}

function reducer(state: Data, action: Action): Data {
    switch (action.type) {
        case 'add':
            return {result: state.result + action.num}
        case 'minus':
            return {result: state.result - action.num}
        default:
            throw new Error()
    }

    return state
}

function App() {

    const [state, dispatch] = useReducer<Reducer<Data, Action>>(reducer, {result: 0})

    return (
<div>
            <div onClick={() => dispatch({type: 'add', num: 1})}>add</div>
            <div onClick={() => dispatch({type: 'minus', num: 1})}>minus</div>
            <div>{state.result}</div>
        </div>
    )

}

export default App
```

useReducer的类型参数传入Reducer<数据的类型,action的类型>
第一个参数是函数reducer，第二个参数是初始值。
