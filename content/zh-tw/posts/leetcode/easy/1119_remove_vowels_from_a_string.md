+++
title = "演算法 [Java] LeetCode #1119. Remove Vowels from a String"
author = "Allen Hsieh"
description = "Given a string s, remove the vowels 'a', 'e', 'i', 'o', and 'u' from it, and return the new string."
featured = true
categories = ["JAVA", "ALGORITHM"]
tags = ["LEETCODE"]
date = "2022-06-18"
aliases = ["leetcode_1119_remove_vowels_from_a_string"]
images = ["images/leetcode.jpeg"]
+++

## 題目
---
Given a string s, remove the vowels 'a', 'e', 'i', 'o', and 'u' from it, and return the new string.

Example 1:
```bash
Input: s = "leetcodeisacommunityforcoders"
Output: "ltcdscmmntyfrcdrs"
```

Example 2:
```bash
Input: s = "aeiou"
Output: ""
```

Constraints:
- 1 <= s.length <= 1000
- s consists of only lowercase English letters.


## 我的思路
由於是將一個 String 中所有的母音都移除，所以我們直接建立一個新的 StringBuilder，將 Input loop 一遍，如果不是母音的話就加入到 StringBuilder 裡面，最後返回 String 

Note: 如果不知道為什麼要使用 StringBuilder，可以 google "String concat vs StringBuilde"。

## 程式碼
```Java
class Solution {
    public String removeVowels(String s) {
        StringBuilder result = new StringBuilder();
        
        for(char c : s.toCharArray()) {
            if (!isVowels(c)) {
                result.append(c);
            }
        }
        return result.toString();
    }
    
    public boolean isVowels(char c) {
        if (c == 'a' ||
            c == 'e' ||
            c == 'i' ||
            c == 'o' ||
            c == 'u') {
            return true;
        }
        return false;
    }
}
```