---
title: Missing-Number
date: 2017-12-22 12:58:27
tags: LeetCode
---

[LeetCode#268 Missing Number](https://leetcode.com/problems/missing-number/description/)

&emsp;&emsp;Given an array containing *n* distinct numbers taken from `0, 1, 2, ..., n`, find the one that is missing from the array.

**Example 1**

```
Input: [3,0,1]
Output: 2
```

**Example 2**

```
Input: [9,6,4,2,3,5,7,0,1]
Output: 8
```

<!--more-->

**Note**:
&emsp;&emsp;Your algorithm should run in linear runtime complexity. Could you implement it using only constant extra space complexity?

#### 解题思路

&emsp;&emsp;emmmm，很没有意思的一道题。题目要求是找到从0-N的数组中缺失的那一个数字（这些数字乱序排列且唯一），我没有试排序的方法而是用另一种思路。我们将0-N加起来，再将数组中的元素加起来，两者相减之后就是缺少的那个元素。

#### 解题代码【.CPP】

```c++
class Solution {
public:
    int missingNumber(vector<int> &nums) {
        int sum1 = 0, sum2 = 0;
        for (int i = 0; i < nums.size(); ++i) {
            sum1 += i;
            sum2 += nums[i];
        }
        sum1 += nums.size();
        return sum1 - sum2;
    }
};
```

