---
layout: best
title: Time-to-Buy-and-Sell-Stock
date: 2018-03-14 12:12:03
tags: LeetCode
---

#### 题目地址

[LeetCode#121 Best Time to Buy and Sell Stock](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/description/)

#### 题目描述

&emsp;&emsp;Say you have an array for which the *i*th element is the price of a given stock on day *i*.

&emsp;&emsp;If you were only permitted to complete at most one transaction (ie, buy one and sell one share of the stock), design an algorithm to find the maximum profit.

<!--more-->

**Example 1:**

```
Input: [7, 1, 5, 3, 6, 4]
Output: 5

max. difference = 6-1 = 5 (not 7-1 = 6, as selling price needs to be larger than buying price)
```

**Example 2:**

```
Input: [7, 6, 4, 3, 1]
Output: 0

In this case, no transaction is done, i.e. max profit = 0.
```

#### 解题思路

&emsp;&emsp;我们定义两个索引 minIdx 与 maxIdx ，遍历给定数组，当值小于 minIdx 所对应的值时更新 minIdx 与 maxIdx （更新 maxIdx 以保证 max 值总是在 min 之后），当当前值大于 maxIdx 对应值的时候，更新 maxIdx 与 result。

#### 解题代码【.CPP】

```c++
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int res = 0;
        int minIdx = 0 , maxIdx = 0;
        for (int i = 1; i < prices.size(); ++i) {
            if(prices[i] <= prices[minIdx]) {
                minIdx = i;
                maxIdx = i;
            } else if(prices[i] >= prices[maxIdx]){
                maxIdx = i;
                res = max(res , prices[maxIdx] - prices[minIdx]);
            }
        }
        return res;
    }
};
```

