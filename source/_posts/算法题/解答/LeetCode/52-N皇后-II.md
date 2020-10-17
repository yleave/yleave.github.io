---
title: 52. N皇后 II
index_img: https://cdn.jsdelivr.net/gh/yleave/imagehost/index_img/LeetCode.jpg
banner_img: 'https://cdn.jsdelivr.net/gh/yleave/imagehost/banner_img/44.png'
date: 2020-10-17 12:05:48
categories:
    - 算法题
    - 解答
    - LeetCode
tags:
    - LeetCode
    - 回溯
---


https://leetcode-cn.com/problems/n-queens-ii/

难度：困难

----

&emsp;&emsp;*n* 皇后问题研究的是如何将 *n* 个皇后放置在 *n*×*n* 的棋盘上，并且使皇后彼此之间不能相互攻击。

<img src="https://cdn.jsdelivr.net/gh/yleave/imagehost/img/image-20201017095826142.png" alt="image-20201017095826142" style="zoom:80%;" />

&emsp;&emsp;上图为 8 皇后问题的一种解法。

&emsp;&emsp;给定一个整数 *n*，返回 *n* 皇后**不同的解决方案的数量**。



**示例:**

输入: 4

输出: 2

解释: 4 皇后问题存在如下两个不同的解法。

```
[
 [".Q..",  // 解法 1
  "...Q",
  "Q...",
  "..Q."],

 ["..Q.",  // 解法 2
  "Q...",
  "...Q",
  ".Q.."]
]
```



## 解法1：回溯

&emsp;&emsp;解法和 [N皇后1](http://yleave.top/2020/10/17/%E7%AE%97%E6%B3%95%E9%A2%98/%E8%A7%A3%E7%AD%94/LeetCode/51-N%E7%9A%87%E5%90%8E/) 是一样的，区别就是本题求的是解法个数， N皇后1 求的是所有的解法棋盘布局。

```js
var totalNQueens = function(n) {
    let dirs = [[-1, 0], [-1, -1], [-1, 1]];
    let res = 0;

    let board = new Array(n).fill(0).map(v => {
        return new Array(n).fill(0);
    });

    const helper = (row) => {
        if (row === n) {
            res++;
            return;
        }
        
        for (let col = 0; col < n; col++) {
            let flag;
            for (let dir of dirs) {
                let r = row, c = col;
                flag = true;
                while (r >= 0 && c >= 0 && c < n) {
                    if (board[r][c] === 1) {
                        flag = false;
                        break;
                    }
                    r += dir[0], c += dir[1];
                }
                if (!flag) {
                    break;
                }
            }

            if (flag) {
                board[row][col] = 1;
                helper(row + 1);
                board[row][col] = 0;
            }
        }
    };

    helper(0);

    return res;
};
```

## 解法2：空间优化

&emsp;&emsp;对于解法1，使用了 `n * n` 的棋盘来帮助判断某个位置是否满足安置皇后的要求，但是，对于每种 N皇后的安置策略，我们仅使用了棋盘中的一部分来帮助判断（即每个位置的两条斜线和直线），因此，我们可以缩小空间，仅使用三个列表来分别表示直行、和两条斜行上是否有皇后。

&emsp;&emsp;例如，对于直行，若第三列上有皇后，则 `cols[3] = 1`，否则，`cols[3] = 0`

&emsp;&emsp;直行判断是否有皇后很简单，只要使用列下标即可。

<img src="https://cdn.jsdelivr.net/gh/yleave/imagehost/img/image-20201017104019691.png" alt="image-20201017104019691" style="zoom:67%;" />

&emsp;&emsp;对于其中一个方向的斜线，棋盘中的每条这个方向的斜线都能使用一个下标来表示，即 **行下标 - 列下标**：

<img src="https://cdn.jsdelivr.net/gh/yleave/imagehost/img/image-20201017110155747.png" alt="image-20201017110155747" style="zoom:67%;" />

&emsp;&emsp;另一个方向的斜线能使用 **行下标 + 列下标** 来表示：

<img src="https://cdn.jsdelivr.net/gh/yleave/imagehost/img/image-20201017110328469.png" alt="image-20201017110328469" style="zoom:67%;" />

&emsp;&emsp;因此，对于棋盘中的一个位置，只需借助这三个列表来判断位置是否合法即可（三个列表中都不含当前位置）

```js
var totalNQueens = function(n) {
    let res = 0;
    let cols = [], diagonal1 = [], diagonal2 = [];

    const helper = (row) => {
        if (row === n) {
            res++;
            return;
        }

        for (let i = 0; i < n; i++) {
            if (cols[i] === 1) {
                continue;
            }

            let d1 = row - i;
            if (diagonal1[d1] === 1) {
                continue;
            }

            let d2 = row + i;
            if (diagonal2[d2] === 1) {
                continue;
            }

            cols[i] = 1;
            diagonal1[d1] = 1;
            diagonal2[d2] = 1;
            helper(row+1);
            cols[i] = 0;
            diagonal1[d1] = 0;
            diagonal2[d2] = 0;
        }
    };

    helper(0);

    return res;
};
```

