# 从零实现一个简单的 VDOM 引擎

## 要实现的功能

1. `h` 创建 VNode
2. `mount` 挂载 VNode
3. `unmount` 移除 VNode
4. `patch` 替换 VNode

## 什么是 VDOM？

简单复习一下概念，VDOM 就是用 JS 对象来描述真实的 DOM；相比真实 DOM，VDOM 没有那么多属性，操作起来开销更小。

VNode 则是 VDOM 的组成部分，如下就是一个 VNode：

```js
const node = {
    tag: 'div',
    props: {
        id: 'title',
    },
    children: [
        {
            tag: 'h1',
            props: {},
            children: 'Hello World',
        },
    ],
};
```

这个简单的 VNode 包括了：

-   `tag` 属性，用来描述 HTML 标签。
-   `props`，用来描述标签的属性。
-   `children`，0 或多个子元素，子元素既可以是 VNode 数组，也可以是字符串。

它描述的是这样一段 HTML：

```html
<div id="title">
    <h1>Hello World</h1>
</div>
```

## VDOM 引擎要实现哪些功能？

**1. 创建 VNode**

VNode 是 VDOM 的砖块，所以首先我们需要一个可以生成 VNode 的函数。

```js
function h(tag, props, children) {}
```

> 这个函数叫 h 只是一个传统。

**2. mount**

负责把 VNode 挂载到指定的 DOM 节点上，这样 VNode 的内容才能显示在页面上。

```js
function mount(vnode, container) {}
```

**3. unmount**

把 VNode 对应的那个 DOM 元素从 DOM 中移除。

```js
function unmount(vnode) {}
```

**4. patch**

将新的 VNode(n2) 和旧的 VNode(n1) 进行比较，找出不同的地方，替换掉。

```js
function patch(n1, n2) {}
```

## 1. 创建 VNode

这个函数比较简单，返回一个 JS 对象就行。

> 这里的 VNode 只有 3 个属性，是非常简易的实现。

```js
function h(tag, props, children) {
    return {
        tag,
        props,
        children,
    };
}
```

像这样就生成了一个表示 `span` 元素的 VNode：

```js
const span = h('span', {}, 'Hello World!');
```

## 2. mount

把 VNode 挂载到 DOM 元素上。

-   首先要为这个 VNode 新建一个元素；
-   然后给元素设置属性和子元素；
-   最后把这个新建的元素 `append` 到指定的 DOM 元素中。

```js
function mount(vnode, container) {
    const { tag, props, children } = vnode;

    vnode.el = document.createElement(tag);

    setProps(vnode.el, props);
    setChildren(vnode.el, children);

    container.appendChild(vnode.el);
}
```

给元素设置属性 `setProps` 函数很简单：

```js
function setProps(ele, props) {
    for (const [key, value] of Object.entries(props)) {
        ele.setAttribute(key, value);
    }
}
```

给元素设置子元素的函数也不复杂，这里只考虑两种情况：

1. 如果 `children` 是字符串，就直接设置其为元素的文本节点。
2. 如果 `children` 是 VNode 数组，就递归地挂载这些 VNode。

```js
function setChildren(el, children) {
    if (typeof children == 'string') {
        el.textContent = children;
    } else {
        children.forEach(child => mount(child, el));
    }
}
```

## 3. unmount

把 VNode 对应的那个 DOM 元素从 DOM 中移除。

```js
function unmount(vnode) {
    vnode.el.parentNode.removeChild(vnode.el);
}
```

## 4. patch

对比新旧两个 VNode，找出不同的地方，替换掉。

`n1` 表示旧的节点，`n2` 表示新的节点。

```js
function patch(n1, n2) {
    // 用 n2 替换 n1

    const el = n1.el;
    n2.el = el;
}
```

如果两个节点的标签类型都不一样，我们就简单粗暴地整个替换掉。

```js
if (n1.tag !== n2.tag) {
    mount(n2, el.parentNode);
    unmount(n1);
}
```

如果两个节点标签一样，我们考虑两种情况：

1. 新节点的子节点是一个简单的字符串。
2. 新节点的子节点是一个 VNode 数组。

**对于第一种情况**

直接替换掉原标签的文本内容并设置新的标签属性就行

```js
if (typeof n2.children == 'string') {
    el.textContent = n2.children;
    setProps(el, n2.props);
}
```

**对于第二种情况**

我们定义一个 `patchChildren` 函数来处理 VNode 数组。

```js
patchChildren(n1, n2);
```

`patchChildren` 函数主要做这几件事情：

1. 如果新旧节点的子节点数量相同，那就分别对它们调用 `patch` 来处理。
2. 旧节点多出来的子节点 `unmount` 掉。
3. 新节点多出来的子节点 `mount` 上。

## 完整代码

[完整代码](https://gist.github.com/suukii/74763ff725470c2811a13a9dc516d371)
