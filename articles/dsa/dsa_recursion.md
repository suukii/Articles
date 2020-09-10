# DSA - 递归

> 本文是对力扣探索卡片 [递归 I](https://leetcode-cn.com/explore/orignial/card/recursion-i/260/conclusion/) 的学习记录。

## 定义

> 递归算法是一种直接或者间接调用自身函数或方法的算法。

递归算法的实质是把问题分解成“缩小版”**子问题**，子问题和原题属于**同类问题**，然后通过递归先求出子问题的解，基于子问题的解最终得到原问题的解。

举个例子，比如原题要求 `F(二叉树)`，从这个问题可以拆分出两个子问题： `F(左子树)`、`F(右子树)`，同样这两个子问题也可以以同样的方式继续拆分，重复这个步骤一直拆分到可解的子问题，再自底向上找到原问题的解。

## 递归要素

1. 一个问题的解可以分解成 N 个子问题的解。
2. 子问题的求解思路除了规模大小之外，与原题没有任何区别。
3. 存在递推关系(recurrence relation)，就是一组规则，说明如何将原题的解拆分为子问题的解。
4. 存在递归终止条件(base case)。

## 递归练习：反转字符串

[344. 反转字符串](https://leetcode-cn.com/problems/reverse-string/)

### 思路

题目要求很简单：将输入的字符串数组原地反转。

**1. 拆解子问题**

让我们来看下原题可以拆分成怎样的子问题：

要反转整个数组，我们必须要将 `s[0]` 和 `s[n-1]` 进行交换，然后将 `s[1]` 和 `s[n-2]` 进行交换，一直重复这个步骤，直到完成数组反转。

从以上思路可以看出来，在递归中我们需要使用两个指针 `l` 和 `r` 来指定要进行交换的两个元素下标，在每次递归中，只需要执行 `swap(s[l], s[r])` 操作，然后将 `l` 和 `r` 分别右移和左移，继续递归。

**2. 递归终止条件**

-   当两个指针指向同一个元素时，不需要交换，终止递归。
-   当 `r` 指针小于 `l` 指针时，说明所有元素都已经完成了交换，终止递归。

### 复杂度分析

-   时间复杂度：$O(N)$，N 为字符串的长度，一共执行了 $N/2$ 次交换。
-   空间复杂度：$O(N)$，N 为字符串的长度，递归过程中使用的栈空间。

### 代码

JavaScript Code

```js
/**
 * @param {character[]} s
 * @return {void} Do not return anything, modify s in-place instead.
 */
var reverseString = function (s, l = 0, r = s.length - 1) {
    if (l >= r) return
    ;[s[l], s[r]] = [s[r], s[l]]
    reverseString(s, ++l, --r)
}
```

## 递归练习：杨辉三角

[118. 杨辉三角](https://leetcode-cn.com/problems/pascals-triangle/)

### 思路

**子问题和 base case**

子问题就很明显，`F(N)` 可以被拆分为 `F(N - 1)`, `F(N - 2)`, ..., `F(2)`, `F(1)`，其中 `F(1)` 就是 base case。我们在每步递归中构造出一行，然后推入结果数组，在构造完第 N 行后就得到了原问题的解。

**递推关系**

第 i 行的数字基于第 i-1 行的数字，具体点是第 i 行第 j 个数字等于第 i-1 行的第 j-1 和第 j 个数字的和：

`pascal[i-1][j] = pascal[i-1][j] + pascal[i-1][j-1]`

不过有两个特殊的情况，分别是每一行的首尾，它们都是 1。

### 复杂度分析

-   时间复杂度：$O(numRows^2)$。进行了 $numRows$ 次递归，每次递归中遍历数组的时间消耗分别是 $1 + 2 + ... + numRows$，高斯求和去常数之后结果是 $O(numRows^2)$；所以最终时间复杂度是 $O(numRows^2)$
-   空间复杂度：$O(numRows^2)$。递归过程中使用的栈空间是 $O(numRows)$；每一行数组消耗的空间是 $O(N)$，N 是行数，所以结果数组所占的总空间是 $1 + 2 + ... + numRows$，也就是 $O(numRows^2)$；所以最终空间复杂度是 $O(numRows^2)$。

### 代码

JavaScript Code

```js
/**
 * @param {number} numRows
 * @return {number[][]}
 */
var generate = function (numRows, res = []) {
    if (numRows <= 0) return []

    // base  case
    if (numRows === 1) {
        res.push([1])
        return res
    }

    // 当前行的构造依赖于上一行
    // 所以要先递归构造出上一行
    generate(numRows - 1, res)

    // 拿到上一行的结果
    // 注意数组是以 0 为初始下标，所以上一行的索引是 numRows - 2
    const lastRow = res[numRows - 2]

    // 开始构造当前行
    const row = Array(numRows).fill(0)
    // 特殊情况：数组首尾都是 1
    row[0] = row[numRows - 1] = 1
    // 根据递推关系构造当前行
    for (let i = 1; i < numRows - 1; i++) {
        row[i] = lastRow[i - 1] + lastRow[i]
    }
    // 加入结果中，最后返回结果
    res.push(row)
    return res
}
```

## 递归练习：杨辉三角 II

[119. 杨辉三角 II](https://leetcode-cn.com/problems/pascals-triangle-ii/)

### 思路

思路和上一题相差无几，先通过递归拿到上一行的数字，然后基于上一行计算当前行的数字。

不同的是本题只要求返回第 k 行，而且为了 $O(k)$ 的空间复杂度，我就直接原地修改上一行数组了。

另外还有一个小细节，本题的杨辉三角是从第 0 行开始的。

### 复杂度分析

-   时间复杂度：$O(k^2)$。
-   空间复杂度：$O(k)$。

### 代码

JavaScript Code

```js
/**
 * @param {number} rowIndex
 * @return {number[]}
 */
var getRow = function (rowIndex) {
    if (rowIndex < 0) return []
    if (rowIndex === 0) return [1]

    const row = getRow(rowIndex - 1)

    let temp = 1
    for (let i = 1; i < rowIndex; i++) {
        const last = temp
        temp = row[i]
        row[i] += last
    }
    row.push(1)

    return row
}
```

## 递归练习：反转链表

[206. 反转链表](https://leetcode-cn.com/problems/reverse-linked-list/)

### 思路

**子问题**

要将以 `head` 为头部的链表进行反转，我们可以先考虑将以 `head.next` 为头部的这一段链表进行反转，进一步拆分的话，那就是先反转 `head.next.next`, ..., 直到链表的最后一个节点。

**递归终止条件**

链表最后一个节点，或者空节点。

**递推关系**

要找到 `F(head)` 与 `F(head.next)` 的关系，`F` 是一个抽象函数，它的功能是反转链表并返回新链表的头部。

也就是说，当我们递归解决完 `F(head.next)` 时，已经得到了反转后链表的头部，那回到 `F(head)` 这一层递归时，我们只需要把 `head` 拼接到新链表中去，然后直接返回新链表头部就好。

### 复杂度分析

-   时间复杂度：$O(N)$，N 为链表长度。
-   空间复杂度：$O(N)$，N 为链表长度，递归栈的空间。

### 代码

JavaScript Code

```js
/**
 * Definition for singly-linked list.
 * function ListNode(val) {
 *     this.val = val;
 *     this.next = null;
 * }
 */
/**
 * @param {ListNode} head
 * @return {ListNode}
 */
var reverseList = function (head) {
    if (!head || !head.next) return head

    const reversedHead = reverseList(head.next)
    head.next.next = head
    head.next = null

    return reversedHead
}
```

## 记忆化递归

### 递归中的重复计算

递归中可能存在很多重复的计算，举个简单的例子，斐波那契数列：

`F(4) = F(3) + F(2) = (F(2) + F(1)) + F(2)`

在上面这个简单的例子中 `F(2)` 被重复计算了，随着 N 的值增加，重复的计算也会增多。

虽然递归算法非常直观有效，但过多的重复计算也可能会导致性能问题。

### 记忆化

记忆化(memoization) 是用来避免递归性能损失的一种常用优化手段，比如我们可以用哈希表来**缓存**已经计算过的 `F(N)`，以额外的空间来换取计算时间的减少。

## 记忆化递归练习：斐波那契数

[509. 斐波那契数](https://leetcode-cn.com/problems/fibonacci-number/)

### 思路

把计算结果存在一个哈希表中，计算 `fib(N)` 时，如果 `N` 存在于哈希表中，可以直接返回缓存结果而无需重复计算。

### 复杂度分析

-   时间复杂度：$O(n)$。每个递归函数中都有 2 次递归调用，所以递归树是一个二叉树，进行优化前，我们相当于把二叉树的节点都访问了一遍，所以时间复杂度是 $O(2^n)$。使用记忆化优化后，时间复杂度降低为 $O(n)$，也就是二叉树的高度。
-   空间复杂度：$O(n)$，递归栈的空间是 $O(n)$，哈希表的空间也是 $O(n)$。

### 代码

JavaScript Code

```js
const cache = {
    map: {},
    has(N) {
        return N in this.map
    },
    getC(N) {
        return this.map[N]
    },
    put(N, val) {
        this.map[N] = val
        return val
    },
}

/**
 * @param {number} N
 * @return {number}
 */
var fib = function (N) {
    if (N === 0) return 0
    if (N === 1) return 1
    if (cache.has(N)) return cache.getC(N)
    return cache.put(N, fib(N - 1) + fib(N - 2))
}
```

**类似题目**

[70. 爬楼梯](https://leetcode-cn.com/problems/climbing-stairs/)

## 递归练习：二叉树的最大深度

[题目链接：104. 二叉树的最大深度](https://leetcode-cn.com/problems/maximum-depth-of-binary-tree/)

**[题解](https://github.com/suukii/91-days-algorithm/blob/master/basic/day-13.md#%E6%96%B9%E6%B3%95-1%E9%80%92%E5%BD%92)**

## 递归练习：Pow(x, n)

[50. Pow(x, n)](https://leetcode-cn.com/problems/powx-n/)

### 思路

根据之前的递归思路，我们很容易可以写出

```js
/**
 * @param {number} x
 * @param {number} n
 * @return {number}
 */
var myPow = function (x, n) {
    const helper = (x, n) => {
        if (n === 0) return 1
        return x * helper(x, n - 1)
    }

    return n > 0 ? helper(x, n) : 1 / helper(x, -n)
}
```

定义一个 `helper` 来处理递归计算幂，n 为负数的情况在外层函数处理。

但是这种写法是不能 AC 的，当 n 的值非常大的时候，会发生调用栈溢出错误。所以每次递归的时候 n 不能只减一，这样递归层次太深。

**快速幂算法**

每次递归 n 都除以 2：

-   当 n 为偶数时，递归结果再平方就是我们要的结果了；
-   当 n 为奇数时，递归结果平方后再乘一个 x 就是我们要的结果。

JavaScript Code

```js
/**
 * @param {number} x
 * @param {number} n
 * @return {number}
 */
var myPow = function (x, n) {
    const helper = (x, n) => {
        if (n == 0) return 1
        // 为什么要用 Math.floor：
        // n 为奇数时先考虑 n-1 那部分
        // 递归计算 (n-1)/2
        // 递归结果平方后再乘一个 x
        const r = helper(x, Math.floor(n / 2))
        return n % 2 === 0 ? r ** 2 : r ** 2 * x
    }

    return n >= 0 ? helper(x, n) : 1 / helper(x, -n)
}
```

### 复杂度分析

-   时间复杂度：$O(log n)$，每次递归都会将问题规模减半。
-   空间复杂度：$O(log n)$，递归栈的空间。

## 递归练习：合并两个有序链表

[21. 合并两个有序链表](https://leetcode-cn.com/problems/merge-two-sorted-lists/)

### 思路

首先我们有一个 `mergeTwoLists` 方法，它的功能就是合并两个链表，并返回新链表的头部。

那么子问题是什么呢？有两种情况：

-   `mergeTwoLists(l1.next, l2)`：我们选定 `l1` 节点作为新链表的头，然后递归地去合并 `l1.next` 这部分和 `l2`。
-   `mergeTwoLists(l1, l2.next)`：我们选定 `l2` 节点作为新链表的头，然后递归地去合并 `l2.next` 这部分和 `l1`。

至于选择哪种，取决于 `l1` 和 `l2` 节点值的大小。递归结束后再将新链表头和递归结果连接起来。

### 复杂度分析

-   时间复杂度：$O(n + m)$，n 和 m 分别是两个链表的长度。`mergeTwoLists` 每次最多有 1 次递归，因此时间复杂度取决于合并后链表的长度。
-   空间复杂度：$O(n + m)$，n 和 m 分别是两个链表的长度。

### 代码

JavaScript Code

```js
/**
 * Definition for singly-linked list.
 * function ListNode(val, next) {
 *     this.val = (val===undefined ? 0 : val)
 *     this.next = (next===undefined ? null : next)
 * }
 */
/**
 * @param {ListNode} l1
 * @param {ListNode} l2
 * @return {ListNode}
 */
var mergeTwoLists = function (l1, l2) {
    if (!l1) return l2
    if (!l2) return l1
    if (l1.val < l2.val) {
        l1.next = mergeTwoLists(l1.next, l2)
        return l1
    } else {
        l2.next = mergeTwoLists(l1, l2.next)
        return l2
    }
}
```

## 递归练习：第 K 个语法符号

[779. 第 K 个语法符号](https://leetcode-cn.com/problems/k-th-symbol-in-grammar/)

### 思路

我首先试了最直觉的方法，递归地构造每一行的数字。不过因为每一行数字都翻倍了，这种方法毫不意外报了调用栈溢出错误。

接着我观察了一下，发现如果将第 n 行的数字两两分组，那每一组就分别对应第 n-1 行的一个数字。尝试将第 n 行的下标和第 n-1 行的下标对应起来，那就是

`row[n][k] = row[n - 1][((k + 1) / 2) >> 0]`

也就是说，第 n 行的第 `k` 个数字，取决于上一行的第 `(k + 1) / 2` 个数字。而且，每组中的第一个数字(下标为奇数)与上一行的数字相同，第二个数字(下标为偶数)与上一行的数字相反。

> 相反的意思是 0 对应 1，1 对应 0。

### 复杂度分析

-   时间复杂度：$O(N)$。
-   空间复杂度：$O(N)$。

### 代码

JavaScript Code

```js
/**
 * @param {number} N
 * @param {number} K
 * @return {number}
 */
var kthGrammar = function (N, K) {
    if (N === 1) return 0
    const l = kthGrammar(N - 1, ((K + 1) / 2) >> 0)
    // 奇数相同，偶数相反
    return K & 1 ? l : l ^ 1
}
```

逐行构造的解法，不能 AC

```js
/**
 * @param {number} N
 * @param {number} K
 * @return {number}
 */
var kthGrammar = function (N, K) {
    const helper = N => {
        if (N === 1) return '0'
        const last = helper(N - 1)
        return last.replace(/0|1/g, s1 => (s1 === '0' ? '01' : '10'))
    }
    return helper(N)[K - 1]
}
```

## 递归练习：不同的二叉搜索树 II

[95. 不同的二叉搜索树 II](https://leetcode-cn.com/problems/unique-binary-search-trees-ii/)

// TODO:
