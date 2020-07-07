# 策略模式

## 问题

比如要压缩一张图片，可选择的压缩算法有 `jpg`、`png`、`gif` 等等。我们实现了一个 `ImageCompressor` class，提供了 `compress` 方法来处理这个任务，代码如下。

```js
class ImageCompressor {
  constructor() {}
  compress(img, algorithm) {
    let compressedImg = null
    switch (algorithm) {
      case 'jpg':
        // jpg 的压缩算法
        // compressedImg = ...
        break
      case 'png':
        // png 的压缩算法
        break
      case 'gif':
        // gif 的压缩算法
        break
      default:
        break
    }
    return compressedImg
  }
}

const imageCompressor = new ImageCompressor()
const pngImg = imageCompressor('exampleImageFile', 'png')
const jpgImg = imageCompressor('exampleImageFile', 'jpg')
```

可以看到在上面的代码中，所有压缩算法都是在 `compress` 方法中实现的，而在实际中，每种算法的逻辑都十分复杂，如果都写在同一个方法中，这个方法很快就会膨胀了。这种写法导致的问题就是，如果之后要修改或者新增某个算法，都很有可能会不小心影响到其他代码。

为了代码更容易维护，我们需要把这些算法拆分成独立的小块，并通过一个“代言人”来“管理”它们。

> 用“代言人”和“管理”好像不那么准确，但我又想不到什么词了，意会意会。

### 什么时候使用策略模式

从以上代码中可以观察到，`compress` 的输入和输出是差不多的：

- 输入一张图片
- 输出一张压缩后的图片

而不同的是：

- 压缩处理的方法

遇到这种模式的问题我们都可以使用`策略模式`来解决：

- 把不同的 `处理方法` 抽离成各自独立的 class
- 每一个 `处理方法` 就是一个解决问题的 `策略`
- 再通过一个 `Context` class 来提供统一的对外接口，`Context` 内部再调用不同的 `策略` 方法

## 一个简单例子

假设要写一个可以同时处理两数加减乘除的函数，我们可能会实现成以下的样子：

`doMath` 函数接收两个操作数 `a` 和 `b`，以及一个算术类型 `operation` 作为参数，然后根据不同的算术类型返回不同的数学计算结果。

```js
const doMath = (a, b, operation) => {
  switch (operation) {
    case 'ADD':
      return a + b
    case 'MINUS':
      return a - b
    case 'MULTIPLY':
      return a * b
    case 'DIVIDE':
      return a / b
    default:
      return
  }
}

doMath(1, 2, 'ADD') // 3
doMath(1, 2, 'MINUS') // -1
doMath(1, 2, 'MULTIPLY') // 2
doMath(1, 2, 'DIVIDE') // 0.5
```

可以观察到，多次调用 `doMath` 函数的共同点在于：

- 都是输入两个数字
- 得到这两个数字经过某种计算后的结果作为返回值

而不同点就在于：

- 具体的计算方式是不一样的

这个问题模式就很适合使用策略模式来解决。

## 套用策略模式

用 OOP 的形式来实现的话，我们先把上面的例子改写成 class 的形式吧，改写方式之一：

```js
class SimpleMath {
  constructor() {
    this.operations = {
      ['ADD']: (a, b) => a + b,
      ['MINUS']: (a, b) => a - b,
      ['MULTIPLY']: (a, b) => a * b,
      ['DIVIDE']: (a, b) => a / b
    }
  }
  calculate(a, b, operation) {
    return this.operations[operation](a, b)
  }
}
```

目前所有算法都是在 `SimpleMath` 中实现的，接下来我们尝试把每个算法抽离成独立的 class：

```js
// 不同算法被抽离成不同的 class
// 实现的效果是这些 class 的都应该有相同的实例方法，但这些实例方法做的事情不一样
// p.s. 按理说这些 class 都应该实现同样的 interface，但 JS 中并没有 interface
class Add {
  calculate(a, b) {
    return a + b
  }
}
class Minus {
  calculate(a, b) {
    return a - b
  }
}
class Multiply {
  calculate(a, b) {
    return a * b
  }
}
class Divide {
  calculate(a, b) {
    return a / b
  }
}

// 原本的 SimpleMath 就只保留一个对具体算法实例的引用 operation
// SimpleMath 并不关心具体使用什么算法，调用方在调用 SimpleMath 的时候把具体的算法对象传过来，SimpleMath 负责调用这个算法对象的 calculate 方法
class SimpleMath {
  constructor(operation) {
    // operation 保存着具体算法实例
    this.operation = operation
  }
  calculate(a, b) {
    // SimpleMath 的 calculate 方法只是负责调用具体算法实例的 calculate 方法
    return this.operation.calculate(a, b)
  }
}

// 调用方代码：
// 调用方必须知道自己需要的是哪种算法，并把对应的算法实例传给 SimpleMath
const add = new SimpleMath(new Add())
add.calculate(1, 2) // 3
const multiply = new SimpleMath(new Multiply())
multiply(1, 2) // 2
```

## 策略模式的构成

**问题**

- 策略模式中有几个角色？
- 每个角色分别负责什么？
- 各个角色之间是如何联系的？

**回答**

策略模式中有 3 个角色：

- Client: 调用方，是有某项任务需要完成的角色，可以有很多个；
- Context: 比如上方的 `SimpleMath`，是负责完成某项任务的角色，但只知道需要完成一项任务，并不知道任务具体应该如何完成，只有一个；
- Strategy: 完成任务的具体方法，可以有很多个；

Client 通过调用 Context 并指定某个 Strategy 来间接调用相应 Strategy 的某个方法。

> > 其实不用 class 来实现，只要是符合这种思想的都是策略模式吧，把 strategy 拆分成函数也行吧，个人看法个人看法。
