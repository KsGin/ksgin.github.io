---
title: My-Calendar-I
date: 2018-04-21 10:42:05
tags: LeetCode
---

#### 题目地址

[LeetCode#729 My Calendar I](https://leetcode.com/problems/my-calendar-i/description/)

#### 题目描述

&emsp;&emsp;Implement a `MyCalendar` class to store your events. A new event can be added if adding the event will not cause a double booking.

<!--more-->

&emsp;&emsp;Your class will have the method, `book(int start, int end)`. Formally, this represents a booking on the half open interval `[start, end)`, the range of real numbers `x` such that `start <= x < end`.

&emsp;&emsp;A *double booking* happens when two events have some non-empty intersection (ie., there is some time that is common to both events.)

&emsp;&emsp;For each call to the method `MyCalendar.book`, return `true` if the event can be added to the calendar successfully without causing a double booking. Otherwise, return `false` and do not add the event to the calendar.

Your class will be called like this: `MyCalendar cal = new MyCalendar();` ，

`MyCalendar.book(start, end)` 。

**Example 1:**

```
MyCalendar();
MyCalendar.book(10, 20); // returns true
MyCalendar.book(15, 25); // returns false
MyCalendar.book(20, 30); // returns true
Explanation: 
The first event can be booked.  The second can't because time 15 is already booked by another event.
The third event can be booked, as the first event takes every time less than 20, but not including 20.
```

**Note:**

&emsp;&emsp;The number of calls to `MyCalendar.book` per test case will be at most `1000`.

&emsp;&emsp;In calls to `MyCalendar.book(start, end)`, `start` and `end` are integers in the range `[0, 10^9]`.

#### 解题思路

&emsp;&emsp;在 book 方法中我们遍历当前区间数组进行插入，就可以了。

#### 解题代码

```c++
class MyCalendar {

    struct Interval {
        int start;
        int end;
        Interval(int s , int e) : start(s) , end(e){}
    };

    vector<Interval> intervals;

public:
    MyCalendar() {

    }

    bool book(int start, int end) {
        if (intervals.empty() || end <= intervals.front().start) {
            intervals.insert(intervals.begin() , Interval(start , end));
            return true;
        }
        for (auto it = intervals.begin(); it != intervals.end() - 1; ++it) {
            if (start >= (*it).end && end <= (*(it+1)).start) {
                intervals.insert(it+1 , Interval(start , end));
                return true;
            }
        }
        if (intervals.back().end <= start){
            intervals.insert(intervals.end() , Interval(start , end));
            return true;
        }
        return false;
    }
};

/**
 * Your MyCalendar object will be instantiated and called as such:
 * MyCalendar obj = new MyCalendar();
 * bool param_1 
```

