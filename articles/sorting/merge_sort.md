# 排序 - 归并排序

合并排序是一种**Divide and Conquer**(分治)的排序算法，把问题逐步拆分成子问题，直到子问题的规模小到可以解决，再自底向上合并子问题的解，最终得到原问题的解。

## 算法

**Divide**

假设 m 是数组 A 的中点，那么我们可以把数组分成 `A[0..m]` 和 `A[m+1..end]` 两个部分。

**Conquer**

继续将每个小数组都拆分成两个部分，直到不能再拆分，也就是数组中只剩下 1 个或 0 个数字的时候(base case)。

**Combine**

将这两个部分分别排好序后，再将它们合并成一个新的有序数组。将两个有序数组合并成一个有序数组，这个还是比较简单的。

## 复杂度

-   时间复杂度：$O(n*log n)$，n 为数组长度。不断地将数组一分为二，这个操作的时间复杂度是 $O(log n)$。每次递归中，合并两个有序数组的时间复杂度是 $O(n)$。
-   空间复杂度：$O(n)$，合并两个子数组时会需要新建一个临时数组，而这个临时数组的最大长度就是原数组的长度。

## 代码

JavaScript Code

```js
function mergeSort(arr, left, right) {
    // base case
    if (left >= right) return 0;

    const mid = Math.floor(left + (right - left) / 2);
    mergeSort(arr, left, mid);
    mergeSort(arr, mid + 1, right);

    merge(arr, left, mid, right);
}

function merge(arr, left, mid, right) {
    // 1. 使用一个临时数组，
    // 将有序的左右两部分 [left, mid] 和 [mid + 1, right] 先合并到临时数组 sortedPart 中
    const sortedPart = [];
    let p = left,
        q = mid + 1,
        s = 0;
    while (p <= mid && q <= right) {
        if (arr[p] <= arr[q]) {
            sortedPart[s++] = arr[p++];
        } else {
            sortedPart[s++] = arr[q++];
        }
    }
    while (p <= mid) {
        sortedPart[s++] = arr[p++];
    }
    while (q <= right) {
        sortedPart[s++] = arr[q++];
    }

    // 2. 将 sortedPart 里面的元素一一更新到原数组中的对应位置
    p = 0;
    q = left;
    while (p < sortedPart.length) {
        arr[q++] = sortedPart[p++];
    }
}
```

或者另一种写法，差不多。

```js
function mergeSort(arr) {
    // base case
    if (arr.length <= 1) return arr;

    // 将数组一分为二，分别排序(两个子数组是拷贝)
    const m = Math.floor(arr.length / 2);
    const sortedLeft = mergeSort(arr.slice(0, m));
    const sortedRight = mergeSort(arr.slice(m));

    // 拿到排好序的 sortedLeft 和 sortedRight后，直接在原数组 arr 上合并成一个有序数组
    let s = 0,
        p = 0,
        q = 0;
    while (p < sortedLeft.length && q < sortedRight.length) {
        if (sortedLeft[p] <= sortedRight[q]) {
            arr[s++] = sortedLeft[p++];
        } else {
            arr[s++] = sortedRight[q++];
        }
    }
    while (p < sortedLeft.length) {
        arr[s++] = sortedLeft[p++];
    }
    while (q < sortedRight.length) {
        arr[s++] = sortedRight[q++];
    }

    // 排好序后这个部分也需要返回
    // arr 本身也是一个更大的数组的一部分
    return arr;
}
```

## 应用

-   求逆序对问题。[【剑指 Offer 51. 数组中的逆序对】](https://leetcode-cn.com/problems/shu-zu-zhong-de-ni-xu-dui-lcof/)
