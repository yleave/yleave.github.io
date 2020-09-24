---
title: LeetCode 968.监控二叉树
index_img: https://cdn.jsdelivr.net/gh/yleave/imagehost/index_img/LeetCode.jpg
banner_img: 'https://cdn.jsdelivr.net/gh/yleave/imagehost/banner_img/45.jpg'
date: 2020-09-23 00:04:54
categories:
tags:
---

https://leetcode-cn.com/problems/binary-tree-cameras/

难度：困难

---

&emsp;&emsp;给定一个二叉树，我们在树的节点上安装摄像头。

&emsp;&emsp;节点上的每个摄影头都可以监视**其父对象、自身及其直接子对象。**

&emsp;&emsp;计算监控树的所有节点所需的最小摄像头数量。



**示例 1：**

<img src="https://cdn.jsdelivr.net/gh/yleave/imagehost/img/image-20200922220425128.png" alt="image-20200922220425128" style="zoom:80%;" /> 



&emsp;&emsp;输入：`[0,0,null,0,0]`

&emsp;&emsp;输出：`1`

&emsp;&emsp;解释：如图所示，一台摄像头足以监控所有节点。

**示例 2：**

<img src="https://cdn.jsdelivr.net/gh/yleave/imagehost/img/image-20200922220447483.png" alt="image-20200922220447483" style="zoom:67%;" /> 

&emsp;&emsp;输入：`[0,0,null,0,null,0,null,null,0]`

&emsp;&emsp;输出：`2`

&emsp;&emsp;解释：需要至少两个摄像头来监视树的所有节点。 上图显示了摄像头放置的有效位置之一。



**提示：**

1. 给定树的节点数的范围是 `[1, 1000]`。
2. 每个节点的值都是 0。



## 解法1：递归+状态判断

&emsp;&emsp;仔细思考，根据题目，我们可以知道，对于一个节点 `node`，它只可能会存在三种状态（下图中**红色**代表安装监控器，**蓝色**代表被监控到，**绿色**代表未被监控到，或是监控与未监控两者都可）：

- `node` 任意一个孩子结点已安装监视器，那么当前 `node` 节点也会被监控到，这种状态记为 `1`；

  关于下面**右孩子的蓝色**，这边解释一下，因为我们是从下往上遍历的，且因为一些条件判断，我们会保证遇到这种情况时，孩子结点会处于监控状态，具体可看代码。

  <img src="https://cdn.jsdelivr.net/gh/yleave/imagehost/img/image-20200922213305179.png" alt="image-20200922213305179" style="zoom:80%;" />

- `node` 任意一个孩子节点未安装监视器，且该孩子节点也未被监控到，那么当前节点必须安装监视器，这种状态我们记为 `2`；

  <img src="https://cdn.jsdelivr.net/gh/yleave/imagehost/img/image-20200922214339811.png" alt="image-20200922214339811" style="zoom:80%;" />

- `node` 的孩子节点都没安装监视器，但是都被监控到了，那么当前节点可以安装监视器，也可以不安装监视器，但是本着节省资源的目的，需要将这种情况返回给父节点，让父节点做裁断（到底是父节点安装，还是当前节点安装）。

  <img src="https://cdn.jsdelivr.net/gh/yleave/imagehost/img/image-20200922214438620.png" alt="image-20200922214438620" style="zoom:80%;" />

  ​	**JS 代码：**

  ```js
  var minCameraCover = function(root) {
      let res = 0;
      
      const dfs = (node) => {
          if (!node) { // 对于空节点，默认其是未安装监视器且被监控到的状态
              return 3;
          }
  
          let left = dfs(node.left), right = dfs(node.right);
  		// 若有一个孩子结点未被监控到，那么当前节点需要安装监控器，造成的结果就是返回父节点时，表现是安装了监控器
          if (left === 2 || right === 2) {
              res++;
              return 1;
          }
  		// 否则，若有一个孩子结点安装了监控器，当前节点会处于被监控的状态，且上面的判断语句已经排除了有孩子结点未被监控到的情况，因此当前节点返回到其父节点的表现就是没装监控器但是被监控到的情况
          if (left === 1 || right === 1) {
              return 3;
          }
  		// 否则，当前节点会处于未装监控器且未被监控到的情况，也就是状态 2.
          return 2;
      };
  
      if (dfs(root) === 2) {
          res++;
      }
  
      return res;
  };
  ```

  

