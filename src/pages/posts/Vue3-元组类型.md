---
layout: '../../layouts/MarkdownPost.astro'
title: 'Vue3-元组类型'
pubDate: 2024-04-08
description: 'Vue3-元组类型'
author: '保安'
cover:
  url: '../img/cc.jpg'
  square: '../img/cc.jpg'
  twitter: '../img/cc.jpg'
  alt: 'cover'
tags: ["Vue", "TypeScript" ]
theme: 'light'
featured: true
---

![封面|wide](/img/cc.jpg)[1]


## Vue3-元组类型

### 什么是元组

TypeScript中，数组和元组的表示方法都用[]来表示。
- 数组通常表示一个包含多个相同类型元素的集合。
- 元组代表一个**固定长度**的数组，其中每个元素可以是不同类型。

### 数组与元组的区别
- 数组：number[]代表一个number类型的数组
- 元组：[id:number, age:string, sex:number]代表固定元素数量，且每个元素不必相同。**元组中每个元素的类型和出现位置是固定的，一旦定义，不可改变**。

### 为什么在defineEmits中使用元组
在defineEmits中，Vue的事件线程系统允许每个事件event传递一定数量的参数，且这些参数可以是不同类型。

恰恰emits的实际使用上，emit('change',row,index,name)，可以自由传递不同类型的多个参数，刚好与元组的定义高度重合。

emit与元组，好比干柴烈火，niko配上了沙鹰，simple配上了龙狙，步步高配上打火机。

所以，在defineEmits的上下文中，用[]定义元组，这元组代表参数list。

比如
```typescript
const emit = defineEmits<{
  change: [id: number];
  update: [value: string];
}>();
```
这里：
- change: [id:number] 表示，change事件，应该接受一个参数，病名为id，类型为number。
-  update: [value: string]表示，update事件，应该接受一个参数，key是value，类型为string。

### 为什么不用 change: (id: number)

因为()在TypeScript中通常描述函数的参数列表，而在defineEmits中，我们并不是定义一个函数。而是在定义一个事件，和它的参数类型。

### 调用签名

TypeScript中的调用签名是一种**描述函数或方法类型**的语法。它定义了函数的参数类型和返回类型。通常可以在接口或类型别名中定义调用签名。

一个简单的调用签名语法通常为这样：
```typescript
type BiuBiuBiuFunction = {
    (arg1:string, arg2: number): boolean
}
```

这里定义了一个BiuBiuBiuFunction类型表示一个函数，它接受一个string和一个number的参数，并返回boolean类型的值。

### Vue3.3+可用的语法

```typescript
const emit = defineEmits({
  change: (id: number) => true, // 返回值类型不重要，因为 emit 函数不关心它
  update: (value: string) => true
});
```

emit内的返回类型，emit函数并不会使用它，但是这可以让TypeScript推断出事件的参数类型。


### 元组的高级用法

比如剩余元素（rest elements）、可选元素。列入：
```typescript
let user: [string, number, boolean?]
user = ['mafeifei',18] // 正确，第三个元素是可选的
user = ['mafeifei',true] // 也正确，第三个元素类型可不确定

let numbers: [number, ...number[]]
numbers = [1] // 正确
numbers = [1, 2, 3, 4, 5] // 也正确，剩余元素可以是任意数量的number类型
```

在这些例子中，boolean?表示可选元素, ...number[]表示**剩余任意数量**的元素。
元组的特点就是，它在处理需要固定长度和不同类型元素的数组时十分有效。

.