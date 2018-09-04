---
title: Delete-And-Earn
date: 2018-02-04 19:37:09
tags: LeetCode
---

#### 题目地址

[LeetCode#740 Delete and Earn](https://leetcode.com/problems/delete-and-earn/description/)

#### 题目描述

Given an array `nums` of integers, you can perform operations on the array.

In each operation, you pick any `nums[i]` and delete it to earn `nums[i]` points. After, you must delete **every** element equal to `nums[i] - 1` or `nums[i] + 1`.

You start with 0 points. Return the maximum number of points you can earn by applying such operations.

<!--more-->

**Example 1:**

```
Input: nums = [3, 4, 2]
Output: 6
Explanation: 
Delete 4 to earn 4 points, consequently 3 is also deleted.
Then, delete 2 to earn 2 points. 6 total points are earned.
```

**Example 2:**

```
Input: nums = [2, 2, 3, 3, 3, 4]
Output: 9
Explanation: 
Delete 3 to earn 3 points, deleting both 2's and the 4.
Then, delete 3 again to earn 3 points, and 3 again to earn 3 points.
9 total points are earned.
```

**Note:**

The length of `nums` is at most `20000`.

Each element `nums[i]` is an integer in the range `[1, 10000]`.

#### 解题思路

&emsp;&emsp;很经典的DP题。与此类似的有[Best Time to Buy and Sell Stock with Transaction Fee](https://blog.ksgin.com/2018/01/22/best-time-to-buy-and-sell-stock-with-transaction-fee/)。

&emsp;&emsp;由于delete一个数字的话，和它相差1的数字将不给得分（即如果删除3，则只能得到3的分，2与4的不能得到），所以我们使用两个变量来维护（）take 与 skip），一个用来保存删除当前数字后的分数（take），一个用来保存不删除当前数字（skip）（即删除与它相差1的数字）。

&emsp;&emsp;由于题目要求delete一个数字的话必须删除与相差1的数字，所以我们的take的计算式为：

&emsp;&emsp;take = skip + nums.value * nums.count

&emsp;&emsp;而skip则是取**skip**与**更新前的take**中的较大值（skip保存的是不删除当前的值的最大分数，即上一步中的take与skip较大值）。

&emsp;&emsp;skip = max(take , skip)

#### 解题代码【.CPP】

```c++
class Solution {
public:
    int deleteAndEarn(vector<int>& nums) {
        vector<int> sums(10001, 0);
        int take = 0, skip = 0;
        for (int num : nums) sums[num] += num;
        for (int i = 0; i < 10001; ++i) {
            int takei = skip + sums[i];
            int skipi = max(skip, take);
            take = takei; skip = skipi;
        }
        return max(skip, take);
    }
};
```

