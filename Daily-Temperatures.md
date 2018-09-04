---
title: Daily-Temperatures
date: 2018-02-19 15:40:50
tags: LeetCode
---

#### 题目地址

[LeetCode#739 Daily Temperatures](https://leetcode.com/problems/daily-temperatures/description/)

#### 题目描述

&emsp;&emsp;Given a list of daily `temperatures`, produce a list that, for each day in the input, tells you how many days you would have to wait until a warmer temperature. If there is no future day for which this is possible, put `0` instead.

&emsp;&emsp;For example, given the list `temperatures = [73, 74, 75, 71, 69, 72, 76, 73]`, your output should be `[1, 1, 4, 2, 1, 1, 0, 0]`.

<!--more-->

**&emsp;&emsp;Note:** The length of `temperatures` will be in the range `[1, 30000]`. Each temperature will be an integer in the range `[30, 100]`.

#### 解题思路

&emsp;&emsp;题目的意思是给定一个数组，数组中的每一个数字代表那一天的温度，然后让我们返回多少天后温度升温，即返回多少天后数字变大。

&emsp;&emsp;这道题采用栈实现，当前温度小于栈顶温度时，我们将当前温度的索引入栈，当他大于栈顶索引所指的温度时，则为栈顶索引处赋值为当前索引去栈顶索引。

#### 解题代码【.CPP】

```c++
class Solution {
public:
    vector<int> dailyTemperatures(vector<int>& temperatures) {
        int n = temperatures.size();
        vector<int> res(n, 0);
        stack<int> st;
        for (int i = 0; i < temperatures.size(); ++i) {
            while (!st.empty() && temperatures[i] > temperatures[st.top()]) {
                auto t = st.top(); st.pop();
                res[t] = i - t;
            }
            st.push(i);
        }
        return res;
    }
};
```

