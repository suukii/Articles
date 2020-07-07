# 单例模式

单例模式就是一个 class 永远都返回同一个实例，而且这个实例还可以在全局中访问到。

听起来是不是很像一个全局变量？

## 单例模式 vs 全局变量

- 同样都是全局可访问，全局变量有被其他代码修改的风险，而单例模式提供的实例不能通过外部代码来修改替换，除非这个 ￼￼￼￼class 暴露了修改实例的方法。

- 单例模式还能封装一些代码逻辑。

## 实现思路

单例模式永远返回同一个实例对象，所以我们不能使用普通的构造函数来实现，因为每次 new 的时候都会创建一个新的实例。

> 其实如果是 JS 的普通构造函数语法的话，new 的时候如果函数中返回一个对象，那 new 出来的对象就会被丢弃，然后我们也可以把单例对象作为构造函数的静态属性缓存起来，不过这样我们也把单例对象暴露出去了。

我们得把构造函数和单例对象隐藏起来，然后另外暴露一个获取单例对象的方法。在这个方法中，我们需要实现的是：

1. 首次调用时，在方法中调用隐藏的构造函数，创建实例对象并缓存起来；
2. 之后的调用就直接返回缓存的实例对象；

## 实现代码

因为 JS 的 class 还不支持私有属性，所以我们先用一个 IIFE 来实现单例模式。

```js
let count = 0

const Singleton = (function () {
  // 缓存单例实例对象
  let instance = null

  // 只有首次调用 getInstance 方法时才会用 new 调用 Constructor 方法
  // 外部代码无法通过 new 调用 Constructor 创建实例对象
  // Contructor 只会执行一次
  function Constructor() {
    count++
  }

  function getInstance() {
    // 首次调用，创建单例实例并缓存
    if (!instance) {
      instance = new Constructor()
    }
    // 之后调用，直接返回缓存的实例对象
    return instance
  }
  // 暴露获取单例对象的方法
  return {
    getInstance
  }
})()

const a = Singleton.getInstance()
const b = Singleton.getInstance()
console.log(a === b) // true
console.log(count) // 1
```

## 优点

- 保证了一个 class 只有一个实例；
- 这个实例是全局可访问的；
- 惰性初始化，单例对象只有在首次获取的时候才会被创建，这也是和全局变量不同的一个点。
