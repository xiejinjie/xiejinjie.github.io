---
layout: post
title:  "lecode-两数相加"
date:   2019-08-13 20:35:54 +0800
categories: tech
---

[leetcode002两数相加](https://leetcode-cn.com/problems/add-two-numbers/),java递归解法
<!-- more -->

## 题目
给出两个 非空 的链表用来表示两个非负的整数。其中，它们各自的位数是按照 逆序 的方式存储的，并且它们的每个节点只能存储 一位 数字。

如果，我们将这两个数相加起来，则会返回一个新的链表来表示它们的和。

您可以假设除了数字 0 之外，这两个数都不会以 0 开头。

示例：

```
输入：(2 -> 4 -> 3) + (5 -> 6 -> 4)
输出：7 -> 0 -> 8
原因：342 + 465 = 807
```

## 分析
- 链表按照『个十百千万』顺序存储数字
- 依次将两条链表对应结点相加，加和大于十的同时进位
- 递归需要一个进位参数，和两个对应节点的引用参数
- 递归终止条件是两条链表都到达终点
- 递归过程是将两个结点的数字与进位数相加；对10取余数创建新结点并返回；递归下一个结点并建立新链表

## 代码
```
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        return add(l1,l2,0);
    }
    public ListNode add(ListNode l1, ListNode l2, int a){
        if(l1 == null && l2 == null && a == 0) return null;
        int x = l1==null ? 0 : l1.val; 
        int y = l2==null ? 0 : l2.val;
        int sum = x + y + a;
        ListNode n = new ListNode(sum % 10);
        n.next = add(l1==null ? null : l1.next,
                     l2==null ? null : l2.next,
                     sum/10);
        return n;
    }
}
```