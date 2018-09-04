---
title: Rabbits-in-Forest
date: 2018-03-04 12:00:11
tags: LeetCode
---

#### 题目地址

[LeetCode#781 Rabbits in Forest](https://leetcode.com/problems/rabbits-in-forest/description/)

#### 题目描述

&emsp;&emsp;In a forest, each rabbit has some color. Some subset of rabbits (possibly all of them) tell you how many other rabbits have the same color as them. Those `answers` are placed in an array.

&emsp;&emsp;Return the minimum number of rabbits that could be in the forest.

<!--more-->

```
Examples:
Input: answers = [1, 1, 2]
Output: 5
Explanation:
The two rabbits that answered "1" could both be the same color, say red.
The rabbit than answered "2" can't be red or the answers would be inconsistent.
Say the rabbit that answered "2" was blue.
Then there should be 2 other blue rabbits in the forest that didn't answer into the array.
The smallest possible number of rabbits in the forest is therefore 5: 3 that answered plus 2 that didn't.

Input: answers = [10, 10, 10]
Output: 11

Input: answers = []
Output: 0
```

**Note:**

1. `answers` will have length at most `1000`.
2. Each `answers[i]` will be an integer in the range `[0, 999]`.

#### 解题思路

&emsp;&emsp;这道题题目给定一个数组，数组里的每个数字代表着除了他自身之外还有多少个和他一样颜色的兔子，最终求森林里兔子的总量。

&emsp;&emsp;假定给定数组 [2] ，说明已经找到的有一个兔子，且这个兔子告诉我们除了他之外还有两只和他颜色一样的兔子，于是兔子总数为 3 。

&emsp;&emsp;假定给定数组[2，2]，说明已经找到有两只兔子，且这两只兔子告诉我们除了他本身之外还有两只颜色一样的，这两只兔子的回答一样说明这两只也是一样颜色的，所以没有找到的只有一只了。总数还是为3。

&emsp;&emsp;明白了计算方式，我们用 map 统计数组里每种颜色的兔子（回答的数字一样的为一种）的个数，最后用回答的数字加一（加上本身）减去统计的数字，就是没有统计到的这种颜色的兔子个数。

```c++
class Solution {
public:
    int numRabbits(vector<int>& answers) {
        unordered_map<int , int> rabbitsNumber;
        for (auto answer : answers) rabbitsNumber[answer]++;
        int ret = answers.size();
        for (auto &answer : rabbitsNumber){
            while (answer.second > answer.first + 1)
                answer.second -= answer.first + 1;
            ret += answer.first - answer.second + 1;
        }
        return ret;
    }
};
```

