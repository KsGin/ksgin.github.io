---
title: Maximum-Product-of-Three-Numbers
date: 2017-12-12 08:26:56
tags: LeetCode
---

[LeetCode#628 Maximum Product of Three Numbers](https://leetcode.com/problems/maximum-product-of-three-numbers/description/)

Given an integer array, find three numbers whose product is maximum and output the maximum product.

**Example 1:**

```
Input: [1,2,3]
Output: 6

```

**Example 2:**

```
Input: [1,2,3,4]
Output: 24

```

**Note:**

1. The length of the given array will be in range [3,10^4] and all elements are in the range [-1000, 1000].
2. Multiplication of any three numbers in the input won't exceed the range of 32-bit signed integer.

<!--more-->

#### 解题思路

&emsp;&emsp;很麻瓜，刚开始看到这道题的时候，第一反应是排序然后直接乘倒数前三个，不过仔细看了题之后就放弃了，因为all elements are in the range [-1000, 1000]，及数组元素是可以为负的。暴力三重循环更是不可能，因为数据的数量是3到10^4个，三重循环肯定会超时，只能想想其他办法了。

&emsp;&emsp;这种题分析以分情况讨论为主，我们将数组分为以下几类：

​	&emsp;&emsp;1.全为正数，例如[1 , 2 , 3 , 4 , 5]     

​	&emsp;&emsp;2.全为负数，例如[-5,-4,-3,-2,-1]    

​	&emsp;&emsp;3.一半为负数，一半为正数	 [-5,-4,-3,2,1]

&emsp;&emsp;当数据全为正或者全为负的时候，最大乘积都是倒数前三个。而第三种情况又分为两种：

​	&emsp;&emsp;1.能够找到两个负数，使得这两个负数之积大于任意两个数之积 （排序后是前两个负数）

​	&emsp;&emsp;2.负数数量不足两个或者其积不够大

&emsp;&emsp;当发生第一种情况的时候，我们的选择是正数中最大的（排序后最后一个数字），和前两个负数。第二种情况则直接是可以跟全为正一样的处理方式。

&emsp;&emsp;这样归类下来，建立在已排序的基础上，我们就有两种乘的方式，一种是选前两个数字和最后一个数字，另一种是选倒数前三个数字。这样下来就很简单了 ，我们直接取这两种中比较大的值就可以了。代码很简单

#### 解题代码【.CPP】

```c++
class Solution {
public:
    int maximumProduct(vector<int> &nums) {
        sort(nums.begin(), nums.end());
        return max(nums[0] * nums[1] * nums[nums.size() - 1],
                   nums[nums.size() - 1] * nums[nums.size() - 2] * nums[nums.size() - 3]);
    }
};
```

