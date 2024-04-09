---
layout: '../../layouts/MarkdownPost.astro'
title: "Vue3-toRefs&toRef,toRaw&unref,expose&emit"
pubDate:  "2024-04-09"
description: "Vue3-toRefs&toRef,toRaw&unref,expose&emit"
author: '保安'
cover:
  url: '../img/cc.jpg'
  square: '../img/cc.jpg'
  twitter: '../img/cc.jpg'
  alt: 'cover'
tags: [ "Vue" ]
theme: 'light'
featured: true
---

![封面|wide](/img/cc.jpg)[1]

## Vue3-toRefs&toRef,toRaw&unref,expose&emit

### toRefs&toRef

toRefs用于结构整个reactive和props的属性。因为toRefs是作用于响应式对象（reactive）和props。

为什么props也算呢，因为可以在父组件用reactive或者ref来**创建相应式对象**。但是props本身就是响应式。因为父组件修改了，子组件会响应。

解构Ref

```javascript
const personal = ref({
  name: "mafeifei",
  age: 18,
  sex: "man"
});

const { name } = toRefs(personal.value);
console.log("name", name); //ObjectRefImpl {_object: Proxy(Object), _key: 'name', _defaultValue: undefined, __v_isRef: true}
console.log("name.value", name.value); // mafeifei
console.log("name isRef", isRef(name)); // true
```

toRef用于只需要相应式对象里的一个属性。
```javascript
const age = toRef(personal.value, "age");
console.log("age", age); // ObjectRefImpl {_object: Proxy(Object), _key: 'age', _defaultValue: undefined, __v_isRef: true}
console.log("age.value", age.value); // 18
console.log("age isRef", isRef(age)); // true
```
当然也可以这样，创建相应式的
```javascript
const age = computed(() => personal.value.age)
```

### toRaw&unref

toRaw并不是深拷贝，而会修改引用数据类型的值。
toRaw会返回数据的原始数据类型。
```javascript
const raw = toRaw(personal.value);
console.log("personal.value", personal.value); // Proxy(Object) {name: 'mafeifei', age: 18, sex: 'man'}
console.log("raw", raw); // {name: 'mafeifei', age: 18, sex: 'man'}
raw.age = 999;
console.log("raw end", raw); // {name: 'mafeifei', age: 999, sex: 'man'}
console.log("personal.value end", personal.value); // Proxy(Object) {name: 'mafeifei', age: 999, sex: 'man'}

```

### unref
unref用于显示ref原始值。用于迫切想知道ref原始值的情况。

如果参数是ref，则返回内部值，否则返回参数本身。
```javascript
val = isRef(val) ? val.value : val
```

```javascript
const article = ref("v2qun");
console.log("article", article); //RefImpl {__v_isShallow: false, dep: undefined, __v_isRef: true, _rawValue: 'v2qun', _value: 'v2qun'}
console.log("unref(article)", unref(article)); // v2qun
```


```javascript
const un = unref(personal.value);
console.log("un", un); //Proxy(Object) {name: 'mafeifei', age: 18, sex: 'man'}
console.log("unref(personal.value)", unref(personal.value));
```

这里un打印失效，没有打印出预期值。**因为unref，是从相应式引用里获取内部值**，而**personal是一个相应式对象**。
如果unref传入的是一个相应式对象，那么就会返回这个相应式对象。

### expose&emit

expose暴露给父组件该组件的属性和方法
```vue
<chidComponents ref="chidRef"></chidComponents>

childRef.value.open() // 方法
childRef.value.list // 属性
```

emit用于发布订阅事件，子组件处理好某些事件之后，主动给父组件发消息，告诉子组件有新的派发。

而expose是直接调用子组件的方法和属性，并没有订阅关系。