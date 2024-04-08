---
layout: '../../layouts/MarkdownPost.astro'
title: "React中实现拖拽功能的自定义Hooks"
pubDate:  "2024-03-31"
description: "React中实现拖拽功能的自定义Hooks"
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

## React中实现拖拽功能的自定义Hooks

### 首先要拆分拖拽列表的设计

V = f(state, props)

- 页面
  - DraggableList
  - Draggable
    - Card
  - Bar

hooks封装
- useDraggable
    - 状态
      - dragOver
      - dragging
      - dragList
    - 事件
      - dragStart
      - dragEnd
      - dragOver
      - ...

### useDraggableHooks的实现

```typescript
import React, {useState, DragEvent} from "react";
import {CardTypeItem} from "./App8";

export enum ComponentType  {
    DRAGGABLE = 'DRAGGABLE',
    BAR = 'BAR'
}
interface NewBarItem {
    type: ComponentType,
    id: number,
    data?: CardTypeItem
}
interface DraggableItem {
    type: ComponentType,
    id: number,
    data: CardTypeItem
}
function draggable(item: CardTypeItem, id: number): DraggableItem {
    return {
        type: ComponentType.DRAGGABLE,
        id,
        data: item
    }
}
function insertBars(list: CardTypeItem[]): (DraggableItem | NewBarItem)[] {
    let i = 0;

    /*
     可以这样定义data，那么NewBarItem可以不用传data，而且生成了data为undefined
     这样确保了NewBarItem和DraggableItem结构上一致，
     即使拖拽的item没有data，也可以生成一个占位的undefined
     */

    // const NewBarItem = function (data?: CardTypeItem): NewBarItem {
    //     return {
    //         type: ComponentType.BAR,
    //         id: i++,
    //         data: data
    //     }
    // }

    const NewBarItem = function (): NewBarItem {
        return {
            type: ComponentType.BAR,
            id: i++
        }
    }

    return [NewBarItem()].concat(
        ...list.map(item => {
            return [draggable(item, i++),NewBarItem()]
        })
    )
}

function clacChanging<T>(list:T[] , drag:number, drop:number): T[] {
    list = list.slice()
    const dragItem = list[drag]
    // dir > 0从下往上 <0 从上往下
    const dir = drag > drop ? -2 : 2
    const end = dir > 0 ? drop - 1 : drop + 1

    let i: number = drag
    for (i; i !== end; i += dir) {
        list[i] = list[i + dir]
    }

    list[end] = dragItem

    return list
}

interface useDraggableProps {
    list: CardTypeItem[];
}
export interface DraggingEventHandlers {
    onDragStart: (e: DragEvent<HTMLDivElement>) => void;
    onDragEnd: (e: DragEvent<HTMLDivElement>) => void;
}

export interface DroppingEventHandlers {
    onDragOver: (e: DragEvent<HTMLDivElement>) => void;
    onDragLeave: (e: DragEvent<HTMLDivElement>) => void;
    onDrop: (e: DragEvent<HTMLDivElement>) => void;
}
interface DraggableProps {
    id: number;
    key: string;
    dragging: null | number;
    eventHandlers: DraggingEventHandlers;
}

interface DropperProps {
    id: number;
    dragging: null | number;
    dragOver: null | number;
    eventHandlers: DroppingEventHandlers;
}

export default function useDraggable(props : useDraggableProps) {
    const { list } = props;

    const [dragList, setDragList] = useState<(DraggableItem | NewBarItem)[]>(() => insertBars(list))

    const [dragOver, setDragOver] = useState<null | number>(null);
    const [dragging, setDragging] = useState<null | number>(null);

    return {
        dragList,
        // 放置函数
        createDropperProps: (id: number): DropperProps => {

            return {
                id,
                dragging,
                dragOver,
                eventHandlers: {
                    onDragOver: (e: DragEvent<HTMLDivElement>) => {
                        e.preventDefault()
                        setDragOver(id)
                    },
                    onDragLeave: (e:DragEvent<HTMLDivElement>) => {
                        e.preventDefault()
                        setDragOver(null)
                   },
                    onDrop: (e:DragEvent<HTMLDivElement>) => {
                        e.preventDefault()
                        setDragOver(null)
                        setDragList(list => {
                            return clacChanging(list, dragging!, id)
                        })
                    },
                }
            }
            
        },
        // 拖拽函数
        createDraggerProps: (id: number, key: string): DraggableProps => {
            return{
                id,
                key,
                dragging,
                eventHandlers: {
                    onDragStart: (e: DragEvent<HTMLDivElement>) => {
                        e.stopPropagation()
                        setDragging(id)
                    },
                    onDragEnd: (e: DragEvent<HTMLDivElement>) => {
                        setDragging(null)
                    },

                }
            }
        }
    }
}
```

