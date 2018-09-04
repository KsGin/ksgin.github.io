---
title: Prime-Number-of-Set-Bits-in-Binary-Representation
date: 2018-01-18 14:30:56
tags: LeetCode
---

#### 题目地址

[LeetCode#762 Prime Number of Set Bits in Binary Representation](https://leetcode.com/problems/prime-number-of-set-bits-in-binary-representation/description/)

#### 题目描述

&emsp;&emsp;Given two integers `L` and `R`, find the count of numbers in the range `[L, R]` (inclusive) having a prime number of set bits in their binary representation.

(Recall that the number of set bits an integer has is the number of `1`s present when written in binary. For example, `21` written in binary is `10101` which has 3 set bits. Also, 1 is not a prime.)

<!--more-->

**Example 1:**

```
Input: L = 6, R = 10
Output: 4
Explanation:
6 -> 110 (2 set bits, 2 is prime)
7 -> 111 (3 set bits, 3 is prime)
9 -> 1001 (2 set bits , 2 is prime)
10->1010 (2 set bits , 2 is prime)

```

**Example 2:**

```
Input: L = 10, R = 15
Output: 5
Explanation:
10 -> 1010 (2 set bits, 2 is prime)
11 -> 1011 (3 set bits, 3 is prime)
12 -> 1100 (2 set bits, 2 is prime)
13 -> 1101 (3 set bits, 3 is prime)
14 -> 1110 (3 set bits, 3 is prime)
15 -> 1111 (4 set bits, 4 is not prime)

```

**Note:**

1. `L, R` will be integers `L <= R` in the range `[1, 10^6]`.
2. `R - L` will be at most 10000.

#### 解题思路

&emsp;&emsp;这道题是找出在一定范围内的符合他的二进制里有素数个1的数字，唯一需要考虑的是如何求出一个数字的二进制里有多少个1这个问题，直接看代码吧。

#### 解题代码【.CPP】

```c++
class Solution {
    bool isPrimeNumberOfSetBitsInBinaryRepresentation(int n){
        int c = 0;
        while(n > 0){
            if((n & 1) == 1) c++;
            n = n >> 1;
        }
        if(c < 2) return false;
        for (int i = 2; i <  c; ++i) {
            if(c % i == 0) return false;
        }
        return true;
    }
public:
    int countPrimeSetBits(int L, int R) {
        int ret = 0;
        for (int i = L; i <= R; ++i) {
            ret += isPrimeNumberOfSetBitsInBinaryRepresentation(i);
        }
        return ret;
    }
};
```

