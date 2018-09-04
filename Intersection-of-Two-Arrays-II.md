---
title: Intersection-of-Two-Arrays-II
date: 2017-12-15 20:33:44
tags: LeetCode
---

[LeetCode#350 Intersection of Two Arrays II](https://leetcode.com/problems/intersection-of-two-arrays-ii/description/)

Given two arrays, write a function to compute their intersection.

**Example:**
Given *nums1* = `[1, 2, 2, 1]`, *nums2* = `[2, 2]`, return `[2, 2]`.

**Note:**

- Each element in the result should appear as many times as it shows in both arrays.
- The result can be in any order.

**Follow up:**

- What if the given array is already sorted? How would you optimize your algorithm?
- What if *nums1*'s size is small compared to *nums2*'s size? Which algorithm is better?
- What if elements of *nums2* are stored on disk, and the memory is limited such that you cannot load all elements into the memory at once?

#### 解题思路

&emsp;&emsp;之前有一道题也是求两个数组的交点[Intersection of Two Arrays](http://blog.ksgin.com/2017/12/15/Intersection-of-Two-Arrays/)，不过那个题比较简单，只需要循环然后使用std::find查找就可以了。这个题需要有重复的，我们可以将第一个数组的值和出现的次数存到map里，然后遍历第二个数组，如果在map里出现过就存入结果集里，并且将出现次数减一。

#### 解题代码【.CPP】

```c++
class Solution {
public:
    vector<int> intersect(vector<int>& nums1, vector<int>& nums2) {
        unordered_map<int , int> um;
        for (auto num : nums1){
            if(um.find(num) != um.end()) ++um[num];
            else um[num] = 1;
        }
        vector<int> ret(0);
        for (auto num: nums2) {
            if(um[num] > 0){
                ret.push_back(num);
                --um[num];
            }
        }
        return ret;
    }
};
```

