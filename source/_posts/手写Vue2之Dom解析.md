---
title: 手写Vue2之Dom解析
date: 2024-06-26 21:40:42
tags:
    - 前端
    - Vue
    - 手写
categories:
    - [前端, Vue]
index_img: https://img.zphl.top/blog/articleImg/VUE.JS.jpeg
excerpt: 一把键盘，一个夜晚，一个奇迹
link: /posts/vue2Complie.html
---

### 1. Vue2 Dom 解析流程概览

要想将数据渲染高页面上，首先需要将元素解析为抽象将语法树(AST),将语法树中的 data 属性替换为 data 中的值，再将抽象语法树渲染成真实的 DOM。

**解析流程**

-   查找选项中是否有 render 函数，如果有则使用 render 函数创建 VNode，没有再去查找 template 选项，如果有则使用 template 创建 VNode，最后去查找 el 选项，如果有则根据 el 选项创建 VNode。优先级：render > template > el
-   将元素解析为抽象语法树：
    Vue 获取获取页面上的元素，使用正则去匹配页面中的所有标签，属性，文本，将他们一一对应到 AST 中的节点上

下面我们开始逐行实现解析流程。

### 2.获取页面元素

我们创建 `$mount` 函数，在 mount 函数中将元素解析为抽象语法树，去`options`中查找相应的选项，**优先级为：render > template > el**:

```js
// init.js
// vue 挂载流程： 优先看是否有 rende 函数，有则直接编译，
//没有查看是否有 template 属性， 有则将 template 编译到 render函数，
//没有template，则看是否有 el 属性，有则将 el 里面的内容编译到 template 再编译到 render 函数
Vue.prototype.$mount = function (elementName) {
    const vm = this;
    // 获取到 el 元素
    const el = document.querySelector(elementName);

    let ops = vm.$options;
    let template;
    // 没有render
    if (!ops.render) {
        if (ops.template) {
            template = ops.template;
        } else if (el) {
            template = el.outerHTML;
        } else {
            console.log("没有 render 函数，也没有 template 属性");
        }
        if (template) {
            // 编译元素
            ops.render = compile(template);
        }
    }
    // 执行 render
};
```

### 3.元素编译

这步主要是将页面元素编译为抽象语法中，我们创建函数 complie 来实现这个功能。

#### 3.1 解析开始标签

-   因为标签总是成对出现的，当结束标签出现，说明这个标签的内容已经结束了，我们可以循环匹配字符，匹配成功就将这个字符删除，直到所有字符匹配完成，首先我们匹配开始标签字符，也就是匹配`<`这歌字符， 当这个字符在开头那么这歌就是一个元素的开始标签`<ele`，或者结束标签`</ele`,下面是完整代码：

