---
title: 剑指 Offer 51. 数组中的逆序对
index_img: https://gitee.com/ylea/imagehost/raw/master/index_img/offer.jpg
banner_img: 'https://gitee.com/ylea/imagehost/raw/master/banner_img/44.png'
date: 2020-12-19 15:39:31
categories:
    - 算法题
    - 解答
    - 剑指Offer
tags:
    - 分治
    - 二叉搜索树
---

https://leetcode-cn.com/problems/shu-zu-zhong-de-ni-xu-dui-lcof/



在数组中的两个数字，如果前面一个数字大于后面的数字，则这两个数字组成一个逆序对。输入一个数组，求出这个数组中的逆序对的总数。

**示例 1:**

输入: `[7,5,6,4]`

输出: `5`

**限制：**

`0 <= 数组长度 <= 50000`



----



&emsp;&emsp;在数组中的两个数字，如果前面一个数字大于后面的数字，则这两个数字组成一个逆序对。输入一个数组，求出这个数组中的逆序对的总数。


# 解法1：二叉搜索树

&emsp;&emsp;在一组数字中，如 `[7, 5, 6, 4]`，对于每个数字，能否组成逆序对就得看前面是否有出现过比当前数字大的数字，而这个数字的个数就是与当前数字构成的逆序对的个数。如 `4`，那么前面有 `3` 个比 `4` 大的数，因此与当前数字 `4` 组成的逆序对个数就是 `3`。

&emsp;&emsp;那么我们就需要一种办法来快速找到比当前数字大的出现过的数字的个数，想到效率问题，我们就可能会联想到**二叉搜索树**：**左子树的节点值都比父节点值小，右子树的节点值都比当父节点值大**。

&emsp;&emsp;我们一边顺序遍历数字一边建树，对于遍历到的数字 `x`，在建树的过程中，若当前比较的节点值 `y` 比 `x` 大，那么其右子树上的节点值一定都大于 `x`，因此这个节点值 `y` 和其右子树上的所有节点值都能与 `x` 组成一个逆序对，所以逆序对的数量就是：**右子树节点的个数 + 1（节点 y）**，不过考虑到输入的数组中会出现**重复的数字**，因此每个节点中还需要使用一个变量来保存当前节点值的出现次数 `n`，所以最终逆序对的个数就是：**右子树节点的个数 + 当前节点的出现次数**。

&emsp;&emsp;综上，二叉搜索树的构造为：

```js
function Tree(val) {
	this.val = val;
    this.left = null;
    this.right = null;
    this.rightChildren = 0; // 右子树的节点个数
    this.n = 1;	// 当前节点值的出现次数
}
```

因此算法步骤就是：

1. 遍历输入数组，当前遍历到的值为 `x`，将 `x` 插入二叉搜索树中，二叉搜索树的根为 `nums[0]`。
2. 插入过程中会将 `x` 对树中的节点值进行比较，若当前节点的值大于 `x` ，那么将 `x` 插入其左子树中，并累计逆序对个数：`ans += node.rightChildren + node.n`。
3. 若当前节点值小于 `x`，那么将 `x` 插入其右子树中，且增加右子树节点个数：`node.rightChildren++`
4. 若当前节点值等于 `x`，那么将 `x` 的出现次数加1：`node.n++`；且当前节点的右子树都能与 `x` 组成逆序对，因此 `ans += node.rightChildren`。



用图来表示就是（下图中红色点代表当前插入的节点）：

<img src="https://gitee.com/ylea/imagehost/raw/master/img/image-20201219162125669.png" alt="image-20201219162125669" style="zoom:80%;" />


**想法很好但遇到了超时的 case** 😭

&emsp;&emsp;因为有 `case` 是单调递增、递减的，因此二叉树退化成链表，然后插入操作时间复杂度就变成了 `O(n^2)`。

**JS 代码：**

