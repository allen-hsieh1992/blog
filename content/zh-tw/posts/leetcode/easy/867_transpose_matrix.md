+++
title = "演算法 [Java] LeetCode #867. Transpose Matrix"
author = "Allen Hsieh"
description = "Given a 2D integer array matrix, return the transpose of matrix."
featured = true
categories = ["LEETCODE"]
tags = ["LEETCODE"]
date = "2022-06-02"
aliases = ["leetcode_867_transpose_matrix"]
images = ["images/leetcode.jpeg"]
+++

## 題目
---
Given a 2D integer array matrix, return the transpose of matrix.

The transpose of a matrix is the matrix flipped over its main diagonal, switching the matrix's row and column indices.

Example 1:
```
Input: matrix = [[1,2,3],[4,5,6],[7,8,9]]
Output: [[1,4,7],[2,5,8],[3,6,9]]
```

Example 2:
```
Input: matrix = [[1,2,3],[4,5,6]]
Output: [[1,4],[2,5],[3,6]]
```

Constraints:
{{< katex >}}
  \begin{array}{l}
        m == matrix.length \\
        n == matrix[i].length \\
        1 <= m, n <= 1000 \\
        1 <= m * n <= 105 \\
        -109 <= matrix[i][j] <= 109
  \end{array}
{{< /katex >}}


## 我的思路
---
假設現在已經給了 N * M 的 Array ，那就建立一個新的 2D Array M * N。

Loop 一遍整個 Array，將 要返回的Array[m][n] 設定成 Input Array[n][m];


## 程式碼
---
```Java
class Solution {
    public int[][] transpose(int[][] matrix) {
        int[][] result = new int[matrix[0].length][matrix.length];
        
        for(int n = 0; n < matrix.length; n++) {
            for(int m = 0; m < matrix[0].length; m++) {
                result[m][n] = matrix[n][m];
            }
        }
        
        return result;
    }
}
```