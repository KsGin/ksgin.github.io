---
title: Minimum-Number-of-Arrows-to-Burst-Balloons
date: 2017-12-21 08:30:26
tags: LeetCode
---

[LeetCode#452 Minimum Number of Arrows to Burst Balloons](https://leetcode.com/problems/minimum-number-of-arrows-to-burst-balloons/description/)

&emsp;&emsp;There are a number of spherical balloons spread in two-dimensional space. For each balloon, provided input is the start and end coordinates of the horizontal diameter. Since it's horizontal, y-coordinates don't matter and hence the x-coordinates of start and end of the diameter suffice. Start is always smaller than end. There will be at most 104 balloons.

<!--more-->

&emsp;&emsp;An arrow can be shot up exactly vertically from different points along the x-axis. A balloon with xstart and xend bursts by an arrow shot at x if xstart ≤ x ≤ xend. There is no limit to the number of arrows that can be shot. An arrow once shot keeps travelling up infinitely. The problem is to find the minimum number of arrows that must be shot to burst all balloons.

**Example:**

```c++
Input:
[[10,16], [2,8], [1,6], [7,12]]

Output:
2

Explanation:
   One way is to shoot one arrow for example at x = 6 (bursting the balloons [2,8] and [1,6]) and another arrow at x = 11 (bursting the other two balloons).
```

#### 解题思路

&emsp;&emsp;说实话这道题我刚开始没看懂，最后去求助Google才知道题目要做什么。这道题是要用最少的箭去射破所有的气球，而数组里每个元素就是气球的区间。就是说我们要找到最少的数字以满足所有的区间，拿题目上的例子。我们可以找到2-6中间的任意一个数字去满足`{[2,8],[1,6]}`这两个区间（因为他们有重合区），而用10-12中的任意一个数字求满足`{[10,16] , [7,12]}`两个区间。所以返回2。

&emsp;&emsp;要最小次数的减少箭的个数，首先我们将输入数组排序，然后从头开始遍历，在遍历过程中我们维护一个区间，就是当前的重合区间A1，当下一个气球与这个区间重合（假设重合区为A2），我们更新`A1 = A2`，直到下一个气球不与当前A1重合，这时候说明我们需要新的一枚箭，也就是结果`++ret`。

#### 解题代码【.CPP】

```c++
class Solution {
public:
    int findMinArrowShots(vector<pair<int, int>>& points) {
        sort(points.begin(),points.end(),
             [](pair<int , int> a , pair<int , int> b)
             { return a.first > b.first ;});
        int ret = 0;
        int start = 1 , end = 0;
        for (auto point : points){
            if(point.second < start || point.first > end || start > end){
                ret += 1;
                start = point.first;
                end = point.second;
            } else {
                start = max(start , point.first);
                end = min(end , point.second);
            }
        }
        return ret;
    }
};
```

