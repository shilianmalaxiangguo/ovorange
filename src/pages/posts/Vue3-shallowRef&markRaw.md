---
layout: '../../layouts/MarkdownPost.astro'
title: "Vue3-shallowRef&markRaw"
pubDate:  "2024-04-24"
description: "Vue3-shallowRef&markRaw"
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

## Vue3-shallowRef&markRaw

### shallowRef

shallowRef常用于div的Ref绑定，而不是用`ref()`。

因为shallowRef只处理第一层的相应式，对于绑定DOM而言，只有视图更新，才会触发相应式变更，而引用的DOM内部的发生的变化，并不会深度监听而影响性能，造成性能上的不必要的浪费。

只需要关心引用变化，而非引用本身内部的变化，只要符合这个条件的都可以用shallowRef，绑定Ref来说，刚好适合。

```typescript
// element-plus源码
const thumb = shallowRef<HTMLElement>()
const bar = shallowRef<HTMLElement>()
const editorRef = shallowRef<HTMLElement>()
```

### markRaw

让变量比如对象，变为**不可相应式的**。
可能会用上markRaw的场景
- 复杂的第三方类的实例
- Vue组件对象
- 当有不可变的数据源的时候，比如省市区列表等只可读的数据。跳过代理以提高性能。

```javascript
const foo = markRaw({
  nested: {}
})

const bar = reactive({
  // 尽管 `foo` 被标记为了原始对象，但 foo.nested 却没有
  nested: foo.nested
})

console.log(foo.nested === bar.nested) // false
```

因为Vue把bar.nested视做一个独立的对象，于未响应式的foo.nested属于不同的引用类型。bar.nested属于相应式的一部分。

通常markRaw用于引入组件和引入异步组件

```javascript
{
  component: markRaw(
    defineAsyncComponent({
      loader: () => import("@/components/editor/DefaultEditor.vue"),
      loadingComponent: VLoading,
      delay: 200,
    })
  ),
}
```

markRaw在tencent系和element-plus里通常都为引入的Icon的Components来包裹他们.

```javascript
config.push({
        type: 'button',
        className: 'rule',
        icon: markRaw(Memo),
        tooltip: showRule.value ? '隐藏标尺' : '显示标尺',
        handler: () => uiService?.set('showRule', !showRule.value),
      });
```

### toRaw

处理临时想获得相应式对象的原始值的方法，并不会改变原始响应式，而是只提供了一个访问原始值的方法。

```javascript
const foo = {}

const reactiveFoo = reactive(foo)

toRaw(reactiveFoo) === foo // true
```

比如想快速对比原始值
```javascript
watch(
  [() => props.config, () => props.initValues],
  ([config], [preConfig]) => {
    if (!isEqual(toRaw(config), toRaw(preConfig))) {
      initialized.value = false;
    }
    // ....其他代码
  },
  { immediate: true },
);
```

联动下toValue(),toValue如果是函数会执行一次该函数，如果是响应式就会返回该相应式的原始值。