```js
function complie(html) {
    if (html) {
        // 解析 html
        parseHtml(html);
    }
}

//则删除匹配完成的标签字符串，避免后面使用正则时重复匹配
function subHtml(startIndex) {
    return html.substring(startIndex);
}

function parseHtml(html) {
    //则删除匹配完成的标签字符串，避免后面使用正则时重复匹配
    function subHtml(startIndex) {
        return html.substring(startIndex);
    }
    // 匹配标签名
    const tagName = `[a-zA-Z_][\\-\\.0-9_a-zA-Z]*`;
    // 匹配带命名空间的标签
    const qnameCapture = `((?:${tagName}\\:)?${tagName})`;
    // 匹配开始标签 例如 <div> 会匹配 <div
    const stratTag = new RegExp(`^<${qnameCapture}`);
    const attrReg = /^\s*([^\s"'<>\/=]+)(?:\s*(=)\s*(?:"([^"]*)"+|'([^']*)'+|([^\s"'=<>`]+)))?/;

    // 解析开始标签
    function parseStartTag() {
        // 匹配开始标签
        const start = html.match(stratTag);
        if (start) {
            // 匹配的结果
            const match = {
                tagName: start[1],
                attrs: [],
            };
            // 删除匹配完成的字符串
            const startIndex = start[0].length;
            html = subHtml(startIndex);

            //匹配属性
            let attr, end;
            // 匹配到结束标签，且没有匹配到属性时，退出循环
            while (!(end = html.match(stratTagClose)) && (attr = html.match(attrReg))) {
                match.attrs.push({
                    name: attr[1],
                    value: attr[3],
                });
                html = subHtml(attr[0].length);
            }
            // 匹配开始标签的结束标签 >
            if (end) {
                html = subHtml(end[0].length);
            }

            return match;
        }
    }

    while (html) {
        // 判断是不是标签, 标签是以 < 开始的，例如：<div></div>，<br />
        let isStartTag = html.indexOf("<") === 0;

        // 开始/结束标签的处理
        if (isStartTag) {
            // 开始标签的处理 <div id="..." ...>
            const startMatch = parseStartTag(html);
        }
    }
}
```

`parseStartTag`函数获取匹配开始标签内的内容：

1. stratTag 正则匹配 `<element-name` 字符, 例如 `<div>`匹配的结果为:`['<div', 'div']`, 那么他的标签名为数组中的第二个元素，即`div`，我们创建 match 对象，并将 tagName 属性设置为`div`, 并将 attrs 属性设置为空数组：

```js
// 匹配开始标签
const start = html.match(stratTag);
if (start) {
    // 匹配的结果
    const match = {
        tagName: start[1],
        attrs: [],
    };
}
```

2. 我们需要将匹配过的字符删除，匹配结果：`['<div', 'div']`中的第一个元素就是我们需要删除的元素。创建一个函数`subHtml`，使用`substring`方法来删除匹配过的字符：

```js
//则删除匹配完成的标签字符串，避免后面使用正则时重复匹配
function subHtml(startIndex) {
    return html.substring(startIndex);
}

const start = html.match(stratTag);
// 删除匹配完成的字符串
const startIndex = start[0].length;
html = subHtml(startIndex);
```

3. 匹配属性，使用正则 `attrTag` 来匹配标签上的属性 例如：`<div id="..." class="...">`, 匹配的结果为：`['id="..."', 'id', '"...']`, 第一个元素为匹配的字符串，第二个元素为属性名，第三个元素为属性值；因为标签上可能存在多个属性，所以我们需要循环匹配属性，直到匹配到标签的结束符`>`则退出循环，我们将匹配到的属性添加到 `attrs`数组中：

```js
//则删除匹配完成的标签字符串，避免后面使用正则时重复匹配
function subHtml(startIndex) {
    return html.substring(startIndex);
}
//匹配属性
let attr, end;
// 匹配到结束标签，且没有匹配到属性时，退出循环
while (!(end = html.match(stratTagClose)) && (attr = html.match(attrReg))) {
    match.attrs.push({
        name: attr[1],
        value: attr[3],
    });
    html = subHtml(attr[0].length);
}
```

4. 匹配开始标签的结束标签 `>`，则开始标签匹配完成，将匹配到的字符串 `>`删除即可：

```js
// 匹配开始标签的结束标签 >
if (end) {
    html = subHtml(end[0].length);
}
```

5. 匹配开始标签完成，返回匹配对象 match

```js
return match;
```

#### 3.2 匹配结束标签

如果接下来在我们匹配完一个元素的开始标签后，直接匹配到该元素的结束标签，则说明该元素没有内容：文本、子元素, 而且结束标签也不会有属性，而该元素的标签名我们在匹配开始标签时就已经知道了，所以在这里我们就直接删除该元素的结束标签，以便可以开始下一个元素标签的匹配：

```js
function parseHtml(html) {
    while (html) {
        // 判断是不是标签, 标签是以 < 开始的，例如：<div></div>，<br />
        let isStartTag = html.indexOf("<") === 0;

        // 开始/结束标签的处理
        if (isStartTag) {
            // 开始标签的处理 <div id="..." ...>
            const startMatch = parseStartTag(html);
            if (startMatch) {
                stratHandle(startMatch.tagName, startMatch.attrs);
                continue;
            }
            // 结束标签处理 </div>
            let endTag = html.match(endTagReg);
            if (endTag) {
                endHandle(endTag[1]);
                html = subHtml(endTag[0].length);
                continue;
            }
        }
    }
}
```

#### 3.3 匹配文本节点

如果我们在开始标签后，匹配的不是 `<` 字符（`isStartTag`为 false），那么就说明匹配到了元素内的文本节点，我们就可以获取该节点，并删除匹配到的文本节点：

```js
function parseHtml(html) {
    while (html) {
        // 判断是不是标签, 标签是以 < 开始的，例如：<div></div>，<br />
        let isStartTag = html.indexOf("<") === 0;

        // 开始/结束标签的处理
        if (isStartTag) {
            // 开始标签的处理 <div id="..." ...>
            const startMatch = parseStartTag(html);
            if (startMatch) {
                stratHandle(startMatch.tagName, startMatch.attrs);
                continue;
            }
            // 结束标签处理 </div>
            let endTag = html.match(endTagReg);
            if (endTag) {
                endHandle(endTag[1]);
                html = subHtml(endTag[0].length);
                continue;
            }
        }

        // 文本处理
        function textHandle(text) {
            console.log("文本", text);
            text = text.trim();
            text &&
                currentParent.children.push({
                    type: TEST_TYPE,
                    text,
                });
        }
    }
}
```

**完整流程**
![流程](https://img.zphl.top/blog/articleImg/98.png)

**至此，我们会循环匹配每个元素，直到元素文本为空，并将匹配到的文本转换为对象，那么如何将者一个个对象连接起来呢，这就是我们接下来要做的**

### 4.创建 AST

我们已经循环了每一个元素，并将他们转换为一个个对象，下面我们就将他们连接起来，形成一个树状结构

#### 4.1 开始标签处理

-   获取并创建当前节点

```js
let root,
    currentParent,
    stack = [];