## 解法2：递归

&emsp;&emsp;**解法2是官方的解题思路，看了几遍才看懂.....因此附上一些自己的理解。**



&emsp;&emsp;**约定**：若某棵树的所有节点都被监控，则称这棵树被**覆盖**。



&emsp;&emsp;设当前节点为 `root`，其左右孩子为 `left` 和 `right` 。若想要覆盖以 `root` 为根的树，那么有两种情况：

- 情况1：若给 `root` 设置了摄像头，那么其孩子节点 `left`、`right` 也会被监控到。此时只需要保证 `left` 和 `right` **的**两棵子树也被覆盖即可。
- 情况2：若 `root` 没有设置摄像头，那么除了要保证以`left`为根结点的树 和 以`right`为根结点的树被覆盖之外，还需要保证 `left` 和 `right` 之中至少有一个节点安装了摄像头，这样 `root` 才能被监控到。



&emsp;&emsp;根据这两种情况，我们可以分析出，对于每个 `root` ，需要维护三种类型的状态：

- 状态 `a` ：`root` 必须安装摄像头的情况下（即情况1），覆盖整棵树需要的摄像头数量；
- 状态 `b` ：不论 `root` 是否安装摄像头，覆盖整棵树需要的摄像头数量；（也就是若 `root` 安置摄像头的话最好，不然 `left` 或 `right` 两个节点中需要至少一个有安置摄像头来保证 `root` 被监控）
- 状态 `c` ：不论 `root` 是否被**监控**到，覆盖两棵子树需要的摄像头数量；（对应着情况 1 和情况 2 中， `left`树 和 `right`树被覆盖，而`root` 是否被监控表示 `left` 或 `right` 节点可能有安置摄像头，也可能未安置摄像头）



&emsp;&emsp;根据以上定义，一定有 `a >= b >= c`



&emsp;&emsp;设 `root` 节点的左右孩子节点`left`、`right`维护的状态变量为：$(l_a,l_b,l_c)$ 和 $(r_a,r_b,r_c)$ ，那么：

- $a = l_c + r_c + 1$ ：即左右子树被覆盖所需的摄像头数量加上 `root` 安置的 1 个摄像头
- $b = min(a, min(l_a + r_b, l_b + r_a))$ ：即两种情况中的最小值，情况 2 中需要 `left` 和 `right` 中至少一个节点设置摄像头来保证 `root` 被监控，因此是 **`left` 设置 + `right` 未设置** 与 **`left` 未设置 + `right` 设置** 这两种取值中的最小值。

&emsp;&emsp;对于状态 `c` 而言，只需要保证 `left` 和 `right` 两棵子树被覆盖即可（包括 `left` 和 `right`）。

&emsp;&emsp;那么，要么 `root` 处设置一个摄像头（保证 `left` 和 `right` 这两个节点被监控）的情况下`left` 和 `right` **的**两棵子树刚好也被覆盖，那么就是状态`a` 的摄像头数量；

&emsp;&emsp;要么 `root` 处不放置摄像头，`left` 和 `right` 两棵子树保证自己被覆盖，所需的摄像头数量就是 $l_b + r_b$。



&emsp;&emsp;需要额外注意的是，对于 `root` 而言，如果其某个孩子为空，则不能通过在该孩子处放置摄像头的方式，监控到当前节点。因此，该孩子对应的变量 `a` 应当返回一个大整数，用于标识不可能的情形。



&emsp;&emsp;最终，根节点的状态变量 `b` 即为要求出的答案。



```js
var minCameraCover = function(root) {
    const dfs = (root) => {
        if (!root) {
            return [Math.floor(Number.MAX_SAFE_INTEGER / 2), 0, 0];
        }
        const [la, lb, lc] = dfs(root.left);
        const [ra, rb, rc] = dfs(root.right);
        const a = lc + rc + 1;
        const b = Math.min(a, Math.min(la + rb, ra + lb));
        const c = Math.min(a, lb + rb);
        return [a, b, c];
    }

    return dfs(root)[1];
};
```


