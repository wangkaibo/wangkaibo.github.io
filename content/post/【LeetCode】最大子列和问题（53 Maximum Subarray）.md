---
title: "【LeetCode】最大子列和问题（53 Maximum Subarray）"
date: 2016-02-14T00:02:08+08:00
tags: ["LeetCode"]
draft: false
url: '/53-maximum-subarray'
---

最近在学习一些常用的经典算法，比较经典的一个问题就是给定一个序列，求连续子列的最大和。

用数学语言描述如下：

给定序列

`[-2,1,-3,4,-1,2,1,-5,4]`

计算出： $$max([0,..., \sum A_k])$$

<!--more-->

LeetCode 链接 [Maximum Subarray](https://leetcode.com/problems/maximum-subarray)

这个问题使用在线最优化求解(Online Optimization)算法是效率最高的,只需要维护两个变量，一次循环就可以完成。


```python

class Solution(object):
    def maxSubArray(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        max_sum =  nums[0]
        current_sum = 0
        for num in nums:
            current_sum += num
            if current_sum > max_sum:
                max_sum = current_sum
            if current_sum < 0:
                current_sum = 0; # 不可能使后面的和增大，抛弃
                
        return max_sum

```

下面是运行结果：
![image-20180604140044178](http://static.wangkaibo.com/FseOD1eKfuMOj3z0VX40aee2F3hf)

可见这是执行效率比较高的解法，时间复杂度为O(n)，但是 LeetCode 还推荐去练习另一种分而治之的解决办法，应该是类似二分法。有时间再去实现下。