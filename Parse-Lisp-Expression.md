---
title: Parse-Lisp-Expression
date: 2018-03-10 18:00:43
tags: LeetCode
---

#### 题目地址

[LeetCode#736 Parse Lisp Expression](https://leetcode.com/problems/parse-lisp-expression/description/)

#### 题目描述

&emsp;&emsp;You are given a string `expression` representing a Lisp-like expression to return the integer value of.

&emsp;&emsp;The syntax for these expressions is given as follows.

<!--more-->

&emsp;&emsp;An expression is either an integer, a let-expression, an add-expression, a mult-expression, or an assigned variable. Expressions always evaluate to a single integer.

&emsp;&emsp;(An integer could be positive or negative.)

&emsp;&emsp;A let-expression takes the form `(let v1 e1 v2 e2 ... vn en expr)`, where `let` is always the string `"let"`, then there are 1 or more pairs of alternating variables and expressions, meaning that the first variable `v1` is assigned the value of the expression `e1`, the second variable `v2` is assigned the value of the expression `e2`, and so on **sequentially**; and then the value of this let-expression is the value of the expression `expr`.

&emsp;&emsp;An add-expression takes the form `(add e1 e2)` where `add` is always the string `"add"`, there are always two expressions `e1, e2`, and this expression evaluates to the addition of the evaluation of `e1` and the evaluation of `e2`.

&emsp;&emsp;A mult-expression takes the form `(mult e1 e2)` where `mult` is always the string `"mult"`, there are always two expressions `e1, e2`, and this expression evaluates to the multiplication of the evaluation of `e1` and the evaluation of `e2`.

&emsp;&emsp;For the purposes of this question, we will use a smaller subset of variable names. A variable starts with a lowercase letter, then zero or more lowercase letters or digits. Additionally for your convenience, the names "add", "let", or "mult" are protected and will never be used as variable names.

&emsp;&emsp;Finally, there is the concept of scope. When an expression of a variable name is evaluated, **within the context of that evaluation**, the innermost scope (in terms of parentheses) is checked first for the value of that variable, and then outer scopes are checked sequentially. It is guaranteed that every expression is legal. Please see the examples for more details on scope.

**Evaluation Examples:**

```
Input: (add 1 2)
Output: 3

Input: (mult 3 (add 2 3))
Output: 15

Input: (let x 2 (mult x 5))
Output: 10

Input: (let x 2 (mult x (let x 3 y 4 (add x y))))
Output: 14
Explanation: In the expression (add x y), when checking for the value of the variable x,
we check from the innermost scope to the outermost in the context of the variable we are trying to evaluate.
Since x = 3 is found first, the value of x is 3.

Input: (let x 3 x 2 x)
Output: 2
Explanation: Assignment in let statements is processed sequentially.

Input: (let x 1 y 2 x (add x y) (add x y))
Output: 5
Explanation: The first (add x y) evaluates as 3, and is assigned to x.
The second (add x y) evaluates as 3+2 = 5.

Input: (let x 2 (add (let x 3 (let x 4 x)) x))
Output: 6
Explanation: Even though (let x 4 x) has a deeper scope, it is outside the context
of the final x in the add-expression.  That final x will equal 2.

Input: (let a1 3 b2 (add a1 1) b2) 
Output 4
Explanation: Variable names can contain digits after the first character.
```

**Note:**

&emsp;&emsp;The given string `expression` is well formatted: There are no leading or trailing spaces, there is only a single space separating different components of the string, and no space between adjacent parentheses. The expression is guaranteed to be legal and evaluate to an integer.

&emsp;&emsp;The length of `expression` is at most 2000. (It is also non-empty, as that would not be a legal expression.)

&emsp;&emsp;The answer and all intermediate calculations of that answer are guaranteed to fit in a 32-bit integer.

#### 解题思路

&emsp;&emsp;这是一道类似于字符串计算器的题，只是 Lisp 的语法有些麻烦。善用递归循环就可以解决。

#### 解题代码【.CPP】

```c++
class Solution {
    int cal(string expression , unordered_map<string , int> ky){
        string exp = "";
        int ret = 0;
        if(expression[0] == '('){   //解析 ()
            ret = cal(expression.substr(1 , expression.size()-2) , ky);
            return ret;
        } else if(isdigit(expression[0]) || expression[0] == '-'){  //解析数字
            exp += expression[0];
            for (int i = 1; i < expression.size(); ++i) {
                if(!isdigit(expression[i])) break;
                else exp += expression[i];
            }
            ret = atoi(exp.c_str());
            return ret;
        } else if((expression.substr(0,3) == "add" && expression[3] == ' ')
                  || (expression.substr(0,4) == "mult" && expression[4] == ' ')){ //解析 and 和 mult
            int i , k = 0;
            int start = (expression[0] == 'a') ? 4 : 5;
            if(expression[start] == '('){
                k = 0;
                for (i = start + 1; i < expression.size(); ++i) {
                    if(expression[i] == ')' && k == 0) {
                        i++;
                        break;
                    }
                    else {
                        if(expression[i] == '(') k++;
                        if(expression[i] == ')') k--;
                        exp += expression[i];
                    }
                }
            } else {
                for (i = start; i < expression.size(); ++i) {
                    if(expression[i] == ' ') break;
                    else exp += expression[i];
                }
            }
            int par1 = cal(exp , ky);
            exp = string("");
            if(expression[i+1] == '('){
                k = 0;
                for (i = i+2; i < expression.size(); ++i) {
                    if(expression[i] == ')' && k == 0) {
                        i++;
                        break;
                    }
                    else {
                        if(expression[i] == '(') k++;
                        if(expression[i] == ')') k--;
                        exp += expression[i];
                    }
                }
            } else {
                for (i = i+1; i < expression.size(); ++i) {
                    if(expression[i] == ' ') break;
                    else exp += expression[i];
                }
            }
            int par2 = cal(exp , ky);
            if(expression[0] == 'a'){
                ret = par1 + par2;
            } else {
                ret = par1 * par2;
            }
            return ret;
        } else if(expression[0] == 'l'){  //解析let
            string key = "";
            int k = 0;
            for (int i = 4; i < expression.size(); ++i) {
                if(expression[i] == '('){
                    k = 0;
                    i++;
                    while ((expression[i] != ')' && i < expression.size()) || k != 0) { //key
                        if(expression[i] == '(') k++;
                        if(expression[i] == ')') k--;
                        key += expression[i];
                        i++;
                    }
                    i++;
                } else {
                    while (expression[i] != ' ' && i < expression.size()) { //key
                        key += expression[i];
                        i++;
                    }
                }
                if(i == expression.size()) break;
                ++i;
                if(expression[i] == '('){
                    i++;
                    k = 0;
                    exp = string("");
                    while (expression[i] != ')' || k != 0){
                        if(expression[i] == '(') k++;
                        if(expression[i] == ')') k--;
                        exp += expression[i];
                        i++;
                    }
                    i++;
                    ky[key] = cal(exp , ky);
                    key = string("");
                } else {
                    exp = string("");
                    while (expression[i] != ' '){
                        exp += expression[i];
                        i++;
                    }
                    ky[key] = cal(exp , ky);
                    key = string("");
                }
            }
            ret = cal(key , ky);
            return ret;
        } else {
            exp = string("");
            ret = ky[expression];
            return ret;
        }
    }

public:
    int evaluate(string expression) {
        int ret = cal(expression , unordered_map<string , int>());
        return ret;
    }
};
```

