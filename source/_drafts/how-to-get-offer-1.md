---
title: 剑指 offer（01）：二维数组中的查找
permalink: how-to-get-offer-1
comments: true
date: 2019-03-06 21:56:47
updated: 2019-03-06 21:56:47
tags:
  - 二维数组
categories:
  - 剑指 offer
---

***问题描述：*** 在一个二维数组中，每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个函数，输入这样的一个二维数组和一个整数，判断数组中会否存在该整数。以如二维数组为例：

```shell
1  2  8  9
2  4  9  12
4  7  10 13
6  8  11 15
```

***解决思路：*** 将二维数组作为一个矩形来看待。选取右上角的数字与目标数字比较（比如选择 7），如果该数字大于目标数字，那么目标数字不会出现在该列，此时我们删除该列。同样的，如果该数字小于目标数字，那么目标数字也不会出现在该行，此时我们删除该行。如果该数字等于目标数字，那么就是我们需要找的结果。

***代码实现：***

```java
import sun.font.TrueTypeFont;

import java.util.Scanner;

public class Test {
    public static void main(String[] args) {
        int array[][] = {{1, 2, 8, 9},
                {2, 4, 9, 12},
                {4, 7, 10, 13},
                {6, 8, 11, 15}};

        int target = 7;

        findNumbers(array, target);
    }

    private static boolean findNumbers(int[][] numbers, int target) {
        int row = 0;
        int col = numbers[0].length - 1;

        while (row <= numbers.length - 1 && col >= 0) {
            if (numbers[row][col] == target) {
                System.out.println("找到了：row == " + row + "  col == " + col);
                return true;
            } else if (numbers[row][col] > target) {
                col = col - 1;
            } else {
                row = row + 1;
            }
        }
        return false;
    }

}
```
