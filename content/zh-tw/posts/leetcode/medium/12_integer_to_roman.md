+++
title = "演算法 [Java] LeetCode #12. Integer to Roman"
author = "Allen Hsieh"
description = "Roman numerals are represented by seven different symbols: I, V, X, L, C, D and M."
featured = true
categories = ["LEETCODE"]
tags = ["LEETCODE"]
date = "2022-06-21"
aliases = ["leetcode_12_integer_to_roman"]
images = ["images/leetcode.jpeg"]
+++

## 題目
---
Roman numerals are represented by seven different symbols: I, V, X, L, C, D and M.

|Symbol  |Value     |
|--------|----------|
|I       |1         |
|V       |5         |
|X       |10        |
|L       |50        |
|C       |100       |
|D       |500       |
|M       |1000      |

For example, 2 is written as II in Roman numeral, just two one's added together. 12 is written as XII, which is simply X + II. The number 27 is written as XXVII, which is XX + V + II.

Roman numerals are usually written largest to smallest from left to right. However, the numeral for four is not IIII. Instead, the number four is written as IV. Because the one is before the five we subtract it making four. The same principle applies to the number nine, which is written as IX. There are six instances where subtraction is used:

- I can be placed before V (5) and X (10) to make 4 and 9. 
- X can be placed before L (50) and C (100) to make 40 and 90. 
- C can be placed before D (500) and M (1000) to make 400 and 900.

Given an integer, convert it to a roman numeral.

Example 1:
```Bash
Input: num = 3
Output: "III"
Explanation: 3 is represented as 3 ones.
```

Example 2:
```Bash
Input: num = 58
Output: "LVIII"
Explanation: L = 50, V = 5, III = 3.
```

Example 3:
```Bash
Input: num = 1994
Output: "MCMXCIV"
Explanation: M = 1000, CM = 900, XC = 90 and IV = 4.
```

Constraints:
- 1 <= num <= 3999


## 我的思路
---
除了千位數，每一個位數先看有沒有 9 和 4 ，如果有就直接 Append 對應的羅馬數字。如果沒有就看有沒有大於五，有就將五的數字羅馬數字加上去，然後再把剩下一加上。

例如: 
- 80: 由於 80 的十位數為 8，那就先加上 50(L) ，然後 8 - 5 = 3 ，就代表要加上 3 個 10(X)，最後等於 LXXX 

## 我的思路
---
```Java
class Solution {
    public String intToRoman(int num) {
        StringBuilder result = new StringBuilder();
        appendThousandsRoman(num, result);
        appendHundredsRoman(num, result);
        appendTensRoman(num, result);
        appendDigitsRoman(num, result);
        return result.toString();
    }
    
    public void appendThousandsRoman(int num, StringBuilder result) {
        int nums_of_thousands_roman = num / 1000;
        for(int i = 0; i < nums_of_thousands_roman; i++) {
            result.append("M");
        }
    }
    
    public void appendHundredsRoman(int num, StringBuilder result) {
        int nums_of_hundreds_roman = (num % 1000) / 100;
        if (nums_of_hundreds_roman == 9) {
            result.append("CM");
            return;
        } else if (nums_of_hundreds_roman == 4) {
            result.append("CD");
            return;
        } else if (nums_of_hundreds_roman >= 5) {
            result.append("D");
            nums_of_hundreds_roman -= 5;
        }

        for(int i = 0; i < nums_of_hundreds_roman; i++) {
            result.append("C");
        }
    }

    public void appendTensRoman(int num, StringBuilder result) {
        int nums_of_hundreds_roman = (num % 100) / 10;
        if (nums_of_hundreds_roman == 9) {
            result.append("XC");
            return;
        } else if (nums_of_hundreds_roman == 4) {
            result.append("XL");
            return;
        } else if (nums_of_hundreds_roman >= 5) {
            result.append("L");
            nums_of_hundreds_roman -= 5;
        }

        for(int i = 0; i < nums_of_hundreds_roman; i++) {
            result.append("X");
        }
    }
    
    public void appendDigitsRoman(int num, StringBuilder result) {
        int nums_of_hundreds_roman = num % 10;
        if (nums_of_hundreds_roman == 9) {
            result.append("IX");
            return;
        } else if (nums_of_hundreds_roman == 4) {
            result.append("IV");
            return;
        } else if (nums_of_hundreds_roman >= 5) {
            result.append("V");
            nums_of_hundreds_roman -= 5;
        }

        for(int i = 0; i < nums_of_hundreds_roman; i++) {
            result.append("I");
        }
    }
}
```