# 力扣的 TypeScript 面试题

## 前言

几个礼拜前我在 GitHub 上面看到了力扣的面试题仓库，其中有一道 TypeScript 题目，要求编写复杂类型定义。当时我对类型只是有一个泛泛的了解，所以这题目是看得一头雾水。最近稍微看了一下 TypeScript 的文档，尤其是这两天看了两个“高级”的用法，所以决定重新尝试一下这道题。题目原地址在[这里](https://github.com/LeetCode-OpenSource/hire/blob/master/typescript_zh.md)，下面是原题描述。

## 题目描述

假设有一个叫 `EffectModule` 的类

```ts
class EffectModule {}
```

这个对象上的方法**只可能**有两种类型签名:

```ts
interface Action<T> {
  payload?: T
  type: string
}

asyncMethod<T, U>(input: Promise<T>): Promise<Action<U>>

syncMethod<T, U>(action: Action<T>): Action<U>
```

这个对象上还可能有一些任意的**非函数属性**：

```ts
interface Action<T> {
    payload?: T;
    type: string;
}

class EffectModule {
    count = 1;
    message = 'hello!';

    delay(input: Promise<number>) {
        return input.then(i => ({
            payload: `hello ${i}!`,
            type: 'delay',
        }));
    }

    setMessage(action: Action<Date>) {
        return {
            payload: action.payload!.getMilliseconds(),
            type: 'set-message',
        };
    }
}
```

现在有一个叫 `connect` 的函数，它接受 EffectModule 实例，将它变成另一个对象，这个对象上只有**EffectModule 的同名方法**，但是方法的类型签名被改变了:

```ts
asyncMethod<T, U>(input: Promise<T>): Promise<Action<U>>  变成了
asyncMethod<T, U>(input: T): Action<U>
```

```ts
syncMethod<T, U>(action: Action<T>): Action<U>  变成了
syncMethod<T, U>(action: T): Action<U>
```

例子:

EffectModule 定义如下:

```ts
interface Action<T> {
    payload?: T;
    type: string;
}

class EffectModule {
    count = 1;
    message = 'hello!';

    delay(input: Promise<number>) {
        return input.then(i => ({
            payload: `hello ${i}!`,
            type: 'delay',
        }));
    }

    setMessage(action: Action<Date>) {
        return {
            payload: action.payload!.getMilliseconds(),
            type: 'set-message',
        };
    }
}
```

connect 之后:

```ts
type Connected = {
    delay(input: number): Action<string>;
    setMessage(action: Date): Action<number>;
};
const effectModule = new EffectModule();
const connected: Connected = connect(effectModule);
```

**要求：** 将下面代码中的 `any` 替换成题目的解答，让编译能够顺利通过。

```ts
// 修改 Connect 的类型，让 connected 的类型变成预期的类型
type Connect = (module: EffectModule) => any;
```

## 分析题目

我们来梳理一下题目，可以大概理解为有一个 `EffectModule` 类型，它有两种**函数签名**，还可以有任意的**非函数属性**，如下定义

```ts
type EffectModule<T, U> = {
    [prop: string]: any;
    <T, U>(input: Promise<T>): Promise<Action<U>>; // asyncMethod
    <T, U>(action: Action<T>): Action<U>; //syncMethod
};
```

另外有一个 `Connected` 类型，它只有 `EffectModule` 上的**同名方法**，但是方法的类型签名变了，而且它没有**非函数属性**。

```ts
type Connected = {
    <T, U>(input: T): Action<U>; // asyncMethod
    <T, U>(action: T): Action<U>; // syncMethod
};
```

`Connect` 是一个函数签名，它接收一个 `EffectModule` 类型的对象，返回一个 `Connected` 类型的对象。

```ts
type Connect = (module: EffectModule) => any;
```

我们要做的就是把 `any` 改成符合要求的类型定义。

总结一下，我们有两件事情要做：

1. 把 `Connect` 参数对象上的非函数属性去掉
2. 把 `Connect` 参数对象上的函数签名改成符合题目要求

## 开始解题

### 去除非函数属性

去除**非函数属性**，也就是挑出**函数属性**。

首先想一下，如果我们要基于原有类型创建新类型，要怎么做？比如

```ts
type MakeConnected<T> = {
    [P in keyof T]: T[P];
};
```

上面的代码只是定义了一个和原有类型一模一样的新类型，其中 `keyof T` 相当于遍历了类型 `T` 的所有属性。

如果我们把 `keyof T` 部分替换成**只包含函数属性的集合**，那不就完了吗？我们来改一下上面的代码

```ts
type MakeConnected<T> = {
    [P in PickFuncKey<T>]: T[P];
};
```

可惜 TypeScript 并没有提供一个 `PickFuncKey` 方法，所以我们来自己实现一个。

**要求：** `PickFuncKey` 接收一个类型参数 `T`，返回 `T` 中所有函数属性的 `key` 的集合。注意我们只需要 `key`。

这里有一个关键知识点就是：**如何判断属性是否函数属性？**

答案就是 `extends` 关键字。

`PickFuncKey<T>` 可以定义如下

```ts
type PickFuncKey<T> = {
    [K in keyof T]: T[K] extends Function ? K : never;
}[keyof T];
```

### 改变函数签名

第二步是改变方法的类型签名，我们的对象只有两种函数签名。如果是

```ts
<T, U>(input: Promise<T>): Promise<Action<U>>
```

就改成

```ts
<T, U>(input: T): Action<U>
```

如果是

```ts
<T, U>(action: Action<T>): Action<U>
```

就改成

```ts
<T, U>(action: T): Action<U>
```

当然首先我们得判断函数属性具体是哪种签名，这又用到了 `extends`。

p.s. 这里我把 `MakeConnected` 的形参改成了 `M`，便于区分。

```ts
type MakeConnected<M> = {
    [P in PickFuncKey<M>]: M[P] extends (
        input: Promise<T>,
    ) => Promise<Action<U>>
        ? TODO
        : M[P] extends (action: Action<T>) => Action<U>
        ? TODO
        : never;
};
```

首先用

```ts
M[P] extends (input: Promise<T>) => Promise<Action<U>>
```

来判断是不是 asyncMethod 的签名，是就返回处理后的函数签名(TODO)，不是的话就继续用

```ts
M[P] extends (action: Action<T>) => Action<U>
```

来判断符不符合 syncMethod 的函数签名，是就返回处理后的函数签名(TODO)，不是就返回 `never`。

接着就是如何**修改函数签名**了，首先第一个

```ts
(input: Promise<T>) => Promise<Action<U>>
```

要改成

```ts
(input: T) => Action<U>
```

需求简单明确，但问题是，`T` 和 `U` 都从哪里来？我们的 `MakeConnected` 只接收一个参数 `M`，而且，在每个函数属性的签名中， `T` 和 `U` 都是不一样的。

这时候 `infer` 关键字就派上用场了。

> 如果你不知道 `infer` 是干什么的，我这里有一篇[小笔记](https://github.com/suukii/Articles/blob/master/articles/typescript_infer.md)，或者去查下文档吧。

有了 `infer` 之后我们就可以这样修改函数签名了

```ts
M[P] extends (input: Promise<infer T>) => Promise<Action<infer U>> ? (input: T) => Action<U> : never
```

第二种函数签名同理

```ts
M[P] extends (action: Action<infer T>) => Action<infer U> ? (action: T): Action<U> : never
```

### 完整代码

```ts
type PickFuncKey<F> = {
    [P in keyof F]: F[P] extends Function ? P : never;
}[keyof F];

type MakeConnected<M> = {
    [P in PickFuncKey<M>]: M[P] extends (
        action: Action<infer T>,
    ) => Action<infer U>
        ? (action: T) => Action<U>
        : M[P] extends (input: Promise<infer T>) => Promise<Action<infer U>>
        ? (input: T) => Action<U>
        : never;
};

// 使用 `MakeConnected` 时传入 module 的类型
type Connect = (module: EffectModule) => MakeConnected<typeof module>;
```

## 后记

其实这样子分析完之后发现，这道题目也没有特别复杂嘛，主要还是自己学得太少。

完整的代码在[这里](https://github.com/suukii/daily/blob/master/collections/lc_hire_typescript.ts)。
