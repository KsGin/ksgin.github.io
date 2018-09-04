---
title: Elimination-Game
date: 2018-03-17 12:40:33
tags: 
	- LeetCode
---

#### 题目地址

&emsp;&emsp;There is a list of sorted integers from 1 to *n*. Starting from left to right, remove the first number and every other number afterward until you reach the end of the list.

<!--more-->

&emsp;&emsp;Repeat the previous step again, but this time from right to left, remove the right most number and every other number from the remaining numbers.

&emsp;&emsp;We keep repeating the steps again, alternating left to right and right to left, until a single number remains.

&emsp;&emsp;Find the last number that remains starting with a list of length *n*.

**Example:**

```
Input:
n = 9,
1 2 3 4 5 6 7 8 9
2 4 6 8
2 6
6

Output:
6
```

#### 解题思路

&emsp;&emsp;题目要求按照给定的顺序多次删除数组内容：首先按照从左到右删除奇数索引上的数字，然后从右到左删除奇数索引上的数字。依次循环直到只剩一个数字然后返回这个数字。

&emsp;&emsp;以下这个霸气的解法不是我自己的（很惭愧智商不够），这个解法将题目分为镜像的子题目，使用递归求解。

> &emsp;&emsp;After first elimination, all the numbers left are **even** numbers.
> Divide by 2, we get a continuous new sequence from 1 to n / 2.
> For this sequence we start from right to left as the first elimination.
> Then the original result should be two times the mirroring result of lastRemaining(n / 2).

#### 解题代码【.CPP】

```c++
class Solution {
public:
    int lastRemaining(int n) {
        return n == 1 ? 1 : 2 * (n / 2 + 1 - lastRemaining(n / 2));
    }
};
```

