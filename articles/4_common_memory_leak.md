# 4种常见的内存泄漏及解决方法

内存泄漏，简单地说就是，有一些数据明明不会再被用到，但因为有变量引用着它们，所以一直占据着内存，导致内存没有及时被释放。下面来看下哪些情况会导致内存泄漏。

## I. 闭包引起的内存泄漏

```js
window.onload = onWindowLoaded() {
  let button = document.querySelector('button')
  button.bigString = new Array(1000).join(new Array(1000).join('some big data'))
  button.onclick = onButtonClicked() {
    console.log('糟糕，内存泄漏了')
  }
}
```

在上面的代码中，`onWindowLoaded` 函数做了 3 件事情：

1. 创建 `button` 变量并引用了 `<button>` DOM 对象；
2. 给 `button` 对象添加了属性 `bigString` 并存放了大量数据；
3. 监听 `<button>` 元素的 `onclick` 事件；

预想中，`onWindowLoaded` 完成以上 3 个操作之后就结束了，它的环境对象也应该被销毁了，`button` 占用的内存也应该被回收释放。

但事实上，`onWindowLoaded` 的环境对象没有被销毁，`button` 也没有被回收，原因是，`onButtonClicked` 回调函数是一个闭包，它保存了对它外部作用域的引用，也就是 `onWindowLoaded` 的作用域。即使在 `onButtonClicked` 中我们并不需要使用 `button` 变量，但它还是被保留在内存中了。那应该怎样才能让垃圾收集器知道 `button` 引用的对象是可以被回收的呢？

### 方法1：手动释放内存

在 `button` 不再被需要的时候手动把它设为 `null`，这样垃圾收集器就可以把 `button` 之前引用的对象回收了。

```js
window.onload = onWindowLoaded() {
  let button = document.querySelector('button')
  button.bigString = new Array(1000).join(new Array(1000).join('some big data'))
  button.onclick = onButtonClicked() {
    console.log('很好，没有内存泄漏')
  }
  button = null
}
```

### 方法2：借助另一个函数

```js
window.onload = onWindowLoaded() {
  function onButtonClicked() {
    console.log('很好，没有内存泄漏')
  }
  (function anotherFn() {
    let button = document.querySelector('button')
    button.bigString = new Array(1000).join(new Array(1000).join('some big data'))
    button.onclick = onButtonClicked
  })()
}
```

现在 `button` 是在 `anotherFn` 的环境变量中了，而 `anotherFn` 中没有闭包，所以执行结束之后它的环境变量就被回收了，`button` 的内存也会被释放。而 `onButtonClicked` 函数，由于是在 `onWindowLoaded` 函数中定义的，所以只会保留对 `onWindowLoaded` 作用域的引用，它没有访问 `anotherFn` 作用域的权限。

### 方法3：不使用闭包

```js
function onButtonClicked() {
  console.log('很好，没有内存泄漏')
}
window.onload = onWindowLoaded() {
  let button = document.querySelector('button')
  button.bigString = new Array(1000).join(new Array(1000).join('some big data'))
  button.onclick = onButtonClicked
}
```

## II. 全局变量引起的内存泄漏

* 显性声明全局变量

如果是显性地声明了全局变量并引用了比较大的数据，最好在确定变量不再被需要后手动把它设为 `null` 释放内存。

```js
let bigData = new Array(1000).join(new Array(1000).join('some big data'))
// do sth with bigData
// don't need bigData anymore
bigData = null
```

* 隐性声明全局变量

有两种情况可能会导致无意中声明了全局变量，不过，这两种情况在严格模式中都可以避免：

1. 在函数中采用不使用 `var`, `let`, `const` 的方式声明变量，导致隐性地创建了全局变量。

```js
function foo() {
  bigData = new Array(1000).join(new Array(1000).join('some big data'))
}
```

2. 在函数中给 `this` 添加属性然后单独调用函数，`this` 默认为 `window`，无意中创建了全局变量。

```js
function foo() {
  // 相当于 window.bigData = new Array(1000).join(new Array(1000).join('some big data'))
  this.bigData = new Array(1000).join(new Array(1000).join('some big data'))
}
foo()
```

## III. 定时器/回调引起的内存泄漏

* 定时器

```js
const bigData = new Array(1000).join(new Array(1000).join('some big data'))
setInterval(function () {
  let button = document.querySelector('button')
  if (button) {
    console.log(bigData)
  }
}, 1000)

setTimeout(function () {
  let button = document.querySelector('button')
  button.parentElement.removeChild(button)
}, 5000)
```

上面这段代码：

1. 设置了一个定时器每隔 1 秒去检查 `<button>` 元素是否还存在 DOM 树中，如果在就打印 `bigData`；
2. 设置了一个定时器 5 秒后移除 `<button>` 元素；

显然 5 秒后 `setInterval` 中的代码就没什么意义了，如果这个定时器忘了被取消，由于它的内部引用了 `bigData`，会导致 `bigData` 也没法被回收，造成内存泄漏。

* 回调函数

同样的，在事件处理回调中也可能会有同样的问题，所以在移除元素之前最好先把它的事件监听移除(不过这一步现代浏览器已经替我们处理了)。

```js
const bigData = new Array(1000).join(new Array(1000).join('some big data'))
const button = document.querySelector('button')
button.addEventListener('click', function () {
  console.log(bigData)
})

setTimeout(function () {
  let button = document.querySelector('button')
  button.parentElement.removeChild(button)
}, 5000)
```

## 移除 DOM 元素

```js
let td = document.querySelector('td')
td.remove()
```

- 变量 `td` 引用了 `<td>` 元素；
- `<td>` 元素被从 DOM 树中移除；

但是，这个 `<td>` 元素对象并没有被回收，因为 `td` 变量还在引用着它，又是一个内存泄漏。

但是，这里可不是一个元素对象没有及时被回收这么简单。如果我们移除的是整个 `<table>` 元素，但因为 `td` 变量保留着对 `<td>` 元素的引用，连带着 `<td>` 对象引用的 `parentElement` 以及它的 `parentElement` 等等都会被保留，也就是说，保留对一个 `<td>` 元素对象的引用会导致整个被移除的 `<table>` 都会被保留。所以移除 DOM 元素的同时最好还要把对它的变量引用也移除 `td = null`。
