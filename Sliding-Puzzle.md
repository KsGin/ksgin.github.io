---
title: Sliding-Puzzle
date: 2018-01-29 23:27:39
tags: LeetCode
---

#### 题目地址

[LeetCode#773 Sliding Puzzle](https://leetcode.com/problems/sliding-puzzle/description/)

#### 题目描述

On a 2x3 `board`, there are 5 tiles represented by the integers 1 through 5, and an empty square represented by 0.

A move consists of choosing `0` and a 4-directionally adjacent number and swapping it.

The state of the board is *solved* if and only if the `board` is `[[1,2,3],[4,5,0]].`

Given a puzzle board, return the least number of moves required so that the state of the board is solved. If it is impossible for the state of the board to be solved, return -1.

<!--more-->

**Examples:**

```
Input: board = [[1,2,3],[4,0,5]]
Output: 1
Explanation: Swap the 0 and the 5 in one move.
```

```
Input: board = [[1,2,3],[5,4,0]]
Output: -1
Explanation: No number of moves will make the board solved.
```

```
Input: board = [[4,1,2],[5,0,3]]
Output: 5
Explanation: 5 is the smallest number of moves that solves the board.
An example path:
After move 0: [[4,1,2],[5,0,3]]
After move 1: [[4,1,2],[0,5,3]]
After move 2: [[0,1,2],[4,5,3]]
After move 3: [[1,0,2],[4,5,3]]
After move 4: [[1,2,0],[4,5,3]]
After move 5: [[1,2,3],[4,5,0]]
```

```
Input: board = [[3,2,4],[1,5,0]]
Output: 14
```

**Note:**

- `board` will be a 2 x 3 array as described above.
- `board[i][j]` will be a permutation of `[0, 1, 2, 3, 4, 5]`.

#### 解题思路

&emsp;&emsp;Sliding puzzle是一种棋盘游戏，这道题将其数字化。题目要求是给我们一个2*3的矩阵，矩阵内有无序排列的0-5六个数字，每一步交换只允许交换与0毗连的数字和0，让我们找出要将给定矩阵还原成123450所需要的最少步数。

&emsp;&emsp;由于仅仅是2*3棋盘，所可能存在的序列很有限，所以我们可以从最终状态（即123450）来逆推出所有能到达最终状态的状态并且统计出其到达最终状态的最少步数。在这里我们使用递归，每一次递归反映为一个状态，这个状态经过题目要求的交换后可能会产生多个新的状态，这是递归内容，而每一次递归的时候所产生的状态以及到达这一状态所需要的步数我们会进行保存，如果递归时发现当前状态已存在且存储的那个最小步数比当前的小则结束这一个分支（即结束条件是当前状态所能产生的状态都已经存在且存储的步数小于或者等于当前步数）。

&emsp;&emsp;这样计算后我们会得到所有的可到达最终状态的状态以及到达最终状态的最少步数集合，只需要判断输入状态是否在集合里，在返回对应步数，不在返回-1（说明输入状态无法到达最终状态）。

#### 解题代码【.CPP】

```c++
class Solution {
    void helper(unordered_map<string,int>& um , string state , int step){
        if(um.count(state) == 1) um[state] = min(step , um[state]);
        else um[state] = step;
        int idx = static_cast<int>(state.find_first_of('0'));
        if(idx < 3){
            swap(state[idx] , state[idx+3]);
            if(!um.count(state) || um[state] > step)
                helper(um , state , step+1);
            swap(state[idx] , state[idx+3]);
        }
        if(idx >= 3){
            swap(state[idx] , state[idx-3]);
            if(!um.count(state) || um[state] > step)
                helper(um , state , step+1);
            swap(state[idx] , state[idx-3]);
        }
        if(idx != 2 && idx != 5){
            swap(state[idx] , state[idx+1]);
            if(!um.count(state) || um[state] > step)
                helper(um , state , step+1);
            swap(state[idx] , state[idx+1]);
        }
        if(idx != 0 && idx != 3){
            swap(state[idx] , state[idx-1]);
            if(!um.count(state) || um[state] > step)
                helper(um , state , step+1);
            swap(state[idx] , state[idx-1]);
        }
    }

public:
    int slidingPuzzle(vector<vector<int>>& board) {
        string start = "";
        for (auto i : board){
            for (auto j : i) start.push_back(static_cast<char>(j + '0'));
        }
        string end = "123450";
        unordered_map<string , int> um;
        helper(um , end , 0);
        if(um.count(start) == 1) return um[start];
        return -1;
    }
};
```