页面里实现
```typescript
import {Component, ReactNode, useState} from "react";
import './App8.css'
import cc from './images/cc.jpg'
import useDraggable, {ComponentType, DraggingEventHandlers, DroppingEventHandlers} from "./useDraggable";
export interface CardTypeItem {
    src?: string;
    title?: string;
}

const LIST: CardTypeItem[]  = [
    {
        src: cc,
        title: 'cc1'
    },
    {
        src: cc,
        title: 'cc2'
    },
    {
        src: cc,
        title: 'cc3'
    },

]

function cls (def: string,...condition: [boolean,string][])  {
    const list:string[] = [def];
    condition.forEach(cond => {
        if (cond[0]) {
            list.push(cond[1]);
        }
    })
    return list.join(' ');
}
interface BarProps {
    id: number;
    dragging: number | null;
    dragOver: number | null;
    eventHandlers: DroppingEventHandlers; // 使用正确的事件处理类型
}

// Draggable 组件的类型也应该对应更新
interface DraggableProps {
    children: ReactNode;
    eventHandlers: DraggingEventHandlers; // 使用正确的事件处理类型
    dragging: number | null;
    id: number;
}
function Bar({id, dragging, dragOver, eventHandlers}: BarProps) {
    /**
     * // 如果拖拽之后，中间CardTypeItem走了之后，
     * 原本在这个CardTypeItem的上下存在的必然会各有1个Bar，
     * 那么就会导致有两个Bar，所以删除第二个Bar
     */
    if (dragging !== null && id === dragging  + 1) {
        return null
    }
    return (
        <div
            {...eventHandlers}
            className={cls('draggable--bar',[dragOver === id,'dragover'])}
        >
            <div
                className='inner'
                style={{
                    height: id === dragOver ? '100px' : '0px'
                }}
            >
                <h3>Bar:id:{id},dragging:{dragging},dragOver:{dragOver}</h3>

            </div>
        </div>
    )
}
function Card ({src, title}: CardTypeItem) {
    return (
        <div className='card'>
            <img src={src} alt=""/>
            <span>{title}</span>
    </div>)
}

// Draggable 组件的类型也应该对应更新
interface DraggableProps {
    children: ReactNode;
    eventHandlers: DraggingEventHandlers; // 使用正确的事件处理类型
    dragging: number | null;
    id: number;
}

function Draggable({children, eventHandlers, dragging, id}:DraggableProps) {
    return (
        <div
            {...eventHandlers}
            draggable={true}
            className={cls('draggable',[dragging === id,'dragging'])}
        >
            <h3>dragging:{dragging},id:{id}</h3>
            {children}

        </div>
    )
}

function App() {

    return (
       <div className='App'>
           <div>
               <DraggableList list={LIST}/>
           </div>
       </div>
    )
}
interface DraggableListProps {
    list: CardTypeItem[]
}
function DraggableList({list} :DraggableListProps) {
    const { dragList, createDraggerProps, createDropperProps } = useDraggable({ list });
    return (
        <>
            {dragList.map((item, i) => {
                if (item.type === ComponentType.BAR) {
                    return <Bar {...createDropperProps(i)} key={item.id}></Bar>;
                } else {
                    // 注意：这里将 item.data 作为属性传递给 Card，确保 item.data 满足 Card 组件的属性要求
                    // 如果 item.data 的属性可能为 undefined，应该提供默认值或者进行条件渲染
                    return (
                        <Draggable {...createDraggerProps(i, item.id.toString())} key={item.id}>
                            {/* 这里修改了，确保正确地传递属性给 Card 组件*/}
                            <Card {...item?.data}></Card>
                        </Draggable>
                    );
                }
            })}
        </>
    )
}
export default App
```

### 需要注意的点
第一次用ts来写demo
遇到几个自己不懂的点
1. DraggableItem 和 NewBarItem的类型区别
DraggableItem比NewBarItem多了一个确定性的data字段。考虑需要用一个接口还是两个接口来完善类型。
一个的话，那么在NewBarItem填入data字段的方法就可以在形参里面定义data?: CardTypeItem。

2. 不一定什么东西都需要上泛型
本来想给insertBars上泛型的，但是GPT给出的建议是，传入的东西其实都有确定性的。也就是传入的东西都是可控的，那么泛型就没有必要，定义一个接口类型就可以搞定了、

3. 对泛型T有更深的认知
就是对于一个你想去描述的泛型有点复杂，但是可以抽象出他的传递其实和实现并没有强联系。而是其实可以return回去。所以怎么来就可以怎么走。所以定义泛型T就可以知道不确定中的确定性。
回顾下clacChanging方法
```typescript
// 这个T就是对list的描述，本身这个list可以用DraggableItem 和 NewBarItem来描述
// 但是其实没有必要，因为list作为参数，只是对list进行了重排，并没有修改其数据结构，并且也返回了这个list
// 所以没必要对这个Item有跟深的交流，而是重心放在逻辑上的处理就是
function clacChanging<T>(list:T[] , drag:number, drop:number): T[] {
    list = list.slice()
    const dragItem = list[drag]
    // dir > 0从下往上 <0 从上往下
    const dir = drag > drop ? -2 : 2
    const end = dir > 0 ? drop - 1 : drop + 1

    let i: number = drag
    for (i; i !== end; i += dir) {
        list[i] = list[i + dir]
    }

    list[end] = dragItem

    return list
}
```

4. 对any[]的担忧
就是ts确实容易写成anyts。因为很多时候大多数人想的是能跑就行。ts相当于是对你的道德约束。确实也不能强制你去遵守道德规范，遵守实践与否，道不远人，自在人心。

### 感悟

很多事情其实不要想着完全学会了才开始去实践操作。而最好的就是，just do it。干就完了。要完成一个东西，先把这个东西完成到60%，然后再去完善。这项做反馈是最有效的，并且对于自己来说，付出的性价比更高。

**对于我们这些小卡拉咪来说，看源码是一种对实践的浪费**。因为源代码有大量的数据结构和算法，你没有先天圣体，很难get到这样那样，作者想表达的tips。因为我们这方面的功底欠缺或者功底不够。对于我们提升最大的，更希望把重心放在其背后的思想上。

精力是有限的，肯定想让自己的效益最大化，就好比OP需要刷一个角色的练度，0~80%的练度提升是最快的，80%~100%的练度需要大量刷圣遗物来堆起来。所以提升最快的就是60%理论。**听人劝，吃饱饭。饭要一口一口的吃，成长要一步一步来。**