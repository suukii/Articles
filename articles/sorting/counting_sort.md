# 排序 - 计数排序

## 概念

计数排序是先统计原数组中每个数字出现的次数，然后再根据这些统计构建结果数组。

统计时是使用一个辅助数组，将原数组中的数字作为辅助数组的下标，将数字出现的次数作为值。

## 应用

计数排序的优点是：

-   线性复杂度

缺点是：

-   数字的大小范围受限
-   额外的空间，如果数字范围比较大，空间消耗也会变大

所以适合使用计数排序的场景是：

-   数字都是整数，而且数字范围比较小
-   需要实现线性复杂度时

## 复杂度

-   时间复杂度：$O(n+k)$，n 是排序数组的长度，k 是排序数组中数字的范围，也就是最大的数字。一共有 4 层循环，找最大数字以及计算每个数字出现次数的时间复杂度是 $O(n)$；生成结果数组时，遍历辅助数组 `countArr` 的时间是 $O(k)$，在遍历 `countArr` 内部还有一个 `while` 循环，那个加起来也是 $O(n)$，所以总的时间复杂度是 $O(3n+k)$，也就是 $O(n+k)$。
-   空间复杂度：$O(n+k)$，n 是结果数组的长度，k 是辅助计数数组的长度。

## 代码

JavaScript Code

```js
function countingSort(array) {
    const maxNum = Math.max(...array);
    const countArr = Array(maxNum + 1).fill(0);

    array.forEach(n => countArr[n]++);

    const resArr = Array(array.length);
    let index = 0;
    countArr.forEach((count, num) => {
        while (count > 0) {
            resArr[index++] = num;
            count--;
        }
    });
    return resArr;
}
```
