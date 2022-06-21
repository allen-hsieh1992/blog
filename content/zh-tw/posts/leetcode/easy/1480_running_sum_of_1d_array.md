+++
title = "演算法 [Java] LeetCode #1480. Running Sum of 1d Array"
author = "Allen Hsieh"
description = "Given an array nums. We define a running sum of an array as runningSum[i] = sum(nums[0]…nums[i])."
featured = true
categories = ["LEETCODE"]
tags = ["LEETCODE"]
date = "2022-06-01"
aliases = ["leetcode_1480_running_sum_of_1d_array"]
images = ["images/leetcode.jpeg"]
+++

## 題目
---
Given an array nums. We define a running sum of an array as runningSum[i] = sum(nums[0]…nums[i]).

Return the running sum of nums.

Example 1: 
```bash
Input: nums = [1,2,3,4]
Output: [1,3,6,10]
Explanation: Running sum is obtained as follows: [1, 1+2, 1+2+3, 1+2+3+4].
```
Example 2:
```bash
Input: nums = [1,1,1,1,1]
Output: [1,2,3,4,5]
Explanation: Running sum is obtained as follows: [1, 1+1, 1+1+1, 1+1+1+1, 1+1+1+1+1].
```

Example 3:
```
Input: nums = [3,1,2,10,1]
Output: [3,4,6,16,17]
```

Constraints:
{{< katex >}}
  \begin{array}{l}
    1 <= nums.length <= 1000 \\
    -10^6 <= nums[i] <= 10^6 
  \end{array}
{{< /katex >}}


## 我的思路
---
第 i 個 runningSum 等於 nums[0] + ... + nums[i-1] + nums[i];

而 nums[0] + nums[i-1] 等於 runningSum[i-1];

所以可以得到 runningSum[i] = runningSum[i-1] + nums[i];


## 程式碼
---
```Java
class Solution {
    public int[] runningSum(int[] nums) {
        int[] runningSum = new int[nums.length];
        runningSum[0] = nums[0];
        for(int i = 1; i < nums.length; i++) {
            runningSum[i] = runningSum[i-1] + nums[i];
        }
        return runningSum;
    }
}
```