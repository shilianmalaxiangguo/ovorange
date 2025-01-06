---
layout: '../../layouts/MarkdownPost.astro'
title: 'extends和implements'
pubDate: 2025-01-06
description: 'extends和implements'
author: '保安'
cover:
  url: '../img/cc.jpg'
  square: '../img/cc.jpg'
  twitter: '../img/cc.jpg'
  alt: 'cover'
tags: [ "TS基础" ]
theme: 'dark'
featured: true
---

![封面|wide](/img/cc.jpg)[1]

## extends（继承）

```javascript
// 基础服装店
class ClothingShop {
  // 基本功能
  sellClothes() { }
  manageInventory() { }
  processPayment() { }
}

// 优衣库继承服装店
class Uniqlo extends ClothingShop {
  // 自动获得了服装店的所有基本功能
  // 还可以添加自己特有的功能
  sellUniqloExclusives() { }
}
```

就像优衣库是一种特殊的服装店：
优衣库自动拥有了服装店的所有基本功能（卖衣服、管理库存、处理付款）
还可以添加自己特有的功能（卖优衣库独家商品）
一个类只能继承一个父类（一个店铺不能同时是优衣库又是H&M）


## implements（实现接口）
想象商场制定的标准规范，每个店铺都必须遵守：

```javascript
// 商场规定的标准
interface ShoppingMallStandards {
  openOnTime(): void;      // 准时开门
  keepClean(): void;       // 保持清洁
  handleComplaints(): void; // 处理投诉
}

// 优衣库实现商场标准
class Uniqlo implements ShoppingMallStandards {
  openOnTime() {
    // 必须实现准时开门
  }
  keepClean() {
    // 必须实现保持清洁
  }
  handleComplaints() {
    // 必须实现投诉处理
  }
}

// H&M也实现商场标准
class HM implements ShoppingMallStandards {
  openOnTime() { }
  keepClean() { }
  handleComplaints() { }
}
```
就像商场里的每个店铺都必须遵守商场的规定：
商场规定了一系列标准（接口）
每个店铺都必须实现这些标准
一个店铺可以同时实现多个标准（比如既遵守商场标准，又遵守消防标准）

## 实际项目中

```javascript
// PrismaService 既继承了 PrismaClient，又实现了两个生命周期接口
export class PrismaService extends PrismaClient implements OnModuleInit, OnModuleDestroy {
  // ...
}
```
这就像：
一个特殊的数据库管理店（PrismaService）继承了数据库操作公司（PrismaClient）的所有功能
同时还要遵守商场（NestJS）规定的两个标准（OnModuleInit 和 OnModuleDestroy）
开业时必须做的事（连接数据库）
关门时必须做的事（断开数据库连接）
简单来说：
extends：继承（是什么） - "优衣库是一家服装店"
implements：实现（必须做什么） - "优衣库必须遵守商场规定"