const ELEMENT_TYPE = 1;
const TEST_TYPE = 3;
function createAST(tag, attrs) {
    return {
        tag,
        type: ELEMENT_TYPE,
        children: [],
        attrs,
        parent: null,
    };
}
function stratHandle(tag, attrs) {
    // 创建 ast 节点
    let node = createAST(tag, attrs);
}
```

-   判断有没有 root(根节点)，没有则说明刚刚解析，那么当前 node 就是 root 节点

```js
// 根节点
if (!root) {
    root = node;
}
```

-   如果有 root, 那说明当前 node 对象是上个节点的子节点，那么如何获取上个 node 呢，我们可以通过 stack 数组来获取，每次处理一次开始节点，我们就把该节点 push 到 stack 中，当处理结束节点的时候，说明该元素的所有内容匹配完成了，开始匹配下一元素了，我们就可以把该元素从 stack 中 pop 出来:

```js
// 节点栈，前面的是后面的父节点
stack.push(node);
```

stack 前一位元素是后一位元素的父元素

我们还需要判断是否存在父元素，存在，则当前 node 节点对象的 parent 是该父元素，而该父元素的 children 是该 node 对象，我们相互赋值：

```js
// 父节点，currentParrent 是stack前一位的节点，node是当前节点
if (currentParent) {
    node.parent = currentParent;
    currentParent.children.push(node);
}
```

在最后我们将当前 node 对象赋值给 currentParent，这样下一次循环获取元素里面的节点时，我们就可以获取到该节点的父节点:

```js
// 当前的父节点
currentParent = node;
```

**完整代码：**

```js
// 匹配标签名
const tagName = `[a-zA-Z_][\\-\\.0-9_a-zA-Z]*`;
// 匹配带命名空间的标签
const qnameCapture = `((?:${tagName}\\:)?${tagName})`;
// 匹配开始标签 例如 <div> 会匹配 <div
const stratTag = new RegExp(`^<${qnameCapture}`);
const endTagReg = new RegExp(`^<\\/${qnameCapture}[^>]*>`);
const attrReg = /^\s*([^\s"'<>\/=]+)(?:\s*(=)\s*(?:"([^"]*)"+|'([^']*)'+|([^\s"'=<>`]+)))?/;
// 匹配开始标签的结束：例如 <div> 匹配 >, 当 >或者 /> 开头则会匹配到，否则则不匹配, 例如<div> 则不匹配，用来判断是不是匹配到标签的结束
//位置
const stratTagClose = /^\s*(\/?)>/;
const defaultTagRE = /\{\{((?:.|\r?\n)+?)\}\}/g;

function parseHtml(html) {
    let root,
        currentParent,
        stack = [];
    const ELEMENT_TYPE = 1;
    const TEST_TYPE = 3;

    // 创建 ast 函数
    function createAST(tag, attrs) {
        return {
            tag,
            type: ELEMENT_TYPE,
            children: [],
            attrs,
            parent: null,
        };
    }

    // 开始标签处理
    function stratHandle(tag, attrs) {
        console.log("开始标签", tag);
        // 创建 ast 节点
        let node = createAST(tag, attrs);
        // 根节点
        if (!root) {
            root = node;
        }
        // 父节点，currentParrent 是stack前一位的节点，node是当前节点
        if (currentParent) {
            node.parent = currentParent;
            currentParent.children.push(node);
        }
        // 节点栈，前面的是后面的父节点
        stack.push(node);
        // 当前的父节点
        currentParent = node;
    }

    //则删除匹配完成的标签字符串，避免后面使用正则时重复匹配
    function subHtml(startIndex) {
        return html.substring(startIndex);
    }

    // 解析开始标签
    function parseStartTag() {
        // 匹配开始标签
        const start = html.match(stratTag);
        if (start) {
            // 匹配的结果
            const match = {
                tagName: start[1],
                attrs: [],
            };
            // 删除匹配完成的字符串
            const startIndex = start[0].length;
            html = subHtml(startIndex);

            //匹配属性
            let attr, end;
            // 匹配到结束标签，且没有匹配到属性时，退出循环
            while (!(end = html.match(stratTagClose)) && (attr = html.match(attrReg))) {
                match.attrs.push({
                    name: attr[1],
                    value: attr[3],
                });
                html = subHtml(attr[0].length);
            }
            // 匹配开始标签的结束标签 >
            if (end) {
                html = subHtml(end[0].length);
            }

            return match;
        }
    }

    while (html) {
        // 判断是不是标签, 标签是以 < 开始的，例如：<div></div>，<br />
        let isStartTag = html.indexOf("<") === 0;

        // 开始/结束标签的处理
        if (isStartTag) {
            // 开始标签的处理 <div id="..." ...>
            const startMatch = parseStartTag(html);
            if (startMatch) {
                // 解析为抽象语法树
                stratHandle(startMatch.tagName, startMatch.attrs);
                continue;
            }

        }
}
```

**流程图:**

![流程图](https://img.zphl.top/blog/articleImg/97.png)

#### 4.2 结束标签的处理

当我们匹配到结束标签时，则说明当前标签已经匹配完成，我们将该标签从 stack 数组中删除，并将 currenParent 设置为 stack 数组中的上一个 node 节点：

```js
// 结束标签处理
function endHandle(tag) {
    console.log("结束标签", tag);
    let node = stack.pop();
    currentParent = stack[stack.length - 1];
}
```

#### 4.3 文本节点的处理

文本节点一定是当前 node 节点的子节点，因为文本在标签内，那么它就是当前 node 对象 children 属性数组的元素，而在处理开始标签最后，我们将当前 node 赋值给 currenParent，那么在处理文本节点时，只需要将文本节点添加到当前 node 节点的 children 数组中即可：

```js
// 文本处理
function textHandle(text) {
    console.log("文本", text);
    text = text.trim();
    text &&
        currentParent.children.push({
            type: TEST_TYPE,
            text,
        });
}
```

现在所有一个个单独的 node 节点对象通过 parent 与 chidren 属性组成了抽象语法树，并赋值给了 root, complie 函数的完整代码:

```js
// 匹配标签名
const tagName = `[a-zA-Z_][\\-\\.0-9_a-zA-Z]*`;
// 匹配带命名空间的标签
const qnameCapture = `((?:${tagName}\\:)?${tagName})`;
// 匹配开始标签 例如 <div> 会匹配 <div
const stratTag = new RegExp(`^<${qnameCapture}`);
const endTagReg = new RegExp(`^<\\/${qnameCapture}[^>]*>`);
const attrReg = /^\s*([^\s"'<>\/=]+)(?:\s*(=)\s*(?:"([^"]*)"+|'([^']*)'+|([^\s"'=<>`]+)))?/;
// 匹配开始标签的结束：例如 <div> 匹配 >, 当 >或者 /> 开头则会匹配到，否则则不匹配, 例如<div> 则不匹配，用来判断是不是匹配到标签的结束
//位置
const stratTagClose = /^\s*(\/?)>/;
const defaultTagRE = /\{\{((?:.|\r?\n)+?)\}\}/g;

