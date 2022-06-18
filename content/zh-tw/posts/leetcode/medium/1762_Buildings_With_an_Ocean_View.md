+++
title = "演算法 [Java] LeetCode #1762. Buildings With an Ocean View"
author = "Allen Hsieh"
description = "There are n buildings in a line. You are given an integer array heights of size n that represents the heights of the buildings in the line."
featured = true
categories = ["JAVA", "ALGORITHM"]
tags = ["LEETCODE"]
date = "2022-06-17"
aliases = ["leetcode_1762_buildings_with_an_ocean_view"]
images = ["images/leetcode.jpeg"]
+++

## 題目
---
There are n buildings in a line. You are given an integer array heights of size n that represents the heights of the buildings in the line.

The ocean is to the right of the buildings. A building has an ocean view if the building can see the ocean without obstructions. Formally, a building has an ocean view if all the buildings to its right have a smaller height.

Return a list of indices (0-indexed) of buildings that have an ocean view, sorted in increasing order.

Example 1:
```bash
Input: heights = [4,2,3,1]
Output: [0,2,3]
Explanation: Building 1 (0-indexed) does not have an ocean view because building 2 is taller.
```

Example 2:
```bash
Input: heights = [4,3,2,1]
Output: [0,1,2,3]
Explanation: All the buildings have an ocean view.
```

Example 3:
```bash
Input: heights = [1,3,2,4]
Output: [3]
Explanation: Only building 3 has an ocean view.
```

Constraints:
{{< katex >}}
  \begin{array}{l}
    1 <= heights.length <= 10^5 \\
    1 <= heights[i] <= 10^9 
  \end{array}
{{< /katex >}}


## 我的思路
---
我們現在要看有多少建築物可以看到右邊的海洋，所以當我在第 i 個建築物時，i 建築物右邊的全部建築物都需要比 i 建築物矮。

所以我會從最右開始往左看，用一個變數去記錄當下最高建築物的高度，如果當下的建築物沒有比記錄的建築物高，也就代表可以看到海洋。

由於回傳的結果必須是由左到右的，所以這邊我會先用 Stack 將可看見海洋的建築物存下來，最後在將 Stack 裡面的東西轉換回 Array 返回。


## 程式碼
---
```Java
class Solution {
    public int[] findBuildings(int[] heights) {
        // 初始化最右邊的建築物
        int heightest = heights[heights.length - 1];
        Stack<Integer> stack = new Stack();
        // 最右邊的放子一定看的到海
        stack.push(heights.length - 1);
        
        for(int i = heights.length - 1; i >= 0; i--) {
            if (heights[i] > heightest) {
                stack.push(i);
                heightest = heights[i];
            }
        }
        
        int size = stack.size();
        int[] result = new int[size];
        for(int i = 0; i < size; i++) {
            result[i] = stack.pop();
        }
        return result;
    }
}
```

Time complexity: O(N)

Space complexity: O(N)
