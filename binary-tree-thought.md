---
title: 二叉树的转换思想
name: binary-tree-thought
date: 2020-03-24
id: 0
tags: [algorithm]
categories: []
abstract: ""
---


使用树的思想转化问题


<!--more-->


使用树的思想转化问题

<!--more-->

### 二叉树的DFS

假设有有一层楼梯，需要走到第n节楼梯(假设n=4)，每次只能走1步或者3步。请问有几种走法？

如果不使用传统的数学解法，可以把题目抽象为一个**DFS**的问题

![img](http://file.mgek.cc/images/blog/btt1.webp)

假设从根节点0开始走，左子树代表1步，右子树代表3步。那么问题转变为对这个二叉树的DFS问题，只需要找到叶子节点为4的个数

```python
ans = 0
n = 4 #需要到达的节点
def bt1(k):
    """
    k 根节点的值
    :param k:
    :return:
    """
    global ans
    if k==n:
        #找到一条路径
        ans+=1
        return
    elif k>n:
        return
    bt1(k+1)
    bt1(k+3)

if __name__ == '__main__':
      bt1(0)
      print(ans)
```

使用递归来进行深度优先遍历

### 优化后得DFS解法

使用上面得DFS算法解决这个问题时间复杂度为log(n^2)，为了优化算法使用记忆性得DFS算法

```python
class Solution2:
    """
    记忆化递归
    """
    def climbStairs(self, n: int) -> int:
        """
        memo是存储每次结果的数组
        :param n:
        :return:
        """
        memo = [0 for x in range(n)]
        return self.seek(0,n,memo)

    def seek(self,i,n,memo):
        if i>n:
            return 0
        if i==n:
            return 1
        if memo[i]>0:
            return memo[i]
        memo[i] = self.seek(i+1,n,memo)+self.seek(i+2,n,memo)
        return memo[i]
```

假设一个数组`memo`用于记录每次走的结果，`i`是当前所在的阶梯数(从0开始)，`n`是定义的要到达的阶梯数。

每次`i==n`时表示这次结果达到要求，`memo[i]`存储的即为从阶梯i走到阶梯n的可行结果数。

递归表达式：结果数=一步要走的结果数+2步要走的结果数

### 使用动态规划

动态规划把问题化为n个小步骤的问题解决

拆分问题，已知每次要么走一步，要么走两步。假设`dp[i]`表示走到第**i**阶楼梯的可行结果

那么最后的结果`dp[i]=dp[i-1] + dp[i-2]`

也就是最终结果=差一步到达和差两步到达的结果之和，已知`dp[1] = 1`,`dp[2] = 2`

```python
class Solution3:
    def climbStairs(self,n):
        """
        动态规划，使用dp记录第n阶需要多少次数
        一共两种爬法：第i-1阶爬1步
                     第i-2阶爬2步
        :param n:
        :return:
        """
        if n == 1:
            return 1
        dp = [0 for x in range(n+1)]
        dp[1] = 1
        dp[2] = 2
        for i in range(3,n+1):
            dp[i] = dp[i-1]+dp[i-2]
        return dp[n]
```

其中的i即为所需的4，代入后根据动态规划得到结果