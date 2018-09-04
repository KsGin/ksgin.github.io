---
title: Employee-Free-Time
date: 2018-01-19 18:08:40
tags: LeetCode
---

#### 题目地址

[LeetCode#759 Employee Free Time](https://leetcode.com/problems/employee-free-time/description/)

#### 题目描述

We are given a list `schedule` of employees, which represents the working time for each employee.

Each employee has a list of non-overlapping `Intervals`, and these intervals are in sorted order.

Return the list of finite intervals representing **common, positive-length free time** for *all* employees, also in sorted order.

<!--more-->

**Example 1:**

```
Input: schedule = [[[1,2],[5,6]],[[1,3]],[[4,10]]]
Output: [[3,4]]
Explanation:
There are a total of three employees, and all common
free time intervals would be [-inf, 1], [3, 4], [10, inf].
We discard any intervals that contain inf as they aren't finite.
```

**Example 2:**

```
Input: schedule = [[[1,3],[6,7]],[[2,4]],[[2,5],[9,12]]]
Output: [[5,6],[7,9]]
```

(Even though we are representing `Intervals` in the form `[x, y]`, the objects inside are `Intervals`, not lists or arrays. For example, `schedule[0][0].start = 1, schedule[0][0].end = 2`, and `schedule[0][0][0]` is not defined.)

Also, we wouldn't include intervals like [5, 5] in our answer, as they have zero length.

**Note:**

1. `schedule` and `schedule[i]` are lists with lengths in range `[1, 50]`.
2. `0 <= schedule[i].start < schedule[i].end <= 10^8`.

#### 解题思路

&emsp;&emsp;这道题是给你多个员工的工作时间表，需要得到所有员工的共同休息时间，每个员工的时间表包括一个或多个[Start Time : End Time]这种组合，即代表在Start Time -> End Time这段时间是工作的。这道题可以先求出每个员工的休息时间，然后找出公共时间，也可以将所有的[Start Time : End Time]放到一个数组里，假想他们是一个员工的工作时间，这样问题变成了求一个员工的休息时间。

&emsp;&emsp;每个[Start Time : End Time]都是一个时间段，我们需要找出数组里所有时间段不包括的时间，即数组里如果有[3,4]和[5,6]两个时间段，则有[-inf,3],[4-5]和[6,inf]是除去他们之后的时间段，由于

> We discard any intervals that contain inf as they aren't finite.

所以只剩下[4,5]。而具体求法很简单，我们将所有的时间段根据开始时间从小到大排序，然后维持一个变量End作为当前时间段的结束时间。然后遍历我们刚才得到的时间段数组，如果其中时间段N的开始时间小于End，说明这个时间段N与我们当前的时间段是相交的，这时候我们更新End为当前End和N.End中较大的一个，如果N的开始时间大于End，说明这两个时间段不相交，那么[End , N.Start]就是我们要求的空闲时间段之一，此时更新End为N.End。

#### 解题代码【.CPP】

```c++
/**
 * Definition for an interval.
 * struct Interval {
 *     int start;
 *     int end;
 *     Interval() : start(0), end(0) {}
 *     Interval(int s, int e) : start(s), end(e) {}
 * };
 */
class Solution {
public:
    vector<Interval> employeeFreeTime(vector<vector<Interval>>& schedule) {
        vector<Interval> allInterval(0);
        for (auto em : schedule){
            for (auto e : em){
                allInterval.push_back(e);
            }
        }

        sort(allInterval.begin(),allInterval.end() ,
             [](Interval a , Interval b)
             {
                 return a.start < b.start;
             });

        vector<Interval> ret(0);
        if(allInterval.empty())
            return ret;
        int end = allInterval[0].end;
        for (auto ins : allInterval){
            if(ins.start <= end) end = max(end , ins.end);
            else {
                ret.emplace_back(end,ins.start);
                end = ins.end;
            }
        }
        return ret;
    }
};
```

