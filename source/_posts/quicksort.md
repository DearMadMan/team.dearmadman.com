title: 算法系列 —— 快速排序
date: 2016-10-20 10:47:17
tags: [php, javascript]
---

# 快速排序

> 快速排序（英语：Quicksort），又称划分交换排序（partition-exchange sort），一种排序算法，最早由东尼·霍尔提出。在平均状况下，排序 n 个项目要 Ο(n log n) 次比较。在最坏状况下则需要 Ο(n2) 次比较，但这种状况并不常见。事实上，快速排序通常明显比其他 Ο(n log n) 算法更快，因为它的内部循环（inner loop）可以在大部分的架构上很有效率地被实现出来。

> 快速排序使用分治法（Divide and conquer）策略来把一个序列（list）分为两个子序列（sub-lists）。

## 步骤

1. 从数列中选出一个元素作为基准（pivot)。
2. 对数列进行排序，所有大于基准的元素放在基准的右面，小于基准的元素放在基准的左边。
3. 递归的对左边和右边数列进行排序。

## 实现

### PHP

``` php
<?php

function quickSort($arr)
{
    if (count($arr) <= 1) {
        return $arr;
    }

    $pivot = $arr[0];
    $left = $right = [];

    for ($i = 1; $i < count($arr); $i++) {

        if ($arr[$i] > $pivot) {
            $right[] = $arr[$i];
        } else {
            $left[] = $arr[$i];
        }

    }

   return  array_merge(quickSort($left), [$pivot], quickSort($right));
}

$arr = [2, 33, 99, 2889, 192, 991, 91, 28, 1, 9 ,88 ,13];

print_r(quickSort($arr));
```

### JavaScript

``` js
'use strict'

function quickSort(arr)
{
    let length = arr.length
    if (length <= 1) return arr

    let left = []
    let right = []
    let pivot = arr[0]

    for (let i = 1; i < length; i++) {
        if (arr[i] > pivot) {
            right.push(arr[i])
        } else {
            left.push(arr[i])
        }
    }

    return Array.concat(quickSort(left), [pivot], quickSort(right));
}

let arr = [1, 8, 1234, 10, 88, 98, 871, 77, 1234, 998, 123, 22, 87, 78, 8]

console.log(quickSort(arr))
```