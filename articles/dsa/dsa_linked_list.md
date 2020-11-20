# DSA - 链表

## 简介

各种数据结构的底层实现都是数组和链表，数组和链表的根本区别在于它们使用内存的方式。

如果将计算机内存简单看成下图那样，每一个内存单元就是一个小格子。

可以看到，数组是需要连续空间的，而链表则不一定。

![](https://cdn.jsdelivr.net/gh/suukii/Articles/assets/dsa_linked_list_0.png)

由于数组是连续的内存空间，因此:

    - 可以按下标随机访问，访问的时间复杂度是 $O(1)$。
    - 插入或者删除操作很麻烦，由于需要移动插入/删除元素后面的所有元素，时间复杂度平均是 $O(N)$。只有在数组尾部进行插入/删除操作时间复杂度才是 $O(1)$。
    - 总的来说，数组是【查询友好，插入/删除不友好】。

而链表不要求连续的内存空间，各个节点可以随意散落在内存中，节点之间通过 `next` 指针连起来就行，所以：

    - 需要通过遍历来查找某一个节点，访问的时间复杂度是 $O(N)$。
    - 插入或者删除操作很方便，插入操作只需要在内存中随便找一个空格子存节点数据，然后把原链表尾部节点的 `next` 指针指向这个格子就好了。删除则只需要把待删除节点的空间回收，并把指向它的指针改为指向下一个节点就行。

## 基本操作

-   插入，给定节点插入位置的话，时间复杂度是 $O(1)$，否则还要遍历找到插入的节点位置，时间为 $O(N)$。
-   删除，$O(1)$，如果要先找到删除节点，时间也是 $O(N)$。
-   遍历，$O(N)$。

## 链表 vs 数组

都是线性的数据结构，只在细微的操作和使用场景上有差别。

## 考点

1. 指针的修改
2. 链表的拼接

### 指针的修改

典型的题目是 `反转链表`。

**反转链表模板**

反转任意一段链表：

JavaScript Code

```js
// 反转一段子链表，并返回反转后新的头尾节点
function reverse(head, tail, terminal) {
    let prev = null,
        cur = head;

    while (cur !== terminal) {
        // 保存下一个节点，防止丢失
        const next = cur.next;
        // 修改指针
        cur.next = prev;

        // 继续前移
        prev = cur;
        cur = next;
    }
    // 反转后的新的头尾节点返回出去
    return [tail, head];
}
```

### 链表的拼接

比如反转链表 II，以及合并有序链表等。

## 注意点

1. 环
2. 边界
3. 递归

## 技巧

1. 虚拟头
2. 快慢指针

## 题目

-   [x] [21. 合并两个有序链表](https://github.com/suukii/91-days-algorithm/blob/master/basic/linked-list/ext-merge-two-sorted-lists.md)
-   [ ] [82. 删除排序链表中的重复元素 II](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-list-ii/)
-   [x] [83. 删除排序链表中的重复元素](https://github.com/suukii/91-days-algorithm/blob/master/basic/linked-list/ext-remove-duplicates-from-sorted-list.md)
-   [ ] [86. 分隔链表](https://leetcode-cn.com/problems/partition-list/)
-   [ ] [92. 反转链表 II](https://leetcode-cn.com/problems/reverse-linked-list-ii/)
-   [ ] [138. 复制带随机指针的链表](https://leetcode-cn.com/problems/copy-list-with-random-pointer/)
-   [ ] [141. 环形链表](https://leetcode-cn.com/problems/linked-list-cycle/)
-   [ ] [142. 环形链表 II](https://leetcode-cn.com/problems/linked-list-cycle-ii/)
-   [ ] [143. 重排链表](https://leetcode-cn.com/problems/reorder-list/)
-   [ ] [148. 排序链表](https://leetcode-cn.com/problems/sort-list/)
-   [x] [206. 反转链表](https://github.com/suukii/91-days-algorithm/blob/master/basic/linked-list/ext-reverse-linked-list.md)
-   [ ] [234. 回文链表](https://leetcode-cn.com/problems/palindrome-linked-list/)
