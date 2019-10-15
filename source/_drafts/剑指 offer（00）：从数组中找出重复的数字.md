---
title: 剑指 offer（01）：从数组中找出重复的数字
permalink: how-to-get-offer-0
comments: true
date: 2019-03-05 21:00:39
updated: 2019-03-05 21:00:39
tags:
  - 数组中找出重复的数字
categories:
  - 剑指 offer
---
***问题描述：*** 在一个长度为 `n` 的数组里的所有数字都在[0,n-1]的范围内。数组中某些数字是重复的，但不知道有几个数字重复了，也不知道每个数字重复了几次。请找出数组中任意一个重复的数字（以数组 {2, 3, 1, 0, 2, 5, 3} 为例）。
<!--more-->

***解决思路：*** 由于数组中的数字都在[0,n-1]之间，如果这个数组中没有重复的数字，那么当数组排序之后的数字 `i` 将出现在下标为 `i` 的位置。如果数组中有重复的数字，那么有些位置可能会存在多个数字，有些位置则可能没有数字。

***解决方案：*** 从头到尾扫描数组中的每一个数字。当扫描到小标为 `i` 的数字时，首先比较这个数字 `m` 是否等于 `i`。如果相等，则继续扫描下一个数字，如果不相等，则用它与下标为 `m` 的数字进行比较。如果它等于下标 `m` 对应的数字，那么就找到了一个重复的数字。如果它不等于下标 `m` 对应的数字，则交换 `i` 和 `m` 的位置。

***代码实现：***

```java
public class Test {
    public static void main(String[] args) {

        int[] numbers = {2, 3, 1, 0, 2, 5, 3};
        duplicate(numbers);
    }

    private static void duplicate(int[] numbers) {
        if (numbers == null || numbers.length == 0) {
            return;
        }
        int length = numbers.length;
        for (int i = 0; i < length; i++) {
            while (numbers[i] != i) {
                if (numbers[i] == numbers[numbers[i]]) {
                    System.out.println("重复的数字： " + numbers[i]);
                    break;
                } else {
                    int temp = numbers[i];
                    numbers[i] = numbers[temp];
                    numbers[temp] = temp;
                }
            }
        }
    }
}
```

上述的解决方案中，我们通过对数组进行重新排序的方式找出了重复的数字。但是如果要求我们不能修改输入的数组的话，我们应该怎么来解决呢？

***问题描述：*** 在一个长度为 `n+1` 的数组中，所有的数字都在 [1,n] 区间内，所以数组中至少存在一个数字是重复的，请在不修改数组的情况下找出任意一个重复的数字。以数组 {2，3，5，4，3，2，6，7} 为例。


***解决思路：*** 如果数组中没有重复的数字，那么 [1,n] 的数组的长度应该为 `n`。由于数组长度为 `n+1`，所以数组内包含了重复的数字。

***解决方案：*** 我们把 1 到 `n` 的数字从中间的数字 `m` 分为两部分，前面的一部分为 [1,m]，后面的一部分为 [m+1,n]。如果数组 [1,m] 的数字个数超过了 `m`，那么这一半的区间里一定包含重复的数字。否则，另一半 [m+1,n] 的数组中就一定包含了重复的数字。按照上述思路，把包含重复数字的一部分数组继续按照上述方式拆分。

***代码实现：***

```Java
public class Test {
    public static void main(String[] args) {

        int[] numbers = {2, 3, 5, 4, 3, 2, 6, 7};
        duplicate(numbers);
    }

    private static void duplicate(int[] numbers) {
        if (numbers == null || numbers.length == 0) {
            return;
        }
        int start = 1;
        int end = numbers.length;
        while (start <= end) {
            int mid = (start + end) / 2;
            int count = countRange(numbers, start, mid);
            if (start == end) {
                if (count > 1) {
                    System.out.println(start);
                    break;
                } else {
                    continue;
                }
            }
            if (count > (mid - start + 1)) {
                end = mid;
            } else {
                start = mid + 1;
            }
        }
    }


    private static int countRange(int[] numbers, int start, int end) {
        if (numbers == null)  //数组为空
            return 0;

        int count = 0;

        for (int i = 0; i < numbers.length; i++) {
            if (numbers[i] >= start && numbers[i] <= end) { //统计区间内的数在数组中出现的次数
                count++;
            }
        }
        return count;
    }
}
```
