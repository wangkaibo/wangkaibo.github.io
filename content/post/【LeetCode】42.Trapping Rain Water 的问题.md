---
title: "【LeetCode】42.Trapping Rain Water 问题"
date: 2016-02-23T23:09:57+08:00
tags: ["LeetCode"]
draft: false
url: '/trapping-rain-water'
---

给定的 n 个非负整数表示每个宽度为 1 栅栏的海拔地图的高度，计算在下雨之后能够捕获多少水？

![示例图](http://www.leetcode.com/static/images/problemset/rainwatertrap.png)

<!--more-->

#### 问题英文描述

Given n non-negative integers representing an elevation map where the width of each bar is 1, compute how much water it is able to trap after raining.

举例：

给出 [0,1,0,2,1,0,1,3,2,1,2,1], 返回 6。


这个题目有个最好理解的思路，就是先找到最高点，然后将整个 List 分成两部分去解决。
找到最高点后，采取从两边「爬楼梯」的方式来统计存水的多少，正常的楼梯是不会存水的，因为楼梯都是节节高的。从左右两边开始爬，爬楼的过程中维护当前爬楼的临时最高值。低于这个临时最高值的一楼肯定会存水，这时很容易就能计算出存水量。

算法的时间复杂度O(n)

Python 都快忘了，顺便练习一下。下面是 Python 实现：

```python
class Solution(object):
    def trap(self, height):
        """
        :type height: List[int]
        :rtype: int
        """
        # 找到最大值
        max_index = height.index(max(height))

        height_len = len(height)
        
        # 初始化左边当前最大值
        current_max = 0;
        
        # 初始化总存水量
        total = 0
        
        # 爬左楼
        for i in range(max_index):
            if height[i] > current_max:
                # 维护当前最大值
                current_max = height[i]
            else:
                # 累计存水量
                total += current_max - height[i]
        
        # 初始化左边当前最大值
        current_max = height[height_len - 1];
        
        # 爬右楼
        for i in range(max_index, height_len)[::-1]:
            if height[i] > current_max:
                current_max = height[i]
            else:
                total += current_max - height[i]
        print total

height_list = [0,1,0,2,1,0,1,3,2,1,2,1]

solution = Solution()

solution.trap(height_list)
```