function parseHtml(html) {
    let root,
        currentParent,
        stack = [];
    const ELEMENT_TYPE = 1;
    const TEST_TYPE = 3;

    // 创建 ast 函数
    function createAST(tag, attrs) {
        return {
            tag,
            type: ELEMENT_TYPE,
            children: [],
            attrs,
            parent: null,
        };
    }

    // 开始标签处理
    function stratHandle(tag, attrs) {
        console.log("开始标签", tag);
        // 创建 ast 节点
        let node = createAST(tag, attrs);
        // 根节点
        if (!root) {
            root = node;
        }
        // 父节点，currentParrent 是stack前一位的节点，node是当前节点
        if (currentParent) {
            node.parent = currentParent;
            currentParent.children.push(node);
        }
        // 节点栈，前面的是后面的父节点
        stack.push(node);
        // 当前的父节点
        currentParent = node;
    }

    // 文本处理
    function textHandle(text) {
        console.log("文本", text);
        text = text.trim();
        text &&
            currentParent.children.push({
                type: TEST_TYPE,
                text,
            });
    }

    // 结束标签处理
    function endHandle(tag) {
        console.log("结束标签", tag);
        let node = stack.pop();
        currentParent = stack[stack.length - 1];
    }

    //则删除匹配完成的标签字符串，避免后面使用正则时重复匹配
    function subHtml(startIndex) {
        return html.substring(startIndex);
    }

    // 解析开始标签
    function parseStartTag() {
        // 匹配开始标签
        const start = html.match(stratTag);
        if (start) {
            // 匹配的结果
            const match = {
                tagName: start[1],
                attrs: [],
            };
            // 删除匹配完成的字符串
            const startIndex = start[0].length;
            html = subHtml(startIndex);

            //匹配属性
            let attr, end;
            // 匹配到结束标签，且没有匹配到属性时，退出循环
            while (!(end = html.match(stratTagClose)) && (attr = html.match(attrReg))) {
                match.attrs.push({
                    name: attr[1],
                    value: attr[3],
                });
                html = subHtml(attr[0].length);
            }
            // 匹配开始标签的结束标签 >
            if (end) {
                html = subHtml(end[0].length);
            }

            return match;
        }
    }

    while (html) {
        // 判断是不是标签, 标签是以 < 开始的，例如：<div></div>，<br />
        let isStartTag = html.indexOf("<") === 0;

        // 开始/结束标签的处理
        if (isStartTag) {
            // 开始标签的处理 <div id="..." ...>
            const startMatch = parseStartTag(html);
            if (startMatch) {
                stratHandle(startMatch.tagName, startMatch.attrs);
                continue;
            }
            // 结束标签处理 </div>
            let endTag = html.match(endTagReg);
            if (endTag) {
                endHandle(endTag[1]);
                html = subHtml(endTag[0].length);
                continue;
            }
        }

        // 标签内文本处理
        if (!isStartTag) {
            // 开始标签处理完成其删除，只需匹配结束标签的 < index，就可以截取文本
            const textEnd = html.indexOf("<");
            // 文本内容
            const text = html.substring(0, textEnd);
            if (text) {
                textHandle(text);
                // 删除文本字符串
                html = subHtml(text.length);
            }
        }
    }
    console.log("root", root);
}

export default function complie(html) {
    if (html) {
        // 解析 html
        parseHtml(html);
    }
}
```
