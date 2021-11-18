---
title: vue组件使用
date: 2019-11-17 20:47:50
tags:
   - 前端
   - vue
categories: 
   - vue
---





# vue组件使用及通信

<!--more-->

## 1-组件概念和使用:

### 1,组件概念:

`组件是可复用的 Vue 实例, 封装标签, 样式和JS代码`

**组件化** ：封装的思想，把页面上 `可重用的部分` 封装为 `组件`，从而方便项目的 开发 和 维护

**一个页面， 可以拆分成一个个组件，一个组件就是一个整体, 每个组件可以有自己独立的 结构 样式 和 行为(html, css和js)**



组件有点: **各自独立**, **互不影响**

### 2,组件使用:

1.  创建.vue文件–标签–样式–JS进去
2. 导入组件(import xxx from 'path/to/components/xxx.vue')
3. 注册组件(全局/ 局部)
4. 使用组件(组件名用作标签)

语法:

```js
import Vue from 'vue'
import 组件对象 from 'vue文件路径'

Vue.component("组件名", 组件对象)

```



1. 局部注册:任意vue文件中中引入, 注册, 使用
2. 全局注册:局注册PannelG组件名后, 就可以当做标签在任意Vue文件中template里用



总结:

1. (创建)封装html+css+vue到独立的.vue文件中
2. (引入注册)组件文件 => 得到组件配置对象
3. (使用)当前页面当做标签使用



### 3,组件-scoped作用

作用:**解决多个组件样式名相同, 冲突问题**

```js
<style scoped> //style上加scoped, 组件内的样式只在当前vue组件生效
```

原理:

**在style上加入scoped属性, 就会在此组件的标签上加上一个随机生成的data-v开头的属性**

**而且必须是当前组件的元素, 才会有这个自定义属性, 才会被这个样式作用到**



## 2-组件通信:

因为每个组件的变量和值都是独立的

组件通信先暂时关注父传子, 子传父



### 1,父向子通信:

口诀(步骤):

1. 子组件内, props定义变量, 在子组件使用变量
2. 父组件内, 使用子组件, 属性方式给props变量传值



案例: 父向子配合循环:

数据:

```js
list: [
    { id: 1, proname: "超级好吃的棒棒糖", proprice: 18.8, info: '开业大酬宾, 全场8折' },
    { id: 2, proname: "超级好吃的大鸡腿", proprice: 34.2, info: '好吃不腻, 快来买啊' },
    { id: 3, proname: "超级无敌的冰激凌", proprice: 14.2, info: '炎热的夏天, 来个冰激凌了' },
],
```

```js
<template>
  <div>
    <MyProduct v-for="obj in list" :key="obj.id"
    :title="obj.proname"
    :price="obj.proprice"
    :intro="obj.info"
    ></MyProduct>
  </div>
</template>

<script>
// 目标: 循环使用组件-分别传入数据
// 1. 创建组件
// 2. 引入组件
import MyProduct from './components/MyProduct'
export default {
  data() {
    return {
      list: [
        {
          id: 1,
          proname: "超级好吃的棒棒糖",
          proprice: 18.8,
          info: "开业大酬宾, 全场8折",
        },
        {
          id: 2,
          proname: "超级好吃的大鸡腿",
          proprice: 34.2,
          info: "好吃不腻, 快来买啊",
        },
        {
          id: 3,
          proname: "超级无敌的冰激凌",
          proprice: 14.2,
          info: "炎热的夏天, 来个冰激凌了",
        },
      ],
    };
  },
  // 3. 注册组件
  components: {
    // MyProduct: MyProduct
    MyProduct
  }
};
</script>

<style>
</style>
```

### 2,组件通信_单项数据流:

在vue中需要遵循单向数据流原则

    1. 父组件的数据发生了改变，子组件会自动跟着变
    2. 子组件不能直接修改父组件传递过来的props  props是只读的

`父组件传给子组件的是一个对象，子组件修改对象的属性，是不会报错的，对象是引用类型, 互相更新`

`总结: props的值不能重新赋值, 对象引用关系属性值改变, 互相影响`

### 3,子向父通信:

`总结: 父自定义事件和方法, 等待子组件触发事件给方法传值`

语法:

* 父: @自定义事件名="父methods函数"
* 子: this.$emit("自定义事件名", 传值) - 执行父methods里函数代码

组件:

```js
<template>
  <div class="my-product">
    <h3>标题: {{ title }}</h3>
    <p>价格: {{ price }}元</p>
    <p>{{ intro }}</p>
    <button @click="subFn">宝刀-砍1元</button>
  </div>
</template>

<script>
import eventBus from '../EventBus'
export default {
  props: ['index', 'title', 'price', 'intro'],
  methods: {
    subFn(){
      this.$emit('subprice', this.index, 1) // 子向父
      eventBus.$emit("send", this.index, 1) // 跨组件
    }
  }
}
</script>

<style>
.my-product {
  width: 400px;
  padding: 20px;
  border: 2px solid #000;
  border-radius: 5px;
  margin: 10px;
}
</style>
```

App.vue

```js
,mmm<template>
  <div>
    <!-- 目标: 子传父 -->
    <!-- 1. 父组件, @自定义事件名="父methods函数" -->
    <MyProduct v-for="(obj, ind) in list" :key="obj.id"
    :title="obj.proname"
    :price="obj.proprice"
    :intro="obj.info"
    :index="ind"
    @subprice="fn"
    ></MyProduct>
  </div>
</template>

<script>

import MyProduct from './components/MyProduct_sub'
export default {
  data() {
    return {
      list: [
        {
          id: 1,
          proname: "超级好吃的棒棒糖",
          proprice: 18.8,
          info: "开业大酬宾, 全场8折",
        },
        {
          id: 2,
          proname: "超级好吃的大鸡腿",
          proprice: 34.2,
          info: "好吃不腻, 快来买啊",
        },
        {
          id: 3,
          proname: "超级无敌的冰激凌",
          proprice: 14.2,
          info: "炎热的夏天, 来个冰激凌了",
        },
      ],
    };
  },
  components: {
    MyProduct
  },
  methods: {
    fn(inde, price){
      // 逻辑代码
      this.list[inde].proprice > 1 && (this.list[inde].proprice = (this.list[inde].proprice - price).toFixed(2))
    }
  }
};
</script>

<style>
</style>
```

## 3-总结:

### Vue组件如何进行传值的

父向子 -> props定义变量 -> 父在使用组件用属性给props变量传值

子向父 -> $emit触发父的事件 -> 父在使用组件用@自定义事件名=父的方法 (子把值带出来)
