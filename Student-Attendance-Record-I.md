---
title: Student-Attendance-Record-I
date: 2017-12-24 10:48:42
tags: LeetCode
---

[LeetCode#551 Student Attendance Record I](https://leetcode.com/problems/student-attendance-record-i/description/)

&emsp;&emsp;You are given a string representing an attendance record for a student. The record only contains the following three characters:

1. **'A'** : Absent.
2. **'L'** : Late.
3. **'P'** : Present.

A student could be rewarded if his attendance record doesn't contain **more than one 'A' (absent)** or **more than two continuous 'L' (late)**.

<!--more-->

You need to return whether the student could be rewarded according to his attendance record.

**Example 1:**

```
Input: "PPALLP"
Output: True

```

**Example 2:**

```
Input: "PPALLL"
Output: False
```

#### 解题思路

&emsp;&emsp;Easy题，意思是给你一串字母A代表旷课，L代表迟到，P代表正常在场。如果旷课超过一次或者连续迟到超过两次就给挂掉。只需要一次遍历找出旷课次数和最大连续迟到次数，然后按情况返回就可以了。

#### 解题代码【.CPP】

```c++
class Solution {
public:
    bool checkRecord(string s) {
        int aCount = 0 , lCount = 0 , maxLCount = 0;
        for(auto r : s){
            if(r == 'L') ++lCount;
            else {
                if(lCount > maxLCount) maxLCount = lCount;
                lCount = 0;
                if(r == 'A') ++aCount;
            }
            if(aCount > 1 || maxLCount > 2) return false;
        }
        return lCount <= 2;
    }
};
```

