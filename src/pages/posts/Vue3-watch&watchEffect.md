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

### watch的三个参数

```javascript
watch(id, async (newId, oldId, onCleanup) => {
const { response, cancel } = doAsyncWork(newId)
// 当 `id` 变化时，`cancel` 将被调用，
// 取消之前的未完成的请求
onCleanup(cancel)
data.value = await response
})
```

### watch的返回值
```javascript
const stop = watch(source, callback)

// 当已不再需要该侦听器时：
stop()
```


### watchEffect的参数

```javascript
watchEffect(async (onCleanup) => {
  const { response, cancel } = doAsyncWork(id.value)
  // `cancel` 会在 `id` 更改时调用
  // 以便取消之前
  // 未完成的请求
  onCleanup(cancel)
  data.value = await response
})
```

onCleanUp是在副作用重新执行之前，调用的清理函数。执行例如定时器，清空状态之类的。

### flush参数
决定了副作用函数的执行时机
- pre
  - 默认，副作用函数会在DOM更新前执行
- post
  - 副作用函数会在DOM更新后执行
- sync
  - 副作用函数会同步进行，即响应依赖变更的瞬间执行。通常是需要立即响应变化作出相应动作的场景。
  - 对性能要求极高，如果存在多个数据同时更新，会出现数据一致性的问题。