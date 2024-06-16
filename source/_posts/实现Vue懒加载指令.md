---
title: 实现Vue懒加载指令v-lazy
date: 2024-06-16 20:55:39
tags: [前端, Vue, 手写, 工具代码]
categories: [前端, Vue]
excerpt: 懒加载是一种网页优化技术，在页面滚动时，只加载可视区域内的图片
index_img: https://img.zphl.top/blog/articleImg/1.jpg
banner_img: https://img.zphl.top/blog/bg/bg.jpg
---

前面我们手写了图片的懒加载（可以查看之前文章），这次我结合 Vue 的插件 API 来更好的实现懒加载的指令。

有关插件教程可以查看: [Vue 插件](https://cn.vuejs.org/guide/reusability/plugins.html)，

有关自定义指令相关教程可以查看：[Vue 自定义指令](https://cn.vuejs.org/guide/reusability/custom-directives.html)，这里不再做赘述。

### 1.收集懒加载的元素

当元素绑定 `v-lazy` 指定时，我们将该元素收集到 `markElement`数组中：

```js
export const lazy = {
    // v-lazy 绑定的元素
    markElement: [],
    // 收集元素
    mark(el) {
        this.markElement.push(el);
    },

    install(app) {
        const _this = this;
        app.directive("lazy", {
            beforeMount(el, binding) {
                _this.mark(el);
            },
        });
    },
};
```

### 2.判断元素是否在可视区域

我们已经收集到了所有元素，那么我们就可以判断这些元素是否在可视区域了。判断逻辑还是和之前一样：监听页面滚动，当元素距离页面顶部的高度 `top` 小于 可视窗口的高度 `viewHeight` 时，元素即在可视范围内，加载图片：

```js
// 防抖
  debounce(fn, delay) {
    let timer = null;
    return function () {
      if (timer) {
        clearTimeout(timer);
      }
      timer = setTimeout(() => fn.apply(this, arguments), delay);
    };
  }


 // 判断元素是否在可视范围内，范围内的元素加载图片
  init() {
    this.markElement.forEach(el => {
      const src = el.getAttribute("data-src");
      if (!src) return;
      const viewHeight = window.innerHeight;
      const top = el.getBoundingClientRect().top;
      const bottom = el.getBoundingClientRect().bottom;
      if (top < viewHeight && bottom > 0) {
        el.setAttribute("src", src);
      }
    });
  }


  install(app) {
    app.directive("lazy", {...})
    // 监听页面滚动
    window.addEventListener("scroll", this.debounce(() => {
        this.init();
      }, 200));
  }
```

**在 `install` 中注册滚动事件，并去判断所有收集的元素是否在可视范围内，而不是在指定的生命周期中注册 `scroll` 事件，避免重复注册，造成性能浪费。**

别忘了还可以使用构造函 `IntersectionObserver` 实现，这里我们判断浏览器支持就是用该 api，不支持则使用“滚动判断法”:

```js
 observer() {
    const observer = new IntersectionObserver((entries) => {
      entries.forEach(entry => {
        // 无data src 属性，不处理
        if (entry.isIntersecting && entry.target.getAttribute("data-src")) {
          entry.target.setAttribute("src", entry.target.getAttribute("data-src"));
          observer.unobserve(entry.target);
        }
      });
    }, {
      root: null,
      threshold: 1
    });
    return observer;
  }


  // 支持 IntersectionObserver 则使用 IntersectionObserver 加载图片
        if (IntersectionObserver) {
          const observer = _this.observer();
          _this.markElement.forEach(el => {
            observer.observe(el);
          });
        }

// 不支持 IntersectionObserver则使用 scroll 事件加载图片
    if (!IntersectionObserver) {
      window.addEventListener("scroll", this.debounce(() => {
        this.init();
      }, 200));
    }

```

### 3.初始化

在初次进入页面时我们需要初始化一次判断元素是否在可视范围内，使用 `IntersectionObserver`时，也需要初始化监听所有元素，这里可以在 `mounted` 声明周期中实现:

```js
install(app) {
    const _this = this;
    app.directive("lazy", {
      beforeMount(el, binding) {
        _this.mark(el);
      },
      mounted() {

        // 支持 IntersectionObserver 则使用 IntersectionObserver 加载图片
        if (IntersectionObserver) {
          const observer = _this.observer();
          _this.markElement.forEach(el => {
            observer.observe(el);
          });
        } else {
          // 不支持 使用手写代码
          _this.init();
        }
      }
    })
}
```

下面时完整代码：

```js
export const lazy = {
    // v-lazy 绑定的元素
    markElement: [],
    // 收集元素
    mark(el) {
        this.markElement.push(el);
    },
    // 判断元素是否在可视范围内，范围内的元素加载图片
    init() {
        this.markElement.forEach((el) => {
            const src = el.getAttribute("data-src");
            if (!src) return;
            const viewHeight = window.innerHeight;
            const top = el.getBoundingClientRect().top;
            const bottom = el.getBoundingClientRect().bottom;
            if (top < viewHeight && bottom > 0) {
                el.setAttribute("src", src);
            }
        });
    },
    // 防抖
    debounce(fn, delay) {
        let timer = null;
        return function () {
            if (timer) {
                clearTimeout(timer);
            }
            timer = setTimeout(() => fn.apply(this, arguments), delay);
        };
    },
    observer() {
        const observer = new IntersectionObserver(
            (entries) => {
                entries.forEach((entry) => {
                    if (entry.isIntersecting && entry.target.getAttribute("data-src")) {
                        entry.target.setAttribute("src", entry.target.getAttribute("data-src"));
                        observer.unobserve(entry.target);
                    }
                });
            },
            {
                root: null,
                threshold: 1,
            }
        );
        return observer;
    },
    install(app) {
        const _this = this;
        app.directive("lazy", {
            beforeMount(el, binding) {
                _this.mark(el);
            },
            mounted() {
                // 支持 IntersectionObserver 则使用 IntersectionObserver 加载图片
                if (IntersectionObserver) {
                    const observer = _this.observer();
                    _this.markElement.forEach((el) => {
                        observer.observe(el);
                    });
                } else {
                    // 不支持 使用手写代码
                    _this.init();
                }
            },
        });
        // 不支持 IntersectionObserver则使用 scroll 事件加载图片
        if (!IntersectionObserver) {
            window.addEventListener(
                "scroll",
                this.debounce(() => {
                    this.init();
                }, 200)
            );
        }
    },
};
```

实现效果：

<iframe src="https://codesandbox.io/p/devbox/tu-pian-lan-jia-zai-dmhngn?file=%2Fsrc%2Fplugins%2FvuePlugins.js&embed=1"
     style="width:100%; height: 500px; border:0; border-radius: 4px; overflow:hidden;"
     title="图片懒加载"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

### 参考:

[1^]: https://www.zphl.top/2024/06/16/%E5%9B%BE%E7%89%87%E6%87%92%E5%8A%A0%E8%BD%BD/
[2^]: https://cn.vuejs.org/guide/reusability/custom-directives.html
[3^]: https://cn.vuejs.org/guide/reusability/plugins.html
