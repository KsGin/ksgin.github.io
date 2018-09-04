---
title: Intersection-of-Two-Arrays
date: 2017-12-15 20:50:25
tags: LeetCode
---

[LeetCode#349 Intersection of Two Arrays](https://leetcode.com/problems/intersection-of-two-arrays/description/)

Given two arrays, write a function to compute their intersection.

**Example:**
Given *nums1* = `[1, 2, 2, 1]`, *nums2* = `[2, 2]`, return `[2]`.

**Note:**

- Each element in the result must be unique.
- The result can be in any order.

#### 解题思路

&emsp;&emsp;没什么说的，直接遍历查找，找到就存值，直接看代码。

#### 解题代码【.CPP】

```c++
class Solution {
public:
    std::vector<int> intersection(std::vector<int>& nums1, std::vector<int>& nums2) {
        std::set<int> rset;
        for (auto it = nums1.begin() ; it != nums1.end() ; ++it){
            if(std::find(nums2.begin() , nums2.end() , *it) != std::end(nums2)){
                rset.insert(*it);
            }
        }
        return std::vector<int>(rset.begin(),rset.end());
    }
};
```

