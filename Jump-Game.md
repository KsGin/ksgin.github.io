---
title: Jump-Game
date: 2018-04-04 13:33:05
tags: LeetCode
---

#### 题目地址

[LeetCode#55 跳跃游戏](https://leetcode-cn.com/problems/jump-game/description/)

#### 题目描述

&emsp;&emsp;给定一个非负整数数组，您最初位于数组的第一个索引处。数组中的每个元素表示您在该位置的最大跳跃长度。确定是否能够到达最后一个索引。

<!--more-->

**示例：**
A = `[2,3,1,1,4]`，返回 `true`。

A = `[3,2,1,0,4]`，返回 `false`。

#### 解题思路

&emsp;&emsp;要判断在 i 索引处能不能到达 j 的条件是 i + nums[i] >= j 。所以我们可以维护一个能到达最后一个索引且最小的位置 idx 。从后往前遍历，当 i 能够到达 idx 的时候更新 idx 。

#### 解题代码

```c++
class Solution {
public:
    bool canJump(vector<int>& nums) {
        int firstIdx = nums.size()-1;
        for (int i = nums.size()-2; i >= 0; --i) 
            if(i + nums[i] >= firstIdx) firstIdx = i;
        return firstIdx == 0;
    }
};
```

