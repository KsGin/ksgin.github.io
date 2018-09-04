---
title: My-Calendar-III
date: 2018-02-20 14:58:58
tags: LeetCode
---

#### 题目地址

[LeetCode#732 My Calendar III](https://leetcode.com/problems/my-calendar-iii/description/)

#### 题目描述

&emsp;&emsp;Implement a `MyCalendarThree` class to store your events. A new event can **always** be added.

<!--more-->

&emsp;&emsp;Your class will have one method, `book(int start, int end)`. Formally, this represents a booking on the half open interval `[start, end)`, the range of real numbers `x` such that `start <= x < end`.

&emsp;&emsp;A *K-booking* happens when **K** events have some non-empty intersection (ie., there is some time that is common to all K events.)

&emsp;&emsp;For each call to the method `MyCalendar.book`, return an integer `K` representing the largest integer such that there exists a `K`-booking in the calendar.

Your class will be called like this: 

```c++
MyCalendarThree cal = new MyCalendarThree();
MyCalendarThree.book(start, end)
```

**Example 1:**

```c++
MyCalendarThree();
MyCalendarThree.book(10, 20); // returns 1
MyCalendarThree.book(50, 60); // returns 1
MyCalendarThree.book(10, 40); // returns 2
MyCalendarThree.book(5, 15); // returns 3
MyCalendarThree.book(5, 10); // returns 3
MyCalendarThree.book(25, 55); // returns 3
Explanation: 
The first two events can be booked and are disjoint, so the maximum K-booking is a 1-booking.
The third event [10, 40) intersects the first event, and the maximum K-booking is a 2-booking.
The remaining events cause the maximum K-booking to be only a 3-booking.
Note that the last event locally causes a 2-booking, but the answer is still 3 because
eg. [10, 20), [10, 40), and [5, 15) are still triple booked.
```

**Note:**

&emsp;&emsp;The number of calls to `MyCalendarThree.book` per test case will be at most `400`.

&emsp;&emsp;In calls to `MyCalendarThree.book(start, end)`, `start` and `end` are integers in the range `[0, 10^9]`.

#### 解题思路

&emsp;&emsp;题目要求实现一个类，类的book方法每次传入一个事件的起始时间和结束时间，然后返回当前所存在的所有事件的最多相交次数，即i时间点上有N个事件发生，求max(N0,N1,N2….Ni)。

&emsp;&emsp;这个精妙的思路是参考网上的，我们构建一个map用来存储时间点和时间点对应的事件个数，每当遇到起始时间，则给起始时间的事件个数加1，给结束时间的事件个数减一，最终求和。

#### 解题代码【.CPP】

```c++
class MyCalendarThree {
private:
    map<int, int> freq;
public:
    MyCalendarThree() {}

    int book(int start, int end) {
        ++freq[start];
        --freq[end];
        int cnt = 0, mx = 0;
        for (auto f : freq) {
            cnt += f.second;
            mx = max(mx, cnt);
        }
        return mx;
    }
};

/**
 * Your MyCalendarThree object will be instantiated and called as such:
 * MyCalendarThree obj = new MyCalendarThree();
 * int param_1 = obj.book(start,end);
 */
```

