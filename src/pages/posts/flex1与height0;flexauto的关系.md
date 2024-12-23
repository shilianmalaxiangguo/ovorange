---
layout: '../../layouts/MarkdownPost.astro'
title: 'flex1与height0;flexauto的关系'
pubDate: 2024-12-23
description: 'flex1与height0;flexauto的关系'
author: '保安'
cover:
  url: '../img/cc.jpg'
  square: '../img/cc.jpg'
  twitter: '../img/cc.jpg'
  alt: 'cover'
tags: [ "CSS基础" ]
theme: 'dark'
featured: true
---

![封面|wide](/img/cc.jpg)[1]

## flex1与height0;flexauto的关系

### 需求
我需要一个父div，她有两个子div，一个高度是动态，另一个高度是自动占满剩下空间。

### 实现
[实现地址](https://my-vue3-demo.pages.dev/tab-dialog)

html部分

```html
<template>
  <div>
    <el-button type="primary" @click="dialogVisible = true">打开 Tab Dialog</el-button>

    <el-dialog
      v-model="dialogVisible"
      title="Tab Dialog"
      width="80%"
      style="min-height: 70vh;"
      :destroy-on-close="true"
      class="flex flex-col tab-dialog"
    >
      <el-tabs class="root" v-model="activeTab">
        <el-tab-pane label="数据列表" name="first" style="height: 100%;border: 1px solid red;">
          <div class="flex flex-col h-full">
            <!-- 搜索区域 -->
            <div class="mb-4 shrink-0">
              <el-form :inline="true">
                <el-form-item label="搜索">
                  <el-input v-model="searchText" placeholder="请输入关键词" />
                </el-form-item>
                <el-form-item>
                  <el-button type="primary" @click="handleSearch">搜索</el-button>
                </el-form-item>
              </el-form>
            </div>
            
            <!-- <div class="h-0 flex-auto overflow-y-auto"> -->
            <div class="flex-1 overflow-y-auto">
              <el-table
                :data="tableData"
                style="width: 100%; height: 100%"
              >
                <el-table-column prop="date" label="日期" />
                <el-table-column prop="name" label="姓名" />
                <el-table-column prop="address" label="地址" />
              </el-table>
            </div>
          </div>
        </el-tab-pane>
        
        <el-tab-pane label="其他内容" name="second">
          <div>这是第二个 tab 的内容</div>
        </el-tab-pane>
      </el-tabs>
    </el-dialog>
  </div>
</template>
```
css部分

```scss
.root {
    /**
    这里不能用 flex: 1 替换的原因是：
    .root 是 el-tabs 组件的根元素，它需要先将自身高度设置为 0（height: 0），然后通过 flex: auto 来适应内容
    这里的 flex: auto（即 flex: 1 1 auto）会考虑内容的实际大小
    如果改成 flex: 1（即 flex: 1 1 0%），由于 flex-basis: 0%，可能会导致内容被压缩，tabs 的布局会出问题 
    */
  flex: auto;
  height: 0;
  display: flex;
  flex-direction: column-reverse;
  border: 1px solid rgb(30, 255, 0);
}

.tab-dialog {
  .el-dialog__body {
    display: flex !important;
    flex-direction: column !important;
    /**
    这里可以用 flex: 1 替换的原因是：
    .el-dialog__body 是对话框的内容区域，它的作用是填充剩余空间
    在这个场景下，我们不需要考虑内容的初始大小，只需要它填满剩余空间
    因此 flex: 1 的行为（flex-basis: 0%）正好符合需求
    */
    // flex: auto;
    // height: 0;
    flex: 1;
  }
}
/**
当元素需要考虑其内部组件的布局特性时（如 tabs 组件），使用 flex: auto + height: 0 更合适
当元素只需要简单地填充剩余空间时，使用 flex: 1 更合适
这就是为什么在 .root 中使用 flex: auto + height: 0，而在 .tab-dialog 中使用 flex: 1 的原因  
*/
```

## 关键点

### 关键点1
`flex: auto + height: 0` 其实另外的写法也可以是 `flex: 1 + flex-basis: 0`

然后某种情况也可以是`flex: 1`

当在某个元素之内当子元素，比如el-tabs里，那么这个元素的`flex: 1` 会失效，需要使用`flex: auto + height: 0` 来实现

自己本身就是父级，那么`flex: 1` 会生效，`flex: auto + height: 0` 也会生效

### 关键点2
优先使用`height: 0;flex: auto; `而不是`flex: 1;basis: 0;`

因为语句更好理解，`flex: 1;basis: 0;` 语句更难理解