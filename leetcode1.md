---
title: leetcode笔记1
name: leetcode1
date: 2020-03-13
id: 0
tags: [leetcode]
categories: []
abstract: ""
---


力扣top算法题笔记1

<!--more-->

## 和链表相关的

### 两数之和

题目

```python
给定一个整数数组 nums 和一个目标值 target，请你在该数组中找出和为目标值的那 两个 整数，并返回他们的数组下标。

你可以假设每种输入只会对应一个答案。但是，你不能重复利用这个数组中同样的元素

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/two-sum
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。
```

思路

1. 分别遍历数组，使用map来计算第i个数字和后面n-i个数字的和。如果和为target
可以知道我们要找的索引就是i和map里的j+偏移量j+1
2. 使用散列表的方式来保存对应的两数的差值，如果target等于此散列表的键那么取出其值。键的值保存的是下标

实现

```python
class Solution:
    def twoSum(self, nums, target):
        """
        思路：遍历这个数组，使用map来计算第i个数字和后面n-i个数字的和。如果和为target
        可以知道我们要找的索引就是i和map里的j+偏移量j+1
        :param nums:
        :param target:
        :return:
        """
        for i in range(len(nums)):
            if i == len(nums):
                return False
            else:
                tmp = list(map(lambda x:x+nums[i],nums[i+1:]))
                for j in range(len(tmp)):
                    if tmp[j] == target:
                        return [i,j+i+1]
    def method2(self,nums,target):
        """
        思路使用散列表的方式，每次保存到散列表的时候顺便查找表中是否有对应的元素
        时间复杂度为On
        :param nums:
        :param target:
        :return:
        """
        map = {}
        for i in range(len(nums)):
            cmp = target - nums[i]
            if map.get(cmp)!=None:
                print(map)
                return [map[cmp],i]
            else:
                map[nums[i]] = i
                print(map)
```

### 两数相加

题目

```python
给出两个 非空 的链表用来表示两个非负的整数。其中，它们各自的位数是按照 逆序 的方式存储的，并且它们的每个节点只能存储 一位 数字。

如果，我们将这两个数相加起来，则会返回一个新的链表来表示它们的和。

您可以假设除了数字 0 之外，这两个数都不会以 0 开头。

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/add-two-numbers
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。
```

示例

```python
输入：(2 -> 4 -> 3) + (5 -> 6 -> 4)
输出：7 -> 0 -> 8
原因：342 + 465 = 807
```

思路

要注意每一位的计算，因为是十进制所以遇到10要进位。在链表的下一位存在时进行运算，得到进位`carry`如果其不为0那么`carry`对应的模就赋值为下一个节点的值。当运算完成链表遍历完成，判断最后一位的进位是否>0决定是否需要进位

```python
class ListNode:
    def __init__(self, x):
        self.val = x
        self.next = None

class Solution:
    def addTwoNumbers(self, l1: ListNode, l2: ListNode) -> ListNode:
        """
        最低位就在最左边，所以是按照链表顺序的相加
        加入使用遍历而不是递归的方式做题
        还需要注意进位的问题 4+6 = 0进1
        :param l1:
        :param l2:
        :return:
        """
        n = ListNode(0)
        current = n
        #最开始的进位是0
        carry = 0
        while l1 or l2:
            x = l1.val if l1 else 0
            y = l2.val if l2 else 0

            sum = carry + x + y
            #判断进位
            carry = sum // 10
            current.next = ListNode(sum%10)
            current = current.next
            if (l1 != None):l1 = l1.next
            if (l2 != None):l2 = l2.next

        #当都没有下一位时，判断最后一次是否有进位
        if carry > 0:
            current.next = ListNode(carry)

        return n.next
```

### 反转链表

题目 没找到准确描述

示例

```python
1>2>3>4
#反转后
1<2<3<4
```

我的思路

先讨论一般情况，假设首位从0开始，我们对链表进行一次遍历。可以发现到`4`这里以后不存在`next`节点。此时我们找到这样一个**子串**`2>3>4`，它经过一位反转后变为`2>3<4`。

这里我们进行了什么操作？`3.next = none`,`4.next=3`。可以把2看作第`n`位，此时的操作引申为

```python
#伪码
if n.next.next == None:
    n.next.next = n.next
    n.next = None
```

此时已经完成4的反转，假设我们继续操作。此时的`n`变为1，因为`1.next.next = None`。再次经过反转操作后，链表变为`1>2<3<4`

以此类推得到递归

```python
class ListNode:
    def __init__(self, x):
        self.val = x
        self.next = None
        
class Solution:
    def reverseList(self, head: ListNode) -> ListNode:
        if head == None or head.next== None:
            return head
        else:
            #从链表的头开始操作
            #使用递归来完成
            p = self.reverseList(head.next)
            head.next.next = head #反转方向
            head.next = None #去掉前面的方向
            return p
```

Landers的力扣笔记，使用[leetcode](https;//blog.lrenj.top/t/leetcode)标签查看全部文章

