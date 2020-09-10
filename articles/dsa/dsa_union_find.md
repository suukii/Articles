# DSA - 并查集

## 基本概念

并查集(Union Find Data Structure)是一种树形数据结构，它可以用来处理一系列不交集(disjoint sets)的关系，具体点就是用来解决不交集的查找(Find)和合并(Union)问题。

**不交集**

不交集指没有交集的集合，比如集合 `{1}` 和集合 `{2,3}` 的交集是空，它们就是两个不交集。

**代表**

每个不交集都有一个代表，就是我们会从集合中选出一个元素来表示这个集合，这个元素就叫做**代表 representative**，比如我们为集合 `{2,3}` 选的代表是 `2`，集合中的每个元素都有一个“指针”指向这个集合的**代表**。

**操作**

我们来定义并查集中都有什么操作：

-   `Create-Set(x)`：用于创建一个不交集，集合中包含一个元素 `x`；比如调用 `Create-Set(1)` 之后我们就得到了一个集合 `{ 1 }`，这个集合的**代表**就是 `1`。

<br>

-   `Union-Set(x, y)`：用于将 `x 所在集合` 和 `y 所在集合` 合并为一个集合。假设我们有两个不交集 `{1}` 和 `{2}`，调用 `Union-Set(1, 2)` 之后，把 `{2}` 并入到 `{1}` 中，我们就得到了 `{1,2}`，新集合的**代表**还是 `1`，原来的 `{2}` 就被销毁了。就是说每一次 `Union-Set` 操作之后集合的总数就减一了。

<br>

-   `Find-Set(x)`：找到 `x 所在集合` 的**代表**，用于判断两个元素是否在同一个集合中，如果 `Find-Set(x)` 等于 `Find-Set(y)`，那么 `x` 和 `y` 就是在同一个集合中了。

## 实现

### 数组简单实现

-   用一个 `parents` 数组来记录每个元素的**代表**是谁。
-   合并集合 X 和 Y 时，把集合 Y 的**代表**指向集合 X 的**代表**。

TypeScript Code

```ts
class UnionFind {
    private parents: Array<number>

    constructor(size: number) {
        // 初始化 parents 数组
        // 一开始每个元素都在不同的集合中，每个元素的代表都是它们自己
        this.parents = Array(size)
            .fill(0)
            .map((_, i) => i)
    }

    findSet(x: number): number {
        // 递归地找到集合的代表
        if (x !== this.parents[x]) {
            return this.findSet(this.parents[x])
        }
        return this.parents[x]
    }

    unionSet(x: number, y: number): void {
        // 分别找到 x 和 y 所在的集合
        const rootx: number = this.findSet(x)
        const rooty: number = this.findSet(y)
        // 如果 x 和 y 在同一集合中，什么都不做
        if (rootx === rooty) return
        // 如果 x 和 y 在不同集合中，
        // 将 y 所在集合的代表指向 x 所在集合的代表
        this.parents[rooty] = rootx
    }
}
```

### 路径压缩 Path Compression

路径压缩这个优化是用在 `Find-Set(x)` 操作中的，指的是在查找过程中把结果缓存起来，只需要一行代码就可以实现：

`this.parents[x] = this.findSet(this.parents[x])`

因为在合并集合 X 和 Y 的时候，我们仅仅是把 Y 的**代表**指向了 X 的**代表**，比如：

-   集合 X：{1,2}，代表是 1
-   集合 Y：{3,4}，代表是 3

合并之后：{1, 2, 3, 4}，代表是 1

现在 3 指向了 1，不过原来集合 Y 中的其他元素还是指向原来的代表，也就是 4 还是指向 3。所以我们调用 `Find-Set(4)` 的时候，是先找到了 3，再顺着 3 找到 1，才能判断 4 在这个集合中。而**路径压缩**就是，当我们找到 1 的时候，顺便更新 4 的“指针”，让它也指向 1，这样下次查询的时候，查询路径就缩短了。

TypeScript Code

```ts
class UnionFind {
    // ...

    findSet(x: number): number {
        // 递归地找到集合的代表
        if (x !== this.parents[x]) {
-           return this.findSet(this.parents[x])
+           this.parents[x] = this.findSet(this.parents[x])
        }
        return this.parents[x]
    }
}
```

### Rank

Rank 优化是用于 `Union-Set` 操作的，如果要合并两个集合 X 和 Y，那我们是要把 X 合并到 Y 中，还是把 Y 合并到 X 中呢？

从上面路径压缩中我们看到了，合并两个集合之后，我们需要更新其中一个集合所有元素的“指针”，让它们都指向新集合的**代表**，这种情况下，我们当然是要把 `元素较少的集合` 合并到 `元素较多的集合` 中的，如果一样，那就随便了。

我们可以用另一个数组 `rank` 来记录每个集合的元素个数(树的深度)，在 `Union-Set` 操作时，总是把 `较矮的树` 合并到 `较高的树` 中。

TypeScript Code

```ts
class UnionFind {
    // ...
    private rank: Array<number>

    constructor(size: number) {
        // 初始化为 0
        this.rank = Array(size).fill(0)
    }

    unionSet(x: number, y: number): void {
        const rootx: number = this.findSet(x)
        const rooty: number = this.findSet(y)
        if (rootx === rooty) return

        // 把“较矮”的合并到“较高”的中
        if (this.rank[rootx] > this.rank[rooty]) {
            this.parents[rooty] = rootx
        } else {
            this.parents[rootx] = rooty
            // 如果两者高度一样，合并后更新“被并入的那个集合”的 rank
            this.rank[rootx] === this.rank[rooty] && ++this.rank[rooty]
        }
    }
}
```

### 复杂度分析

复杂度证明比较复杂，我不想头秃，就不看了，直接贴一下结论。

-   时间复杂度：$O(α(n))$，$α(n)$ 是 [inverse Ackermann function](https://en.wikipedia.org/wiki/Ackermann_function#Inverse)，不管 n 是多少，$α(n)$ 都小于 5。

## 相关资料

-   [x] [Using Disjoint Set (Union-Find) To Build A Maze Generator](https://coderscat.com/using-disjoint-set-union-find-to-create-maze)
-   [x] [Disjoint Set Data Structure](https://www.topcoder.com/community/competitive-programming/tutorials/disjoint-set-data-structures/)
