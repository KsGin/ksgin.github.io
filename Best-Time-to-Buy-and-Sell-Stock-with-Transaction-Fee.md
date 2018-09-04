---
title: Best-Time-to-Buy-and-Sell-Stock-with-Transaction-Fee
date: 2018-01-22 21:41:43
tags: LeetCode
---

#### 题目地址

[LeetCode#714 Best Time to Buy and Sell Stock with Transaction Fee](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/description/)

#### 题目描述

Your are given an array of integers `prices`, for which the `i`-th element is the price of a given stock on day `i`; and a non-negative integer `fee` representing a transaction fee.

<!--more-->

You may complete as many transactions as you like, but you need to pay the transaction fee for each transaction. You may not buy more than 1 share of a stock at a time (ie. you must sell the stock share before you buy again.)

Return the maximum profit you can make.

**Example 1:**

```
Input: prices = [1, 3, 2, 8, 4, 9], fee = 2
Output: 8
Explanation: The maximum profit can be achieved by:
Buying at prices[0] = 1Selling at prices[3] = 8Buying at prices[4] = 4Selling at prices[5] = 9The total profit is ((8 - 1) - 2) + ((9 - 4) - 2) = 8.
```

**Note:**

`0 < prices.length <= 50000`.

`0 < prices[i] < 50000`.

`0 <= fee < 50000`.

#### 解题思路

&emsp;&emsp;sold保存第i天时卖出手里股票的利润最大值，hold保存第i天保留手里股票的利润最大值，sold计算方式为昨天保留在手里的利润加上今天卖出股票的利润减去服务费和昨天的利润中取较大值（即当今天卖出的利润小于或者等于服务费时不卖），hold计算方式为昨天卖出的总利润减去今天买进股票的利润和昨天不卖的利润中取较大值。

#### 解题代码【.CPP】

```c++
class Solution {
public:
    int maxProfit(vector<int>& prices, int fee) {
        int sold = 0, hold = -prices[0];
        for (int price : prices) {
            int t = sold;
            sold = max(sold, hold + price - fee);
            hold = max(hold, t - price);
        }
        return sold;
    }
};
```

