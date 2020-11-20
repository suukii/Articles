# 快速选择

## 简介

快速选择，quick select，是一种从无序列表中找到第 k 小元素的选择算法，也可以拓展到找到第 k 大元素。

它的原理是快速排序，不过跟快速排序不一样的是：

-   快速排序：

    -   选择 pivot 之后，
    -   将较小的数字划分到 pivot 的左边，
    -   较大的数字划分到右边，
    -   再对左右两边的数字分别递归排序。

-   快速选择：
    -   选择 pivot 之后，
    -   将较小的数字划分到 pivot 的左边，
    -   较大的数字划分到右边。
    -   从这里开始就是根快速排序不一样的地方。
    -   如果 pivot 刚好是第 k 个元素，直接返回，
    -   如果左边元素多于 k 个，就对左边的元素进行递归排序，右边的元素不用管了，
    -   反之则对右边的元素进行递归排序，左边的元素不用管，
    -   直到找到第 k 个元素。

## partition 算法

partition 算法指选择 pivot 后，将元素划分成左右两大阵营的算法，一般是原地修改数组，有以下两种方法：

-   Lomuto Partition
-   Hoare Partition

### Lomuto Partition

#### 算法

-   选择数组最后一个元素作为 pivot
-   用指针 j 遍历数组(不包括最后一个元素)，同时维护一个指针 i
-   当 `arr[j] <= pivot` 的时候，交换指针 i, j 的元素，然后 i += 1
-   最后将数组最后一个元素与 `arr[i]` 进行交换

#### 代码

JavaScript Code

```js
/**
 * 把 arr[r] 当成是 pivot
 * 把小于等于 pivot 的数字放到左边
 * 把大于 pivot 的数字放到右边
 * @param {number[]} arr
 * @param {number} l
 * @param {number} r
 */
function partition(arr, l, r) {
    const pivot = arr[r];
    let i = l;
    // 注意 arr[r] 是没有交换的
    for (let j = l; j < r; j++) {
        if (arr[j] <= pivot) {
            [arr[i], arr[j]] = [arr[j], arr[i]];
            i++;
        }
    }
    // 最后将 pivot 换到分界点
    [arr[i], arr[r]] = [arr[r], arr[i]];
    // 返回 pivot 的下标
    return i;
}
```

### Hoare Partition

#### 算法

-   选择数组第一个元素作为 pivot
-   用两个指针 l, r 一头一尾同时遍历数组
-   l 指针：当 `arr[l] <= pivot` 的时候，说明 `arr[l]` 不需要换位置，l 指针继续右移
-   r 指针：当 `arr[r] > pivot` 的时候，说明 `arr[r]` 不需要换位置，r 指针继续左移
-   如果 l, r 指针碰到一起了，就返回下标 r
-   不然，说明此时 `arr[l] > pivot` 并且 `arr[r] <= pivot`，需要将 l, r 两个位置的元素互换

#### 代码

JavaScript Code

```js
function partition(arr, l, r) {
    const pivot = arr[l];

    while (true) {
        // 小于等于 pivot 的元素留在左边不需要动
        while (arr[l] <= pivot) {
            l++;
        }

        // 大于 pivot 的元素留在右边不需要动
        while (arr[r] > pivot) {
            r--;
        }

        // 阵营分好了，返回分界点下标(分界点是属于左边阵营的)
        // 注意此时 pivot 还是在数组片段最左边的位置，并不在分界点
        if (l >= r) return r;

        // 左边是大于 pivot 的元素，
        // 右边是小于等于 pivot 的元素，
        // 将它们互换
        [arr[l], arr[r]] = [arr[r], arr[l]];
    }
}
```

### 注意点

1. Hoare Partition 结束后，返回的下标是左右两大阵营的分界点，但是 pivot 不一定在这个位置。所以递归排序左边元素的时候，这个下标也要包含进去，所以递归排序左边元素的区间是 `[l, p]`，而递归排序右边元素的区间则是 `[p+1, r]`。跟 Hoare Partition 不同，Lomuto Partition 的 pivot 元素刚好在分界点，所以递归排序的时候可以排除它。
2. 对于选择第 k 小的元素，Hoare Partition 需要选择最左边的元素作为 pivot；如果选择最右边元素作为 pivot 而 pivot 刚好是数组中最大的元素，则会陷入无限递归中。因为所有小于等于它的元素都在它左边，下一次递归排序左侧的时候实际上还是整个数组。如果 pivot 是随机选的，为了避免无限递归，可以将 pivot 和第一个元素互换位置。
3. Hoare Partition 比 Lomuto Partition 效率更高，但也不是稳定的排序算法。如果数组已经排序，则两种 Partition 算法都会导致快速选择的时间复杂度会降级到 $O(n^2)$。

## 快速选择算法

JavaScript Code

```js
/**
 * 在 [left, right] 区间内寻找第 k 小的元素
 * 如果 pivot 的下标刚好是 k - 1，那我们就找到了
 * 如果下标大于 k - 1，那就在 [left, pivotIndex - 1] 这段找第 k 小元素
 * 如果下标小于 k - 1，那就对 [pivotIndex + 1, right] 这段找第 k - pivotIndex + left - 1 小元素
 * @param {number[]} list
 * @param {number} k
 * @param {number} left
 * @param {number} right
 */
function quickSelect(list, k, left = 0, right = list.length - 1) {
    // 区间内只剩一个元素的话，返回这个元素
    if (left >= right) return list[left];

    // 注：这里用的 partition 算法是 Lomuto Partition
    // 如果用 Hoare Partition，要注意递归 quickSelect 的区间
    const pivotIndex = partition(list, left, right);

    // pivotIndex - left 刚好是 [left, right] 区间内第 k 小的元素下标
    if (pivotIndex - left === k - 1) return list[pivotIndex];
    // 在左侧继续寻找第 k 小的元素
    else if (pivotIndex - left > k - 1)
        return quickSelect(list, k, left, pivotIndex - 1);
    // 在右侧寻找第 k - pivotIndex + left - 1 小的元素
    // k - pivotIndex + left - 1 的来源：
    // 本区间左侧有 pivotIndex - left + 1 个元素，所以要在右侧找第 k - (pivotIndex - left + 1) 小的元素
    else
        return quickSelect(
            list,
            k - pivotIndex + left - 1,
            pivotIndex + 1,
            right,
        );
}
```

> 要改成寻找第 k 大的元素，修改下 partition 算法就行了。
