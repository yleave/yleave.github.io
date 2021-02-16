---
title: Morris算法
index_img: https://gitee.com/ylea/imagehost/raw/master/index_img/38.png
banner_img: https://gitee.com/ylea/imagehost/raw/master/banner_img/52.jpg
date: 2020-09-24 14:37:32
categories:
    - 算法题
    - 算法知识
tags:
    - 中序遍历
    - Morris
---

# Morris 算法介绍

&emsp;&emsp;Morris 算法是一种巧妙的方法可以在**线性时间**内，只占用**常数空间**来实现树的遍历。这种方法由 J. H. Morris 在 1979 年的论文「Traversing Binary Trees Simply and Cheaply」中首次提出，因此被称为 Morris 遍历。


&emsp;&emsp;Morris 算法一般用在解决中序遍历的问题，不过稍微修改一下，它也能用在前序遍历和[后序遍历](https://yleave.top/2020/09/30/%E7%AE%97%E6%B3%95%E9%A2%98/%E8%A7%A3%E7%AD%94/LeetCode/145-%E4%BA%8C%E5%8F%89%E6%A0%91%E7%9A%84%E5%90%8E%E5%BA%8F%E9%81%8D%E5%8E%86/#%E8%A7%A3%E6%B3%952morris-%E7%AE%97%E6%B3%95)上。


&emsp;&emsp;正常的中序遍历算法，无论是递归还是迭代，都需要使用到栈来保存节点，因此空间复杂度一般和树的节点数有关，而 Morris 算法是使用了树中子结点的空闲的指针来完成遍历过程。具体是通过底层节点指向`NULL`的空闲指针返回上层的某个节点，从而完成下层到上层的移动。


**Morris 遍历算法**整体步骤如下（假设当前遍历到的节点为 `x`）：

1. 如果 `x` 无左孩子，则访问 `x` 的右孩子，即 `x = x.right`
2. 如果 `x` 有左孩子，则**找到 `x` 左子树上最右的节点**（即左子树中序遍历的最后一个节点，是 `x` 在中序遍历中的前驱节点），记为 `predecessor` 。
   - 如果 `predecessor` 的右孩子为空，则将其右孩子指向 `x`，然后访问 `x` 的左孩子，即 `x = x.left`
   - 如果 `predecessor` 的右孩子不为空，则此时右孩子是指向 `x` 的，说明我们已经遍历完了左孩子；这时将 `predecessor` 的右孩子置空，然后访问 `x` 的右孩子，即 `x = x.right`
3. 重复上述步骤，直到访问完整棵树，即 `x = null` 时。

&emsp;&emsp;(没搞清楚的话可以自己模拟一个简单的二叉树试试)

&emsp;&emsp;感觉这个有点像是线索二叉树，不过这边的线索只是帮助遍历二叉树，用完一次就会清除。



**下面是 Morris 中序遍历算法的模板：**

**JS 版本：**

```js
var inOrder = function(root) {
    let predecessor;

    while(root) {
        if (root.left) {
            let predecessor = root.left;
            // 找到左子树的最右节点
            while(predecessor.right && predecessor.right !== root) {
                predecessor = predecessor.right;
            }

            // 若右孩子为空，表示第一次遍历到这个节点，将其右孩子设为root
            if (!predecessor.right) {
                predecessor.right = root;
                root = root.left;
            } else { // 否则， root 的左子树已经遍历完了，这时相当于中序遍历从左子树回到父节点
                // 此时是刚从左孩子节点返回之后，往右孩子遍历
                root = root.right;
                predecessor.right = null;
            }
        } else {
            // 节点此时可能是借助前面设置的空闲指针返回父节点，也可能是从父节点到右节点遍历的过程
            root = root.right;
        }
    }
};
```

**C++ 版本：**

```c++
TreeNode *cur = root, *pre = nullptr;
while (cur) {
    if (!cur->left) {
        // ...遍历 cur
        cur = cur->right;
        continue;
    }
    pre = cur->left;
    while (pre->right && pre->right != cur) {
        pre = pre->right;
    }
    if (!pre->right) {
        pre->right = cur;
        cur = cur->left;
    } else {
        pre->right = nullptr;
        // ...遍历 cur
        cur = cur->right;
    }
}
```

# Morris 算法实际应用

[LeetCode 501. 二叉搜索树中的众数](https://yleave.top/2020/09/24/%E7%AE%97%E6%B3%95%E9%A2%98/%E8%A7%A3%E7%AD%94/LeetCode/501-%E4%BA%8C%E5%8F%89%E6%90%9C%E7%B4%A2%E6%A0%91%E4%B8%AD%E7%9A%84%E4%BC%97%E6%95%B0/)

