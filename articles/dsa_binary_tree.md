# DSA - 二叉树

## 基本概念

**Terminologies**

-   Node: 至少有一个子节点的叫 internal node，没有子节点的叫 leaf node 或者 external node
-   Edge: 两个节点之间的连线
-   Root
-   Height of a Node: 从这个节点到最深的叶子节点一共有多少条边，也就是以这个节点为根节点的树的最大深度
-   Depth of a Node: 从根节点到这个节点一共有多少条边
-   Height of a Tree: 从根节点到最底下的叶子节点一共有多少条边
-   Degree of a Node: 这个节点的分支数
-   Forest: 没有关联的若干棵树

**树的应用**

-   二叉搜索树(BST)：用来查询一个集合内是否存在某个元素
-   堆(Heap)：用于堆排序
-   Trie：树的一种变体，用于路由器中存储路由信息
-   B 树、T 树：用于数据库
-   AST、CST：用于编译

## 二叉树的遍历

### 中序遍历

1. 首先遍历左子树的所有节点
2. 打印 `root`
3. 再遍历右子树的所有节点

> 在遍历顺序中，根节点是在**中间**的，所以叫做**中序遍历**。

```
inorder(root.left)
print(root)
inorder(root.right)
```

### 前序遍历

1. 首先打印 `root`
2. 然后遍历左子树的所有节点
3. 再遍历右子树的所有节点

> 在遍历顺序中，根节点是在**最前面**的，所以叫做**前序遍历**。

```
print(root)
preorder(root.left)
preorder(root.right)
```

### 后序遍历

1. 首先遍历左子树的所有节点
2. 然后遍历右子树的所有节点
3. 最后打印 `root`

> 在遍历顺序中，根节点是在**最后面**的，所以叫做**后序遍历**。

```
postorder(root.left)
postorder(root.right)
print(root)
```

### Morris Traversal

O(1) 空间实现遍历

**中序遍历思路**

使用一个指针，从 `root` 开始，

-   如果 `root.left` 为空，那就直接打印 `root`，然后指针移动到 `root.right` 并开始遍历右子树。
-   如果 `root.left` 不为空，那我们需要先去遍历左子树，遍历完之后再打印 `root`，然后再去遍历右子树。

那么问题来了，如果我们直接把指针移动到 `root.left`，那左子树遍历完之后我们就没办法回到 `root` 了，所以在这之前，我们需要把 `root` 保存起来，保存到哪里呢？

答案是 `左子树的最右子节点.right`。

为什么呢？我们来看下中序遍历的顺序： `left->root->right`，右子节点是最后遍历的，所以我们在遍历左子树时，结束的地方就是它最右边的子节点，这时也是我们要回到 `root` 的时候。所以，我们事先把 `root` 存在这里(也就是把左子树的最右子节点的 `right` 指针指向 `root`)，然后我们就可以放心地把指针移动到 `root.left` 开始遍历左子树了。

那这样岂不是会陷入死循环？所以还要加一个判断，如果，我们去找 `root` 左子树的最右子节点时，找到的却是 `root` 本身，说明左子树已经被遍历过了，这时我们需要断开 `左子树最右子节点` 与 `root` 之间的关联，然后再依次打印 `root` 和遍历右子树。

JavaScript Code

```js
/**
 * Definition for a binary tree node.
 * function TreeNode(val) {
 *     this.val = val;
 *     this.left = this.right = null;
 * }
 */
/**
 * @param {TreeNode} root
 * @return {number[]}
 */
var inorderTraversal = function (root) {
    if (!root) return []
    const res = []

    let cur = root
    while (cur) {
        if (!cur.left) {
            res.push(cur.val)
            cur = cur.right
        } else {
            // Make cur the rightmost child of its left subtree.
            let temp = cur.left
            while (temp.right && temp.right !== cur) {
                temp = temp.right
            }

            if (!temp.right) {
                temp.right = cur
                cur = cur.left
            } else {
                // If the left subtree has been traversed, restore the tree and start traverse the right subtree
                temp.right = null
                res.push(cur.val)
                cur = cur.right
            }
        }
    }
    return res
}
```

**Noted**

前/中/后序遍历结果都不能单独定义一颗树，`前序+后序` 也不行，但 `前序+中序` 或者 `后序+中序` 就足以定义一颗树了。

## 二叉树的分类

-   Full Binary Tree：每个父节点都有两个或者零个子节点，不同于国内满二叉树的定义，国内似乎没有这个概念。

-   Perfect Binary Tree：每个父节点都有两个子节点，每个叶子节点的深度都一样，也就是每一层的节点数都是最大的，相当于国内满二叉树的概念。

-   完全二叉树(Complete Binary Tree)：

    -   除了最后一层，其余的每一层都是满的，也就是叶子节点只会出现在最后一层和倒数第二层。
    -   最后一层要么是满的，要么在最右边缺少连续的叶子节点。

-   degenerate or pathological tree: 是指每个节点最多只有一个子节点，可以是左子节点或者右子节点。

-   skewed binary tree: 是 degenerate tree 的一种，不过所有子节点都是左子节点，或者都是右子节点。

-   balanced binary tee: 在平衡二叉树中，对于每个节点，它的左子树和右子树的高度差都是 1 或者 0。
