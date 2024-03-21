---
layout: '../../layouts/MarkdownPost.astro'
title: "react(四)-hooks闭包陷阱"
pubDate:  "2024-03-21"
description: "react(四)-hooks闭包陷阱"
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

## react(四)-hooks闭包陷阱及原因

### 怎么产生闭包的

```typescript
import React, {useEffect, useState} from 'react';

function App() {
  const [count, setCount] = useState(0)

  useEffect(() => {
    setInterval(() => {
      console.log('count', count)
      setCount(count + 1)
    },2e3)
  }, [])
  return (
    <div>{count}</div>
  );
}

export default App;
```
setCount(count + 1)依赖的是count，而count引用的是当时的count，也就是初始值0的时候，所以count永远是0，setCount(count + 1)永远是setCount(0 + 1)

### useReducer来解决闭包陷阱

```typescript
import {useReducer,Reducer,useEffect} from "react";

interface Action{
    type:'add' | 'minus';
    num:number;
}
function reducer(state:number,action:Action){
    switch(action.type){
        case 'add':
            return state + action.num;
        case 'minus':
            return state - action.num;
        default:
            return state;
    }
}
function App (){
    /**
     * useReducer返回两个参数
     * state:当前状态
     * dispatch:dispatch方法
     * useReducer接收一个reducer函数和初始状态
     * Reducer<number, Action>是ts的类型声明，表明reducer函数的参数是number和Action类型
     */
    const [state,dispatch] = useReducer<Reducer<number, Action>>(reducer,1000)

    useEffect(() => {
        const interval = setInterval(() => {
            dispatch({type:'minus',num:7}) // 1000 减 7 等于多少
        },1000);
        return () => { // 清除定时器
            clearInterval(interval);
        }
    }, []);

    return (
        <div>
            {state}
        </div>
    )
}
export default App;
```

### useState解决闭包陷阱
```typescript
import {useEffect, useState} from "react";

function App() {
    const [count, setCount] = useState(0)


    useEffect(() => {
        console.log(count)
        const interval = setInterval(() => {
            // 使用函数式更新，这样就不依赖于外部的 `count` 变量
            setCount(count => count + 1)
        }, 1000)
        return () => {
            clearInterval(interval)
        }
    }, []);

    return (
        <div>{count}</div>
    )
}

export default App;
```
这里的console.log(count)始终都是0。因为引用的是声明变量时候的初始值0。

所以修改deps数组为count，count变化的时候重新执行useEffect，执行的函数就引用的是最新的count值。

