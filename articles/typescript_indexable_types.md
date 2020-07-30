# TypeScript 学习笔记 - 任意属性 (Indexable Types)

-   [官方文档](https://www.typescriptlang.org/docs/handbook/interfaces.html#indexable-types)

我们在自定义类型的时候，有可能会希望一个接口允许有任意的属性签名，这时候 `任意属性` 就派上用场了。

任意属性有两种定义的方式：一种属性签名是 `string` 类型的，另一种属性签名是 `number` 类型的。

## string 类型任意属性

第一种，属性签名是 `string`，比如对象的属性：

```ts
interface A {
    [prop: string]: number;
}

const obj: A = {
    a: 1,
    b: 3,
};
```

`[prop: string]: number` 的意思是，`A` 类型的对象可以有任意属性签名，`string` 指的是对象的键都是字符串类型的，`number` 则是指定了属性值的类型。

`prop` 类似于函数的形参，是可以取其他名字的。

## number 类型任意属性

第二种，属性签名是 `number` 类型的，比如数组下标：

```ts
interface B {
    [index: number]: string;
}

const arr: B = ['suukii'];
```

`[index: number]: string` 的意思是，`B` 类型的数组可以有任意的数字下标，而且数组的成员的类型必须是 `string`。

同样的，`index` 也只是类似于函数形参的东西，用其他标识符也是完全可以的。

## 同时定义两种任意属性

需要注意的是，一个接口可以同时定义这两种任意属性，但是 `number` 类型的签名指定的值类型必须是 `string` 类型的签名指定的值类型的子集，举个例子：

```ts
interface C {
    [prop: string]: number;
    [index: number]: string;
}

// Numeric index type 'string' is not assignable to string index type 'number'.
```

上面定义是不成立的，因为 `index` 指定的值类型是 `string`，而 `prop` 指定的值类型是 `number`，`string` 并不是 `number` 的子集。

如果换成下面这样，定义就是成立的，因为 `Function` 是 `object` 的子集：

```ts
interface C {
    [prop: string]: object;
    [index: number]: Function;
}
```

## 同时定义任意属性和其他类型的属性

另外还有一个需要注意的点，**一旦定义了任意属性，那么其他属性(确定属性、可选属性、只读属性等)的类型都必须是它的类型的子集**。

比如说我们想要一个 `Person` 接口，它有一个必选属性 `name` 和一个可选属性 `age`，另外还可以有其他 `string` 类型的任意属性签名。那么 `Person` 接口可能会被定义成这样：

```ts
interface Person {
    name: string;
    age?: number;
    [prop: string]: string;
}

// Property 'age' of type 'number' is not assignable to string index type 'string'.
```

但其实这样子的定义是不成立的，因为 `[prop: string]: string` 的存在，规定了其他属性的类型也必须是 `string`，如果想要解决报错，我们可以使用联合类型：

```ts
interface Person {
    name: string;
    age?: number;
    [prop: string]: string | number;
}
```

对于 `number` 类型的任意属性签名，情况也是一样的：

```ts
type MyArray = {
    0: string;
    [index: number]: number;
};
// Property '0' of type 'string' is not assignable to numeric index type 'number'.
```

但是，`number` 类型的任意属性签名不会影响其他 `string` 类型的属性签名：

```ts
type Arg = {
    [index: number]: number;
    length: string;
};
```

如上，虽然指定了 `number` 类型的任意属性的类型是 `number`，但 `length` 属性是 `string` 类型的签名，所以不受前者的影响。

但是反过来就不一样了，如果接口定义了 `string` 类型的任意属性签名，它不仅会影响其他 `string` 类型的签名，也会影响其他 `number` 类型的签名。这一点可以参考**两种任意类型签名并存时，`number` 类型的签名指定的值类型必须是 `string` 类型的签名指定的值类型的子集**这句话。
