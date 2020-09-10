# TypeScript 学习笔记 - `infer`

先来看一个例子：

```ts
type FlattenIfArray<T> = T extends Array<infer R> ? R : T;
```

我们来分析一下以上代码：

1. 首先判断泛型 `T` 是否数组类型；
2. 如果是数组类型，将数组成员类型 `R` 抽出来并返回；
3. 如果不是数组类型，则原样返回 `T`；

由于数组类型的定义有两种写法，所以上面的代码还有另外一种写法：

```ts
type FlattenIfArray<T> = T extends (infer R)[] ? R : T;
```

再来看一个例子：

```ts
type Unpromisify<T> = T extends Promise<infer R> ? R : T;
```

1. 首先判断泛型 `T` 是否 Promise 的子集；
2. 如果是，将 Promise 中的泛型 `R` 抽出来并返回；
3. 如果不是则原样返回 `T`；

再来看一个复杂一点的例子：

```ts
type FunctionWithOneObjectArgument<P extends { [x: string]: any }, R> = (
    props: P,
) => R;

type DestructuredArgsOfFunction<
    F extends FunctionWithOneObjectArgument<any, any>
> = F extends FunctionWithOneObjectArgument<infer P, any> ? P : never;

const myFunction = (props: { x: number; y: number }): string => {
    return 'ok';
};

const props: DestructuredArgsOfFunction<typeof myFunction> = {
    x: 1,
    y: 2,
};
```

代码分析：

-   `FunctionWithOneObjectArgument` 接收两个泛型 `P` 和 `R`，然后返回一个函数签名 `(props: P) => R`，而 `P extends { [x: string]: any }` 这部分则将参数类型 `P` 限定为一个对象。

-   `DestructuredArgsOfFunction` 接收一个泛型 `F`，然后判断 `F` 是否 `FunctionWithOneObjectArgument<P, R>` 的子集，如果是子集则将其中的 `P` 类型抽出来返回，如果不是则返回 `never`

-   `typeof myFunction` 会返回 `myFunction` 的函数签名，也就是 `(props: { x: number; y: number }): string`

-   `DestructuredArgsOfFunction<typeof myFunction>` 就相当于 `DestructuredArgsOfFunction<(props: { x: number; y: number }): string>`，我们可以将它代入到 `DestructuredArgsOfFunction` 的类型定义中，容易看出 `infer P` 会将 `P` 推导为 `{ x: number; y: number }`，然后我们返回了 `P`，所以 `props` 变量的类型就是 `{ x: number; y: number }` 了。
