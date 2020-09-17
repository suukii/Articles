# 从零实现一个 Mini Vue

## 前置

1. [从零实现一个简单的 VDOM 引擎](./virtual_dom.md)
2. [从零实现一个响应式状态管理](./state_reactivity.md)

前两篇已经实现的功能：

1. `h` 创建 VNode
2. `mount` 挂载 VNode
3. `unmount` 移除 VNode
4. `patch` 替换 VNode

5. `observe` 将一个对象变成响应式

接下来就是将这些组合在一起。

## 使用场景

先来看看需求好了，我们的目标是实现一个计数器。

HTML 是这样子的：

```html
<div id="counter"></div>
<button id="inc">inc</button>
```

-   `#counter` 显示当前数字。
-   `#inc` 按钮被点击时当前数字要加一。

## 响应式状态

首先我们需要一个状态来存储当前数字。

```js
const counter = {
    count: 1,
};
observe(counter);
```

`#inc` 按钮被点击时 `count++`，这没什么好说的。

```js
const incBtn = $('#inc');
incBtn.addEventListener('click', () => {
    counter.count++;
});
```

## 页面更新

重要的是，`counter.count` 更新的时候，页面上的数字也要更新。

首先，就像在 Vue 中的一样，我们需要一个组件来显示计数器的数字。

```js
const counterComponent = {
    render(state) {
        return h('h1', {}, String(state.count));
    },
};
```

跟 Vue 中的 render 方法不同的是，我们这里的 render 方法接收一个响应式对象作为参数。

> 在 Vue 中，如果你使用的是模板写法，模板最终也是被编译成 render 方法的。

有了组件，接下来就是需要定义状态改变时要做的事情了。

```js
autorun(function () {
    // counter.count 改变了
});
```

这里有两种情况：

1. 初始状态时
2. 状态发生改变时

**1. 初始状态时**

在初始状态的时候，直接调用组件的 `render` 方法生成 VNode，然后挂载到 DOM 上就行：

```js
const node = counterComponent.render(counter);
mount(node, counterContainer);
```

**2. 状态发生改变时**

状态发生改变时，需要再一次调用组件的 `render` 方法生成新的 VNode，然后与旧的 VNode 对比，看哪些元素需要更新。

```js
const newNode = counterComponent.render(counter);
patch(oldNode, newNode);
```

组合起来就是这样：

```js
let oldNode = null;
autorun(function () {
    if (oldNode) {
        const newNode = counterComponent.render(counter);
        patch(oldNode, newNode);
        oldNode = newNode;
    } else {
        oldNode = counterComponent.render(counter);
        mount(oldNode, counterContainer);
    }
});
```

## 完整代码

[完整代码](https://gist.github.com/suukii/679ae4bfe41a1590bc8d885b5a05a88f)
