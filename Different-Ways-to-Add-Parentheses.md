---
title: Different-Ways-to-Add-Parentheses
date: 2017-12-10 09:30:40
tags: LeetCode
---

&emsp;&emsp;[LeetCode#241 Different Ways to Add Parentheses](https://leetcode.com/problems/different-ways-to-add-parentheses/description/)

&emsp;&emsp;Given a string of numbers and operators, return all possible results from computing all the different possible ways to group numbers and operators. The valid operators are `+`, `-` and `*`.

Example 1

Input: `"2-1-1"`.

```
((2-1)-1) = 0
(2-(1-1)) = 2
```

Output: `[0, 2]`

Example 2

Input: `"2*3-4*5"`

```
(2*(3-(4*5))) = -34
((2*3)-(4*5)) = -14
((2*(3-4))*5) = -10
(2*((3-4)*5)) = -10
(((2*3)-4)*5) = 10
```

Output: `[-34, -14, -10, -10, 10]`

<!--more-->

#### 解题思路

​	这道题题目描述是给一个计算表达式例如 `2+3*5+6` , 要求在不同的位置添加括号使得其计算顺序不一样而最终获得不同的值。

​	很明显这种题一看就是使用递归，当我们得到一个表达式，将其从不同的位置分隔开（例如将`2+3*5+6`分割成`(2+3)*(5+6)`，之后递归这两个子表达式，得到这两个子表达式所能求出的值的集合，再将其进行交叉计算。）



#### 解题代码(.cpp)

```c++
// Runtime : 3ms
class Solution {
private:
    vector<string> analyseInput(string input) {
        vector<string> ret(0);
        string tmp="";
        for (int i = 0; i < input.size(); ++i) {
            if(input[i] <= '9' && input[i] >= '0'){
                tmp.push_back(input[i]);
            } else {
                ret.push_back(tmp);
                tmp = "";
                tmp.push_back(input[i]);
                ret.push_back(tmp);
                tmp = "";
            }
        }
        ret.push_back(tmp);
        return ret;
    }

    vector<int> recvCompute(vector<string> vp , int left , int right){
        if(right == left) return vector<int>{atoi(vp[left].c_str())};
        if(right == left + 2) {
            return vector<int>{compute(atoi(vp[left].c_str()) , vp[left+1] , atoi(vp[left+2].c_str()))};
        }
        vector<int> res;
        for (int i = left; i < right; i += 2) {
            vector<int> lres = recvCompute(vp , left , i);
            vector<int> rres = recvCompute(vp , i + 2 , right);
            for (auto l : lres){
                for(auto r : rres){
                    res.push_back(compute(l , vp[i+1] , r));
                }
            }
        }
        return res;
    }

    int compute(int af , string op , int bf){
        switch (op[0]){
            case '+':
                return af + bf;
            case '-':
                return af - bf;
            case '*':
                return af * bf;
            default:break;
        }
        return 0;
    }

public:
    vector<int> diffWaysToCompute(string input) {

        vector<string> cp = analyseInput(input);

        vector<int> res = recvCompute(cp , 0 , static_cast<int>(cp.size() - 1));

        sort(res.begin() , res.end());

        return res;
    }
};
```



