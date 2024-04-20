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

如果子组件需要一个基于props的可修改的相应式引用，那么可以用computed的`get`和`set`来共同实现。


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

emit可以结合.sync从子组件修改父组件传入进来的props，来达成双向数据绑定

在vue2里有.sync，但是vue3里已经取消了.sync。



子组件通过`update:value`来直接把emit的值更新了，然后父组件通过`.sync修饰`符来完成数据双向绑定。
`.sync`会自动将子组件更新的值传递给父组件，并更新父组件相应prop的值。


```vue
<!--父组件-->
<script setup>
  import { ref } from 'vue'
  import Child from './child.vue'

  const count = ref(0)

</script>

<template>
  <Child v-model:count="count"></Child>
</template>

<!--子组件-->
<script setup>
  const props = defineProps({
    count:{
      type: Number,
      default: () => 0
    }
  })
  const emit = defineEmits(['update:count'])
  function changeCount () {
    const newCount = props.count + 1; // Increment the count
    emit('update:count', newCount);   // Emit the updated count
  }
</script>
<template>
  <div>
    {{ count }}
  </div>
  <button @click="changeCount">
    changeCount
  </button>
</template>
```

现在这个写法是不能省略v-model:count，因为默认有个props就modelValue，如果是用这个名称的话，在父组件就可以省略。

可以定义是默认的v-model。

```vue
<!--父组件-->
<script setup>
  import { ref } from 'vue'
  import Child from './Child.vue'
  const parentValue = ref('Initial Value');

</script>

<template>
  <div>
    <Child v-model="parentValue"></Child>

  </div>
</template>

<!--子组件-->
<template>
  <div>
    <p>Child Component: {{ modelValue }}</p>
    <button @click="updateValue">Update Parent Value</button>
  </div>
</template>

<script setup>
  import { defineProps, defineEmits } from 'vue';

  const props = defineProps({
    modelValue:{
      type: String,
      default: ''
    }
  });
  const emit = defineEmits(['update:modelValue']);

  const updateValue = () => {
    emit('update:modelValue', 'New Value');
  };
</script>
```

如果想绑定多个v-model，那么除了默认的modelValue之外，其他的正常写就行

```vue
<!--父组件-->
<script setup>
  import { ref } from 'vue'
  import Child from './Child.vue'
  const parentValue = ref('Initial Value');
  const count = ref(0)
</script>

<template>
  <div>
    <Child v-model="parentValue" v-model:count="count"></Child>

  </div>
</template>

<!--子组件-->
<template>
  <div>
    <p>Child Component: {{ modelValue }}</p>
    <button @click="updateValue">Update Parent Value</button>
    <p>count: {{ count}}</p>
    <button @click="updateCount">Update Count Value</button>

  </div>
</template>

<script setup>
  import { defineProps, defineEmits } from 'vue';

  const props = defineProps({
    modelValue:{
      type: String,
      default: ''
    },
    count:{
      type: Number,
      default: 0
    }
  });
  const emit = defineEmits(['update:modelValue','update:count']);

  const updateValue = () => {
    emit('update:modelValue', 'New Value');
  };
  const updateCount = () => {
    emit('update:count',props.count + 1)
  }
</script>
```
### defineModel()

`Vue3.4+`开始，推荐的实现方式是`defineModel()`。

```vue
<!--父组件-->
<script setup>
  import { ref } from 'vue'

  import Comp from './Comp.vue'
  const count = ref(null)


</script>

<template>
  <Comp v-model="count"></Comp>
  <div>father count :{{ count}}</div>
</template>
<!--子组件-->
<script setup>

  const value = defineModel({
    type: Number,
    default: 0
  })

  function handleValue () {
    value.value += 1
  }
</script>

<template>
  <div>
    <div>value:{{ value }}</div>
    <button @click="handleValue"> update </button>
  </div>
</template>

```

defineModel其实是一个语法糖
```vue
<script setup>
const props = defineProps(['modelValue'])
const emit = defineEmits(['update:modelValue'])
</script>

<template>
  <input
    :value="props.modelValue"
    @input="emit('update:modelValue', $event.target.value)"
  />
</template>
```

