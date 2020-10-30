---
title: NC93设计LRU缓存结构
index_img: 'https://cdn.jsdelivr.net/gh/yleave/imagehost/index_img/NK.jpg'
banner_img: 'https://cdn.jsdelivr.net/gh/yleave/imagehost/banner_img/44.png'
date: 2020-10-29 14:00:33
categories:
    - 算法题
    - 解答
    - 牛客网
tags:
    - LRU
    - 双向链表
---


[题目地址](https://www.nowcoder.com/practice/e3769a5f49894d49b871c09cadd13a61?tpId=117&&tqId=35015&rp=1&ru=/ta/job-code-high&qru=/ta/job-code-high/question-ranking)

---

**题目简述：**

&emsp;&emsp;设计数据结构，模拟 LRU（Least Recently Used），且要实现两个时间复杂度为 `O(1)` 的方法：

- `set(key, value)` ：往 LRU 结构中插入记录 `key -> value`
-  `get(key)` ：从 LRU 结构中获取 `key` 对应的 `value` ，若无该记录，返回 `-1`

&emsp;&emsp;每当使用了这两个方法之一，这个 `key` 记录就会变成当前最常用的记录；限制了存储容量 `k`，当保存的记录条数超过 `k` 时，就将其中最不常用的记录删除。

&emsp;&emsp;定义了两个操作 `1` 和 `2` ，`1` 代表了`set`操作，`2` 代表了`get`操作，

&emsp;&emsp;**输入**：`[[1,1,1],[1,2,2],[1,3,2],[2,1],[1,4,4],[2,2]],3`

&emsp;&emsp;**输出**：`[1,-1]`

&emsp;&emsp;**说明**：输入代表了一系列操作，`[1,1,1]` 中第一个数字为操作，后两个数字为 `key` 和 `value`，`[2, 1]` 第一个数字代表操作，第二个数字代表 `key`，最后一个数字 `3` 代表这个结构的容量。



## 解法1：queue + map

&emsp;&emsp;定义一个 `queue ` 和一个 `map`。

&emsp;&emsp;**`queue `** 是一个队列，用于存储结构中现存的 `key`，**队列中 `key` 的顺序就表示了这条记录的常用程度**，处于队尾的最常用，处于队首的最不常用。

&emsp;&emsp;当存储容量仍有剩时，往队尾中加入这个 `key`，当超出了存储容量时，从队首移除这个 `key`。当发生了`get` 或 `set` 操作时，从这个队列中查找是否有对应记录，若有，将其移至队尾。

&emsp;&emsp;`map` 则用于存储记录的键值对，每当插入一个 `key，value` 时，先查找是否存在 `key` 对应的记录，若存在，则更新这条记录的 `value` ，否则新增一条记录。



&emsp;&emsp;这种方法逻辑上可行，但在进行  `set` 或 `get` 操作时，若已存在该记录，则需要将其移至队尾，这样的操作在 `O(1)` 的时间内是无法完成 的。

**JS 代码如下：**

```js
/**
 * lru design
 * @param operators int整型二维数组 the ops
 * @param k int整型 the k
 * @return int整型一维数组
 */
function LRU( operators ,  k ) {
    let map = new Map();
    let list = [];
    let ans = [];
    
    for (let data of operators) {
        let op = data[0];
        let key = data[1], value = data[2];
        if (op === 1) {
            if (list.length >= k) {
                list.shift();
            }
            // 若存在该记录，需要移至队尾
            if (map.get(key)) {
                list.splice(list.indexOf(key), 1);
            }
            map.set(key, value);
            list.push(key);
        } else if (op === 2) {
            let i = list.indexOf(key);
            if (i === -1) {
                ans.push(-1);
            } else {
                list.splice(i, 1);
                list.push(key);
                ans.push(map.get(key));
            }
        }
    }
    return ans;
}
module.exports = {
    LRU : LRU
};
```



## 解法2：双向链表 + map

&emsp;&emsp;解法来源于[解答](https://blog.nowcoder.net/n/d2318a5e738349e194af51fb329ef504?f=comment)

&emsp;&emsp;定义一个双向链表结构和两个辅助节点：`head`、`tail`，再定义一个 `map` 根据 `key` 来存储一个节点。

&emsp;&emsp;每当进行 `set(key,value)` 操作时，检索 `map` 中是否存在对应节点，若存在，更新其 `value`，并将这个节点从当前位置移动到头结点之后，若不存在，则直接往头结点后插入节点 `ListNode(key, value)` ，当容量超出时，需要删除的最不常用的节点就是尾节点之前的节点。

&emsp;&emsp;而对于 `get(key)` 操作也很简单，检索 `map` ，若不存在则返回 `-1`，存在则返回对应节点的 `value`，并将这个节点移至头结点之后。



&emsp;&emsp;**且由于是双向链表，对于插入和删除操作，我们都能够在 `O(1)` 时间内完成。**

&emsp;&emsp;不过比较坑的是，虽然两个操作都能在 `O(1)` 时间内完成，但是牛客网上还是通过不了，会超时，用 C++ 版本的就能正常通过...

**JS 代码如下：**

```js
/**
 * lru design
 * @param operators int整型二维数组 the ops
 * @param k int整型 the k
 * @return int整型一维数组
 */
function LRU( operators ,  k ) {
    let map = new Map();
    let ans = [];

    function ListNode(key, val) {
        this.key = key;
        this.val = val;
        this.pre = null;
        this.next = null;
    }

    let head = new ListNode(-1, -1);
    let tail = new ListNode(-1, -1);
    head.next = tail;
    tail.pre = head;
	
    // 将新节点插入头结点之后
    const insertItem = (item) => {
        item.next = head.next;
        item.pre = head;
        head.next.pre = item;
        head.next = item;
    };
	
    // 将节点从原先的位置移至头结点之后
    const moveToHead = (item) => {
        item.pre.next = item.next;
        item.next.pre = item.pre;
        item.next = head.next;
        item.next.pre = item;
        head.next = item;
        item.pre = head;

    };
    
	// 删除尾节点前面的一个节点
    const removeItem = () => {
        tail.pre = tail.pre.pre;
        tail.pre.next = tail;
    };

    for (let data of operators) {
        let op = data[0];
        let key = data[1], value = data[2];
        if (op === 1) {
            if (map.size >= k) {
                map.delete(tail.pre.key);
                removeItem();
            }
            if (!map.has(key)) {
                let node = new ListNode(key, value);
                insertItem(node);
                map.set(key, node);
            } else {
                let node = map.get(key);
                node.val = value;
                moveToHead(node);
            }
        } else if (op === 2) {
            if (!map.has(key)) {
                ans.push(-1);
            } else {
                let node = map.get(key);
                ans.push(node.val);
                moveToHead(node);
            }
        }
    }
    return ans;
}
module.exports = {
    LRU : LRU
};
```