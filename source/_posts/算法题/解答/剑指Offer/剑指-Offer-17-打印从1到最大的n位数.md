---
title: 剑指 Offer 17. 打印从1到最大的n位数
index_img: https://cdn.jsdelivr.net/gh/yleave/imagehost/index_img/offer.jpg
banner_img: 'https://cdn.jsdelivr.net/gh/yleave/imagehost/banner_img/44.png'
date: 2020-10-28 21:46:53
categories:
    - 算法题
    - 解答
    - 剑指Offer
tags:
    - 回溯
    - 大数
---

https://leetcode-cn.com/problems/da-yin-cong-1dao-zui-da-de-nwei-shu-lcof/

难度：简单

---

&emsp;&emsp;输入数字 `n`，按顺序打印出从 `1` 到最大的 `n` 位十进制数。比如输入 `3`，则打印出 `1`、`2`、`3` 一直到最大的 `3` 位数 `999`。

**示例 1:**

&emsp;&emsp;输入: `n = 1`

&emsp;&emsp;输出: `[1,2,3,4,5,6,7,8,9]`


&emsp;&emsp;说明：

- 用返回一个整数列表来代替打印

- `n` 为正整数



## 解法1：回溯

&emsp;&emsp;这题正常来说思路是非常简单的，一个 `n` (`n`为正整数)位的数，最大值为 $10^n - 1$ ，且返回类型为 `int` ，因此不需要考虑超过 `MAX_INT` 的情况，只要一次遍历即可得出答案：

```c
int* printNumbers(int n, int* returnSize){
    int max = pow(10, n);
    int i;
    int* res = malloc(sizeof(int) * max);
    for (i = 1; i < max; i++) {
        res[i-1] = i;
    }

    *returnSize = max - 1;

    return res;
}
```

**不过真正面试写这题时，这样的写法肯定是不行的！**

&emsp;&emsp;我们必须考虑大数的情况，而大数除了使用 `long long` 或 JAVA 中的 `BigInteger` ，不过这样仍是有范围限制，所以一般会使用数组或字符串来进行表示一个很大的数。

&emsp;&emsp;因此，以数组或字符串的方式来考虑，从 `1` 到 $10^n-1$ 的这些数字，也可以看做数字是 `0 ~ 9` 的组合，而对于组合的获取，一般能使用回溯解决。

&emsp;&emsp;在递归函数中，我们以**从高位向低位的顺序选取组合数字**，如 `1024` ，我们先选取数字 `1` ，再选取数字 `0` ，再选取数字 `2` ，最后选取 `4` 这样。

&emsp;&emsp;不过，由于我们需要从 `0` 开始选取，因此结果中会有前缀 `0` ，如 `000`，`001`，.. `099` 这样，因此我们还需要想办法**去掉这些前缀 `0`**。

&emsp;&emsp;假设 `n = 3` 时，**先不考虑** `000` （全为 `0` ）的这种情况，观察数字组合，我们可以知道，最开始组合会有`n-1 = 2` 个前缀`0`，然后是 `1` 个，什么时候前缀 `0` 变少了呢？就是当数字组合从 `009` 变为 `010` 时，前缀 `0` 数量减少了，也就是**当组合中除了 `0` 以外的数字都是 `9` 之后，前缀 `0` 数量会减少** ，因此我们可以定义一个变量 `mine` 来计算**当前**组合中数字 `9`  的数量，假设我们前缀 `0` 的数量定义为 `start`，**当 `n - start = mine` 时，这时候组合中除了 `0` 其他数字就都是 `9` 了**（总共 `n` 位数，减去前缀 `0` 的数量 `start`，剩余个数和 `9` 的数量相同，因此剩余数字都为 `9`）。

&emsp;&emsp;最后，在获得所有数字组合后，去掉结果数组的第一位 `000` ，就得到了我们想要的结果了。



**按照这种思路，JS 代码如下：**

```js
var printNumbers = function(n) {
    let res = [];
    let temp = [];
    let nums = ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9'];
    let start = n - 1;
    let mine = 0;

    const dfs = index => {
        if (index === n) {
            // 获取第 start - 结尾的数字组合，即去除前缀 0 
            res.push(temp.slice(start).join(''));
            return;
        }
        for (let num of nums) {
            // 记录当前数字组合中 9 的个数
            if (num === '9') {
                mine++;
            }
            temp[index] = num;
            dfs(index+1);
            // 若全为 9 ，则之后的组合，前缀 0 的数量减少一个 
            if (n - start === mine) {
                start--;
            }
            // 最后恢复 mine 的数量
            if (num === '9') {
                mine--;
            }
        }
    };

    dfs(0);
	// 去除全为 0 的数字组合
    res.shift();

    return res;
};
```
