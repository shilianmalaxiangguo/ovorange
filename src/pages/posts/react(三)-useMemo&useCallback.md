---
layout: '../../layouts/MarkdownPost.astro'
title: "react(三)-useMemo&useCallback.md"
pubDate: "2024-03-19"
description: "react(三)-useMemo&useCallback.md"
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

## react(三)-useMemo&useCallback.md

### useMemo

```typescript
    const counter2 = useMemo(function () {
    console.log('counter2运算一次')
    return count * 2
},[deps])
```
useMemo 缓存返回的值，根据deps的依赖来刷新缓存的值

```typescript jsx
import {memo, useMemo, useCallback, useState, useEffect} from "react";

function App() {
    const [count, setCount] = useState(0)

    const memoText = useMemo(() => {
        console.log('this is Date',new Date())
    },[Math.floor(count / 10)])

    useEffect(() => {
        if (count % 10 === 0) {
            // console.log('this is Date', new Date());
        }
    },[count])
    return <div>
        count:{count}
        <button onClick={() => setCount(prev => prev += 1)}>点击btn</button>
    </div>
}

export default App
```

比如我想count等于10的倍数，不包含0。的时候缓存memo里的新返回。为什么不能用count % 10 === 0呢，因为20的时候返回true，21的时候返回false，那么deps发生改变，会触发deps再重新缓存一次。

所以用Math.floor(count / 10)，就保证在20~29之间都是返回2，30~39之间都是返回3。值是在此区间是不变的。





