---
title: Task-Scheduler
date: 2018-03-20 08:26:45
tags: LeetCode
---

#### 题目地址

[LeetCode#621 Task Scheduler](https://leetcode.com/problems/task-scheduler/description/)

#### 题目描述

&emsp;&emsp;Given a char array representing tasks CPU need to do. It contains capital letters A to Z where different letters represent different tasks.Tasks could be done without original order. Each task could be done in one interval. For each interval, CPU could finish one task or just be idle.

&emsp;&emsp;However, there is a non-negative cooling interval **n** that means between two **same tasks**, there must be at least n intervals that CPU are doing different tasks or just be idle. 

<!--more-->

&emsp;&emsp;You need to return the **least** number of intervals the CPU will take to finish all the given tasks.

**Example 1:**

```
Input: tasks = ["A","A","A","B","B","B"], n = 2
Output: 8
Explanation: A -> B -> idle -> A -> B -> idle -> A -> B.
```

**Note:**

1. The number of tasks is in the range [1, 10000].
2. The integer n is in the range [0, 100].

#### 解题思路

&emsp;&emsp;刚开始会错了题意。。。。。浪费了好久的时间，最后还是看了别人的博客才明白过来。

&emsp;&emsp;这道题给定了你一个任务集合，一个代表着每两个相同任务之前必须间隔的时间。所以我们可以出现次数最多的任务来求。假设出现任务最多的是 A ，出现了 K 次，他们每个之间必须间隔 N 个，所以最基础的解就是 `(K - 1) * (N + 1)`（K 次则代表他们总共有 `K-1` 处间隔，每个间隔为 N 个，我们将他和 A 任务本身绑定在一起就是 `N + 1` 个一组，总共有 `(k-1)` 组和剩下的最后一个）。举个例子，最多的是 A ，出现 3 次，题目给定的是相同任务间隔 2 个。

```C++
 A··A··A					// (3-1) * (2+1) + 1
```

 &emsp;&emsp;但是如果出现最多的不止一个呢，比如 A , B 同时出现3次最多，那么应该如下：

```c++
 AB·AB·AB					// (3-1) * (2+1) + 2
```

&emsp;&emsp;可以看出来，我们最后的解应该如下（ k 为最多次数， n 为相同任务最小间隔，count 为出现最多次数的任务个数）：
```c++
result = (k-1) * (n+1) + count;
```

&emsp;&emsp;问题到此并没有结束，这个表达式仅仅是当我们其他的任务可以塞进这些间隔中来计算的。加入其他的任务有很多，除了这些之外还有很多剩余的任务没有完成。我们还应该加上他们，即
```c++
result =  (k-1) * (n+1) + count + abs( taskCount -  (k-1) *((n+1) + count); // 即
result	= max( (k-1) * (n+1) + count  ,  taskCount);
```

#### 解题代码【.CPP】

```c++
class Solution {
public:
    int leastInterval(vector<char>& tasks, int n) {
        vector<int> taskTimes(26);
        for (const auto& task : tasks) {
            ++taskTimes[task - 'A'];
        }
        sort(taskTimes.begin() , taskTimes.end() , [](int a , int b){return a > b;});
        int most = 1;
        for (auto i = 1 ; i < taskTimes.size() ; ++i){
            if (taskTimes[i] < taskTimes[i-1]) break;
            ++most;
        }
        return max((taskTimes[0]-1) * (n+1) + most , (int)tasks.size());
    }
};
```