```js
function Tree(val, left, right) {
    this.val = val;
    this.rightChildren = 0;
    this.n = 1;
    this.left = left || null;
    this.right = right || null;
}

var reversePairs = function(nums) {
    if (nums.length === 0) {
        return 0;
    }
    let root = new Tree(nums[0]);
    let ans = 0;

    const insert = (root, val) => {
        if (val === root.val) {
            ans += root.rightChildren;
            root.n++;
        } else if (val < root.val) {
            ans += root.rightChildren + root.n;
            if (!root.left) {
                root.left = new Tree(val);
            } else {
                insert(root.left, val);
            }
        } else {
            root.rightChildren++;
            if (!root.right) {
                root.right = new Tree(val);
            } else {
                insert(root.right, val);
            }
        }
    };

    for (let i = 1; i < nums.length; i++) {
        insert(root, nums[i]);
    }

    return ans;
};
```

# 解法二：归并

&emsp;&emsp;归并的思想是将一个序列分为两份，如 `L = [8, 12, 16, 22, 100]`， `R = [9, 26, 55, 64, 91]`；

&emsp;&emsp;那么**左边的部分 `L` 中的值的索引一定是小于右边部分 `R` 中的值的索引的。** 因此**若 `L` 中有值比 `R` 中的值大**，那么一定是能组成逆序对的。

&emsp;&emsp;且基于归并排序，这两个序列都是有序的，因此就可以使用双指针来辅助计算逆序对个数 `ans`，我们按顺序比较 `L` 和 `R` 中的值，假设 `L` 的索引是从 `l` 到 `mid` ，`R` 的索引是从 `mid+1` 到 `r`，那么：

- 初始时设置两个指针，指针 `i` 指向 `L` 中的第一个元素 `8`，`j` 指向 `R` 中的第一个元素 `9`
- 若 `nums[i] > nums[j]`，那么 `nums[i]` 能与 `nums[mid+1] ~ nums[j]` 的所有数字构成逆序对（因为序列 `R` 是递增的，因此 `nums[j]` 之前的数一定会小于 `nums[j]`，也就是会小于 `nums[i]`），但此时我们先不计算逆序对个数（后面再一起计算），将 `j` 向后移一位：`j++`
- 若 `nums[i] <= nums[j]`，那么这两个数无法构成逆序对，此时计算 `nums[i]` 能与 `R` 中元素组成的逆序对个数：`ans += j - mid - 1`（`mid + 1 ~ j - 1` 的元素个数），指针 `i` 向后移一位：`i++`
- 当 `i > mid` 或 `j > r` 时，退出循环，此时：
  -  若 `i <= mid`，那么说明 `i` 及 `i` 之后的数都会**大于** `nums[mid + 1] ~ nums[r]`（因为循环中是当 `nums[i] > nums[j]` 时 `j` 才向后移一位），因此这些数字都能与 `R` 中的所有数字组成逆序对，所以能够组成的逆序对的个数是 `L 中剩余的元素个数 * R 中的元素个数`，也就是：`ans += (mid - i + 1) * (r - mid)`

JS 代码为：

```js
var reversePairs = function(nums) {
    let ans = 0;

    // 合并操作，将 [l, mid] 和 [mid+1, r] 位置的元素从小到大合并
    const merge = (l, mid, r) => {
        let res = [];
        let i = l, j = mid + 1;
        while (i <= mid && j <= r) {
            if (nums[i] <= nums[j]) {
                res.push(nums[i]);
                i++;
            } else {
                res.push(nums[j]);
                j++;
            }
        }

        while(i <= mid) {
            res.push(nums[i++]);
        }
        while (j <= r) {
            res.push(nums[j++]);
        }

        i = l;
        for (let n of res) {
            nums[i++] = n;
        }
    };

    const mergeSort = (l, r) => {
        if (l < r) {
            let mid = l + Math.floor((r - l) / 2);
            mergeSort(l, mid);
            mergeSort(mid + 1, r);

            let i = l, j = mid + 1;

            while (i <= mid && j <= r) {
                // 当 nums[i] <= nums[j] 时，计算 nums[i] 能与 nums[mid + 1 ~ j - 1] 组成逆序对
                if (nums[i] <= nums[j]) {
                    i++;
                    ans += j - mid - 1;
                } else {
                    j++;
                }
            }
            // 若出循环后 R 已经遍历完了，那么 L 中的剩余数字都能与 R 中的数字组成逆序对
            if (i <= mid) {
                ans += (mid + 1 - i) * (r - mid);
            }

            merge(l, mid, r);
        }
    };

    mergeSort(0, nums.length-1);

    return ans;
};
```

