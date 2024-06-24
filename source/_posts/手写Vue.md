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

#### 2.1 初始化数据

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

#### 2.2 响应式实现

接下来是 Vue 的核心，响应式的实现

##### 1. 数据对象劫持

遍历 data 数据，对于对象，我们遍历对象属性，劫持每个属性的 **访问器描述符**

```js
class Observer {
    constructor(data) {
        // 遍历data的属性
        this.walk(data);
    }

    // 对象劫持
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

到这步，实际上我们劫持两遍 data 中的数据，一个是 \_data 中的属性，一个是 我们将 \_data 属性值代理到 vm 上时，又劫持了一遍，这里会造成性能的浪费，也就未 vue3 的优化埋下了伏笔

##### 2.数组劫持

判断值是数组，则进行数组的劫持，我们重写数组的 push、pop、shift、unshift、splice、sort、reverse 方法，因为这些方法会改变原数组值，也就涉及到视图的更新。

我们在重写的方法中执行原有的方法，这样原有的方法功能不会丢失，我们又可以在新的方法中做些事情例如视图的更新:

```js
class Observer {
    constructor(data) {
        if (Array.isArray(data)) {
            // 重点：覆盖原型链上的方法，使用重写后的方法： 重写 array 方法，正如Vue改变数据的要求一样，不能直接修改，而是使用 push 等方法更改
            data.__proto__ = newArrayProto;

            this.arrayWalk(data);
        } else {
            // 遍历data的对象属性
            this.walk(data);
        }
    }
    // 数组数据遍历劫持
    arrayWalk(array) {
        array.forEach((item) => {
            observer(item);
        });
    }
}
```

数组方法重写：

```js
// 原数组方法
const oldArrayMethods = Array.prototype;

// 准备重写的数组方法
const newArrayMethods = Object.create(oldArrayMethods);

// 需要重写（劫持）的函数
const menthods = ["push", "pop", "shift", "unshift", "splice"];

// 重写 array 的部分默认方法
menthods.forEach((method) => {
    newArrayMethods[method] = function (...args) {
        // do something...
        console.log("捕获到 数组数据更改");

        // 执行原数组方法
        const length = oldArrayMethods[method].apply(this, args);

        // 数组新增加的值
        let insertValue;
        // 对于数组新增值的方法，我们还需要对新增的值进行监听，判断新增的值是否仍为对象或者数组
        switch (method) {
            case "push":
                insertValue = args;
                break;
            case "unshift":
                insertValue = args;
                break;
            case "splice":
                insertValue = args.slice(2);
                break;
            default:
                break;
        }

        // 对数组新增的值进行监听判断是否仍未对象或数组以及是否需要继续劫持
        this.__ob__.arrayWalk(insertValue);

        return length;
    };
});

export default newArrayMethods;
```

上面的代码我们重写了数组相应的方法，对于数组新增的值(数组)，我们使用原来实例的方法`arrayWalk`重新判断是否需要继续劫持，那么我们如何再这里访问到 Observer 的实例的呢？

这里注意 this 的值指向的是当前数组，我们可以为在数组中添加一个**ob** 属性，将值赋值为 Observer 实例， 这样就可以使用实例中的方法：

```js
class Observer {
    constructor(data) {
        // 为 data 新增 _ob_ 属性，用来保存当前实例，一可以在遍历数组的时候访问当前实例，二可以为data添加标记，有_ob_属性则数据劫持过了不必再次劫持
        // enumerable: false 不可枚举，避免死循环
        Object.defineProperty(data, "__ob__", {
            value: this,
            enumerable: false,
        });
       ......
    }
}
```

这里对象或者数组都会创建一个 ob 属性，当有这个属性也说明里面的属性都被劫持过了，也就不用再次劫持：

```js
export default function observer(data) {
    if (typeof data != "object") return;
    if (data.__ob__) return;
    const oberver = new Observer(data);
    return oberver;
}
```

ob 属性枚举设置为 false 是为了避免死循环，如果为 true,当我们去遍历一个对象的时候，也会去遍历 Ob 属性， ob 属性里有原型，原型又有它自己的原型，就会陷入无限循环

下面是完整代码:

```js
// observer.js
import newArrayProto from "./array";

class Observer {
    constructor(data) {
        // 为 data 新增 _ob_ 属性，用来保存当前实例，一可以在遍历数组的时候访问当前实例，二可以为data添加标记，有_ob_属性则数据劫持过了不必再次劫持
        // enumerable: false 不可枚举，避免死循环
        Object.defineProperty(data, "__ob__", {
            value: this,
            enumerable: false,
        });
        if (Array.isArray(data)) {
            // 覆盖原型链上的方法，使用重写后的方法： 重写 array 方法，正如Vue改变数据的要求一样，不能直接修改，而是使用 push 等方法更改
            data.__proto__ = newArrayProto;

            this.arrayWalk(data);
        } else {
            // 遍历data的属性
            this.walk(data);
        }
    }

    walk(data) {
        Object.keys(data).forEach((key) => {
            // 劫持数据
            defineReactive(data, key, data[key]);
        });
    }
    // 数组数据遍历劫持
    arrayWalk(array) {
        array.forEach((item) => {
            observer(item);
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
    if (typeof data != "object") return;
    if (data.__ob__) return;
    const oberver = new Observer(data);
    return oberver;
}

// array.js
const oldArrayMethods = Array.prototype;

const newArrayMethods = Object.create(oldArrayMethods);

const menthods = ["push", "pop", "shift", "unshift", "splice"];

// 重写 array 的部分默认方法
menthods.forEach((method) => {
    newArrayMethods[method] = function (...args) {
        // do something...
        console.log("捕获到 数组数据更该", this);

        // 执行原数组方法
        const length = oldArrayMethods[method].apply(this, args);
        let insertValue;
        // 对于数组新增值的方法，我们还需要对新增的值进行监听，判断新增的值是否仍为对象或者数组
        switch (method) {
            case "push":
                insertValue = args;
                break;
            case "unshift":
                insertValue = args;
                break;
            case "splice":
                insertValue = args.slice(2);
                break;
            default:
                break;
        }

        // 对数组新增的值进行监听
        this.__ob__.arrayWalk(insertValue);

        return length;
    };
});

export default newArrayMethods;

```

至此，data 中的所有属性改变时我们就都可以监听到了
