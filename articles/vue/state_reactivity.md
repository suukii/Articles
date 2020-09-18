# 从零实现一个响应式状态管理

## 概念

简单地说，响应式状态(state reactivity)是指当程序状态发生改变时，比如说某个变量的值发生了变化，就自动执行某些指定的操作。

这个功能主要分成两个部分：

1. 当一个变量发生改变时，它能发出通知。
2. 收集变量的依赖函数，即哪些函数对这个变量的变化是感兴趣的，把它们收集起来，变量改变时通知它们。

## Dep Class

### getter/setter

由于不存在变量“赋值钩子”这种东西，我们没法直接监听变量的修改情况，所以需要先把变量变成对象，用 `getter` 和 `setter` 来代替读取和赋值操作。

```js
class Dep {
    constructor(value) {
        this._value = value;
    }
    get value() {
        return this._value;
    }
    set value(val) {
        this._value = val;
    }
}
```

-   `Dep` 的实例是一个响应式的对象，目前它只提供了两个“钩子”(getter 和 setter)，让我们可以在读值和赋值的时候进行某些操作。

### notify

接下来我们来实现第一部分的功能，当变量值改变时，发出通知。

```js
class Dep {
    constructor() {
        this.subscribers = [];
    }
    notify() {
        this.subscribers.forEach(fn => fn());
    }
    set value(val) {
        this._value = val;
        this.notify;
    }
}
```

-   `subscribers` 里面存放着当前变量的依赖函数。
-   `notify`，通知各个依赖函数，在 setter 钩子里触发 `notify`，执行各个依赖函数。

### depend

然后是收集依赖函数的工作。很简单，在 value 的 getter “钩子”里收集就行。

唯一的问题是，`depend` 中的依赖函数 `func` 是从哪里传进来的，`this.depend()` 是在 `get value` 中调用的，但 getter 也不能传参数呀。

```js
class Dep {
    constructor() {
        this.subscribers = new Set();
    }
    depend() {
        this.subscriber.add(func);
    }
    get value() {
        this.depend();
        return this._value;
    }
}
```

这一点比较 tricky。在 `get value` 执行之前，执行流是在某个依赖函数 `funcA` 的内部，这时我们用一个全局变量把 `funcA` 存起来，接着在执行 `depend` 的时候读取那个全局变量就能得到 `funcA` 了。

```js
const count = new Dep(1);

let activeUpdate = null;

function update() {
    activeUpdate = update;
    console.log(count.value);
}

update();
activeUpdate = null;

setTimeout(() => (count.value = 2), 2000);
```

-   `activeUpdate`，存放当前依赖函数的全局变量。
-   `update`，变量 `count` 的一个依赖函数

但是我们不能在每个依赖函数中都写上这句 `activeUpdate = update;` 吧。

上面代码需要重构一下，用一个 `autorun` 函数来完成依赖函数收集的功能，这样就不需要修改依赖函数本身的逻辑了。

```js
const count = new Dep(1);

let activeUpdate = null;
function autorun(update) {
    activeUpdate = update;
    update();
    activeUpdate = null;
}

function update() {
    console.log(count.value);
}
autorun(update);
```

到这里就差不多了，我们已经实现了对一个原始值变量的监听，不过我们在用 Vue 的时候，监听的是 `data` 对象的所有属性。

对于对象属性，JS 有提供一个 `Object.defineProperty` API，可以直接将这个属性改成 getter 和 setter。

```js
function observe(obj) {
    Object.keys(obj).forEach(key => {
        let internalValue = obj[key];
        const dep = new Dep();
        Object.defineProperty(obj, key, {
            get() {
                dep.depend();
                return internalValue;
            },
            set(newVal) {
                internalValue = newVal;
                dep.notify();
            },
        });
    });
}

class Dep {
    constructor() {
        this.subscribers = new Set();
    }
    depend() {
        activeUpdate && this.subscribers.add(activeUpdate);
    }
    notify() {
        this.subscribers.forEach(func => func());
    }
}
```

-   `observe` 函数把一个对象的所有属性都变成响应式的。
-   `Dep` 就剩下 `depend` 和 `notify` 两个功能了。

## 完整代码

[完整代码](https://gist.github.com/suukii/ed55d5d1b155ea01ca3a35fd52cd30ce)

> PS. 如果有人对完整代码中的 autorun 函数有疑问的话，可以看下[这里](https://github.com/suukii/frontendmasters/blob/master/advanced-vuejs-features-from-the-ground-up/1-reactivity.md#%E8%AE%A8%E8%AE%BA)，注意，个人理解而已。
