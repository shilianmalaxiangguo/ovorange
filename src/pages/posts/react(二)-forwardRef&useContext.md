---
layout: '../../layouts/MarkdownPost.astro'
title: 'react(二)-forwardRef&useContext'
pubDate: 2024-03-18
description: 'react(二)-forwardRef&useContext'
author: '保安'
cover:
  url: '../img/cc.jpg'
  square: '../img/cc.jpg'
  twitter: '../img/cc.jpg'
  alt: 'cover'
tags: [ "React" ]
theme: 'dark'
featured: true
---

![封面|wide](/img/cc.jpg)[1]

## forwardRef

在 React 中，当你封装一个组件并且使用 forwardRef 来转发引用到内部的 DOM 节点时，你通常会定义两个组件：

- 内部组件 (Input): 这个组件实际上处理渲染逻辑和转发 ref。
- 外部组件 (WrapperInput): 这个组件是你暴露给其他开发者使用的，并且它通过 forwardRef 来包裹内部组件。
在这种情况下，InputProps 应该定义所有你希望从外部传递给 Input 组件的属性。因此，InputProps 应该在外部组件 WrapperInput 中使用，因为这是其他开发者将要传递属性的地方。

你的 Input 组件可以接收 InputProps 作为其属性，因为它需要知道如何处理这些传入的属性（包括 xPlacement 和任何其他可能的自定义属性）。同时，WrapperInput 也需要使用 InputProps 来确保类型正确，并且 TypeScript 能够正确地推断传递给 WrapperInput 的属性。

这里是如何在代码中实现这一点的：
```typescript jsx
import React, {
  useRef,
  useImperativeHandle,
  forwardRef,
  useLayoutEffect,
  ForwardRefRenderFunction,
  HTMLProps,
} from 'react';

interface RefProps {
  focus: () => void;
  blur: () => void;
}

interface InputProps extends HTMLProps<HTMLInputElement> {
  xPlacement: 'top' | 'bottom';
}

const Input: ForwardRefRenderFunction<RefProps, InputProps> = (props, ref) => {
  const inputRef = useRef<HTMLInputElement>(null);
  useImperativeHandle(ref, () => ({
    focus() {
      inputRef.current?.focus();
    },
    blur() {
      inputRef.current?.blur();
    },
  }));

  // 这里你可以根据 xPlacement 来决定样式或者类名
  const className = `input-${props.xPlacement}`;

  return <div className={className}>
    <input type="text" ref={inputRef} {...props} />
  </div>;
};

// 这里使用 InputProps 作为 WrapperInput 的属性类型
const WrapperInput = forwardRef<RefProps, InputProps>(Input);

function App() {
  const wrapperInputRef = useRef<RefProps>(null);
  useLayoutEffect(() => {
    wrapperInputRef.current?.focus();
  }, []);

  return (
    <div>
      <WrapperInput ref={wrapperInputRef} xPlacement="top" />
    </div>
  );
}

export default App;
```

### 需要注意的点

有两个组件，一个是 Input，一个是 WrapperInput。Input 是内部组件，WrapperInput 是外部组件。
useImperativeHandle 可以让你在使用 ref 时自定义暴露给父组件的实例值。
在App里面定义了个useRef，wrapperInputRef -> wrapperInputRef组件 -> Input组件 -> useImperativeHandle里的传入的useRef实例 -> 绑定ref

## useContext

```typescript jsx
import {createContext, useContext} from "react";

const counterContext = createContext(99)

function App() {
    return <div>
        <counterContext.Provider value={299}>
            <Middle></Middle>
        </counterContext.Provider>
    </div>
}

function Middle () {
    return <div>
        <Child></Child>
    </div>
}
function Child () {
    const childCounter = useContext(counterContext)
    return <p>当前context值:{childCounter}</p>
}
export default App

```

用 createContext 创建 context 对象，用 Provider 修改其中的值，
function 组件使用 useContext 的 hook 来取值，class 组件使用 Consumer 来取值

-