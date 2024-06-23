---
title: 手写Vue2
date: 2024-06-23 21:27:10
tags:
    - 前端
    - vue
    - 手写
categories:
    - 前端
index_img: https://img.zphl.top/blog/articleImg/VUE.JS.jpeg
excerpt: 一把键盘，一个夜晚，一个奇迹
link: /posts/vue2.html
---

### 1. VueJs 原理

Vue 是数据驱动的，数据变化，视图自动更新 也就是所谓的 "响应式"，它承载的就是数据与视图的处理，让开发者更加关注逻辑的处理；

Vue 是一个 MVVM 框架，它包含三个部分：Model、View、ViewModel：

-   Model：数据层，负责处理数据，包括数据的增删改查；
-   View：视图层，负责数据的展示；
-   ViewModel：负责数据的处理，包括数据的绑定、监听、过滤等；

Vue 的响应式遵循发布订阅模式

### 2.实现响应式原理

Vue 接受一个对象 options，当我们去 `new Vue` 时，Vue 会调用 init 方法初始化初始数据

```js
const vm = new Vue({
    data: {
        name: "张三",
        age: 12,
        person: {
            names: "里斯",
        },
    },
});
```

初始化 data 数据，data 可以是函数，也可以是对象，但最终返回的是一个对象

```js
// vue.js
import initMixin from "./init";
function Vue(options) {
    // 初始化 Vue 内部的数据
    this._init(options);
}

// 拓展函数
initMixin(Vue);
export default Vue;
```

initMixin 用来给 vue 添加各种内置方法属性，\_init 方法就是在其中添加的：

```js
// init.js
import initState from "./state";

export default function initMixin(Vue) {
    Vue.prototype._init = function (options) {
        // 我们将 options 添加到 this 中
        this.$options = options;
        const vm = this;
        // 初始化state
        initState(vm);
    };
}
```

initState 用来初始化 data 数据

```js
//state.js
import { def } from "./def";
import observer from "./oberver";

export default function initState(vm) {
    let data = vm.$options.data;
    // 判断 data 是否是函数,如果是函数则执行,是对象则直接返回，最终的得到的是对象
    data = typeof data === "function" ? data.call(vm) : data;

    // 将data 挂载到 vue 示例 上
    vm._data = data;

    //代理 data, 我们常常在 vue 中访问data中的属性可以直接使用 this.xxx, 而不是 this.data.xxx
    for (let key in data) {
        // 还是利用 defineProperty, 将 data 的属性代理到 this 上
        def(vm, key, data[key]);
    }
    // 劫持 data
    observer(vm._data);
}
```

def 函数是将 data 中的属性遍历一个个都挂载到 vm 上

```js
// def.js
// 将 _data 代理到 this 上，并重新劫持 get 与 set
export function def(target, key, value) {
    Object.defineProperty(target, key, {
        get() {
            console.log("设置值");
            return value;
        },
        set(newValue) {
            console.log("获取值");
            if (newValue === value) return;
            value = newValue;
        },
    });
}
```

现在我们开始实现 observer 函数，也就是 vue 响应式的核心，通过劫持 get 与 set 函数，当设置值的时候 set 函数触发，我们就可以监听到数据变化，从而启用更新视图，当然这里我们先实现数据的劫持

```js
class Observer {
    constructor(data) {
        // 遍历data的属性
        this.walk(data);
    }

    walk(data) {
        Object.keys(data).forEach((key) => {
            // 劫持数据
            defineReactive(data, key, data[key]);
        });
    }
}

function defineReactive(data, key, value) {
    // 当传递两个参数时，说明没有传递值，我们手动获取
    if (arguments.length === 2) {
        value = data[key];
    }
    // 如果属性还是一个对象，我们则使用递归继续劫持该对象的属性
    if (typeof value === "object") {
        observer(value);
    }
    // 劫持数据
    Object.defineProperty(data, key, {
        get() {
            console.log("获取值");
            return value;
        },
        set(newV) {
            console.log("设置值");
            // 新旧值一样，则不作处理
            if (value === newV) return;
            value = newV;
        },
    });
}
export default function observer(data) {
    const oberver = new Observer(data);

    return oberver;
}
```

到这步，我们发现实际上我们劫持两遍 data 中的数据，一个是 \_data 中的属性，一个是 vm 也就是我们将 \_data 属性值代理到 vm 上时，我们又劫持了一遍
，这里会造成性能的浪费，也就未 vue3 的优化埋下了伏笔
