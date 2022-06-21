+++
title = "演算法 [Java] LeetCode #13. Roman to Integer"
author = "Allen Hsieh"
description = "Roman numerals are represented by seven different symbols: I, V, X, L, C, D and M."
featured = true
categories = ["LEETCODE"]
tags = ["LEETCODE"]
date = "2022-06-20"
aliases = ["leetcode_13_roman_to_integer"]
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

For example, 2 is written as II in Roman numeral, just two ones added together. 12 is written as XII, which is simply X + II. The number 27 is written as XXVII, which is XX + V + II.

Roman numerals are usually written largest to smallest from left to right. However, the numeral for four is not IIII. Instead, the number four is written as IV. Because the one is before the five we subtract it making four. The same principle applies to the number nine, which is written as IX. There are six instances where subtraction is used:

- I can be placed before V (5) and X (10) to make 4 and 9. 
- X can be placed before L (50) and C (100) to make 40 and 90. 
- C can be placed before D (500) and M (1000) to make 400 and 900.

Given a roman numeral, convert it to an integer.

Example 1: 
```bash
Input: s = "III"
Output: 3
Explanation: III = 3.
```

Example 2:
```bash
Input: s = "LVIII"
Output: 58
Explanation: L = 50, V= 5, III = 3.
```

Example 3:
```bash
Input: s = "MCMXCIV"
Output: 1994
Explanation: M = 1000, CM = 900, XC = 90 and IV = 4.
```

Constraints:
- 1 <= s.length <= 15
- s contains only the characters ('I', 'V', 'X', 'L', 'C', 'D', 'M').
- It is guaranteed that s is a valid roman numeral in the range [1, 3999].



## 我的思路
---
先用 HashMap 將所有的符號跟值做 Mapping 存起來。

Loop 一遍 Input 的 String
- 如果遇到 I、Ｘ 或 C 先用一個 Varaible previousChar 存起來並且不做任何事情跳過
- 如果不是就直接把當下代表的數字加上。

然後當發現 previousChar 不等於空時，代表著前一個字母為 I、Ｘ 或 C，這時候有一下兩種可能
- 那六種排列組合之一 例如 IV 、 IX、XL、XC、CD 或 CM。
- 除了以上六種以外，代表著 previousChar 就代表原本數字的意思 (ex I => 1)，將 previousChar 代表的值直接加回去。

最後由於我們之前一直把 I、Ｘ 或 C 跳過，直到下一個字母確認後，才把它加減上。
但這會造成整個字串最後一個字母沒有處理到，所以在 loop 完字串以後，要確認有沒有沒處理到的 I、Ｘ 或 C，然後再加回去。

## 我的思路
---
```Java
class Solution {
    HashMap<Character, Integer> romanMap = new HashMap<>() {{
        put('I', 1);
        put('V', 5);
        put('X', 10);
        put('L', 50);
        put('C', 100);
        put('D', 500);
        put('M', 1000);
    }};

    public int romanToInt(String s) {
        int sum = 0;
        char previousChar = Character.MIN_VALUE;

        for(char c : s.toCharArray()) {
            if (previousChar != Character.MIN_VALUE) {
                if ((previousChar == 'I' && (c == 'V' || c == 'X')) ||
                    (previousChar == 'X' && (c == 'L' || c == 'C')) ||
                    (previousChar == 'C' && (c == 'D' || c == 'M'))) {
                    sum += romanMap.get(c) - romanMap.get(previousChar);
                    previousChar = Character.MIN_VALUE;
                    continue;
                } else {
                    sum += romanMap.get(previousChar);    
                }               
            }

            if (c == 'I' || c == 'X' || c =='C') {
                previousChar = c;
            } else {
                sum += romanMap.get(c);
                previousChar = Character.MIN_VALUE;
            }
        }

        if (previousChar != Character.MIN_VALUE) {
            sum += romanMap.get(previousChar);
        }

        return sum;
    }   
}
```