---
layout: '../../layouts/MarkdownPost.astro'
title: 'Vue3-watch&watchEffect'
pubDate: 2024-04-16
description: 'Vue3-watch&watchEffect'
author: '保安'
cover:
  url: '../img/cc.jpg'
  square: '../img/cc.jpg'
  twitter: '../img/cc.jpg'
  alt: 'cover'
tags: ["Vue" ]
theme: 'light'
featured: true
---

![封面|wide](/img/cc.jpg)[1]


## Vue3-watch&watchEffect

### watch

```javascript
export function useFetch(url) {
  let data = ref(null)
  let error = ref(null)
  const requset = () => {
    fetch(url).then(res => res.json).then(json => {
      data.value = json
    }).catch(e => {
      error.value = e
    })
  }

  // useFetch() 接收一个静态 URL 字符串,
  // 作为输入——因此它只会执行一次 fetch 并且就此结束。
  // 如果我们想要在 URL 改变时重新 fetch 呢？
  
  // 第一个方案
  watch(toRef(ref).value, () => {
    requset()
  }, {
    immediate: true // 立即执行一次
  })
  
  // 第二个方案
  const requset2 = () => {
    // toValue() 是一个在 3.3 版本中新增的 API。
    // 它的设计目的是将 ref 或 getter 规范化为值。
    // 如果参数是 ref，它会返回 ref 的值；
    // 如果参数是函数，它会调用函数并返回其返回值。
    // 否则，它会原样返回参数。
    fetch(toValue(url)).then(res => res.json).then(json => {
      data.value = json
    }).catch(e => {
      error.value = e
    })
  }
  
  watchEffect(() => {
    requset2()
  })
  
  
  return {
    data, error
  }
}
```

(未更完)
