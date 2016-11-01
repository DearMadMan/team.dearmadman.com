title: 算法系列 —— 冒泡排序
date: 2016-10-20 09:43:39
tags: [php, javascript]
---

# 冒泡排序

> 冒泡排序（英语：Bubble Sort，台湾另外一种译名为：泡沫排序）是一种简单的排序算法。它重复地走访过要排序的数列，一次比较两个元素，如果他们的顺序错误就把他们交换过来。走访数列的工作是重复地进行直到没有再需要交换，也就是说该数列已经排序完成。这个算法的名字由来是因为越小的元素会经由交换慢慢“浮”到数列的顶端。

> 冒泡排序对 n 个项目需要 O(n^2)的比较次数，且可以原地排序。尽管这个算法是最简单了解和实现的排序算法之一，但它对于少数元素之外的数列排序是很没有效率的。

## 步骤

1. 比较两个相邻的元素。如果第一个比第二个大，就交换他们两个。
2. 对每一对相邻元素做同样的操作，从开始的一对到最后的一对。这步完成后，最后的元素将是最大的元素。
3. 针对以上元素重复以上步骤，排除最后一个元素。
4. 重复以上步骤，直至没有可以比较的对数。


## 实现 

### PHP

``` php
<?php

function bubble ($arr)
{
    for ($i = 0, $l = count($arr) - 1; $i < $l; $i++) {

        for ($j = 0; $j < $l - $i; $j++) {
            if ($arr[$j] > $arr[$j+1]) {
                $temp = $arr[$j];
                $arr[$j] = $arr[$j+1];
                $arr[$j+1] = $temp;
            }
        }

    }

    return $arr;
}

$arr = [2, 3 ,5 ,8, 1, 2, 9, 11, 8, 113, 89, 1, 0];

var_dump(bubble($arr));

// wittin PHP, you can use following:
// sort($arr)
```

### JavaScript

``` js
function bubble(arr)
{
  for (var i = 0, l = arr.length - 1; i < l; i++) {
    
    for (var j = 0; j < l - i; j ++ ) {
      if (arr[j] > arr[j+1]) {
        var temp = arr[j]
        arr[j] = arr[j+1]
        arr[j+1] = temp
      }
    }

  }
  return arr
}

// within JavaScript, you can use this:
// arr.sort((a, b) => a - b)
```