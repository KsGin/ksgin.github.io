---
title: Insert-Interval
date: 2018-04-08 11:03:18
tags: LeetCode
---

#### 题目地址

[LeetCode#57 插入区间](https://leetcode-cn.com/problems/insert-interval/description/)

#### 题目描述

&emsp;&emsp;给出一个无重叠的按照区间起始端点排序的区间列表。

&emsp;&emsp;在列表中插入一个新的区间，你要确保列表中的区间仍然有序且不重叠（如果有必要的话，可以合并区间）。

<!--more-->

**示例 1:**
给定区间 `[1,3],[6,9]`，插入并合并 `[2,5]` 得到 `[1,5],[6,9]`.

**示例 2:**
给定区间 `[1,2],[3,5],[6,7],[8,10],[12,16]`，插入并合并 `[4,9]` 得到 `[1,2],[3,10],[12,16]`.

这是因为新的区间 `[4,9]` 与 `[3,5],[6,7],[8,10]` 重叠。

#### 解题思路

&emsp;&emsp;题目要求插入一个区间并且合并因为插入导致所有的重叠区间。由于题目给定的区间数组本身是不重叠的，所以计算起来比较简单，我们通过遍历找到第一个和最后一个与要插入区间重叠的区间，然后将他们合并。

#### 解题代码

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
    vector<Interval> insert(vector<Interval> &intervals, Interval newInterval) {
        // 原数组为空
        if (intervals.empty()) {
            return vector<Interval>{newInterval};
        }
        // 新的区间在最前
        if (newInterval.end < intervals.front().start) {
            vector<Interval> ret = intervals;
            ret.insert(ret.begin(), newInterval);
            return ret;
        }
        // 在最后
        if (newInterval.start > intervals.back().end) {
            vector<Interval> ret = intervals;
            ret.push_back(newInterval);
            return ret;
        }
        int start = 0, end = 0;
        vector<Interval> ret(0);
        bool f = false;
        for (int i = 0; i < intervals.size(); ++i) {
            if (!f && intervals[i].end >= newInterval.start) {
                start = min(intervals[i].start, newInterval.start);
                for (int j = i; j < intervals.size(); ++j) {
                    if (intervals[j].start > newInterval.end) break;
                    ++i;
                }
                end = max(intervals[--i].end, newInterval.end);
                ret.emplace_back(start, end);
                f = true;
            } else {
                ret.push_back(intervals[i]);
            }
        }
        return ret;
    }
};
```

