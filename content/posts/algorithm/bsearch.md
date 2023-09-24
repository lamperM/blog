---
title: "一道题搞定二分法的细节"
date: 2023-08-20T20:30:35+08:00
tags: [Algorithm]
categories: ["Algorithm"]
---

实际上我做过的二分搜索的题目并不少，但是一直以来没有静下心去研究它的
【循环条件】【边界调整】【返回值】的细节，通过这个题目希望自己能完整、
清晰的了解二分搜索。

## 题目

https://leetcode.cn/problems/find-first-and-last-position-of-element-in-sorted-array

> 给你一个按照非递减顺序排列的整数数组 nums，和一个目标值 target。请你找出给定目标值在数组中的开始位置和结束位置。
>
> 如果数组中不存在目标值 target，返回 [-1, -1]。
>
> 你必须设计并实现时间复杂度为 O(log n) 的算法解决此问题。
>
> > 示例 1：
> >
> > 输入：nums = [5,7,7,8,8,10], target = 8
> > 输出：[3,4]

## 解答(python):

```py
def searchRange(self, nums: List[int], target: int) -> List[int]:

    if nums == []:
    return [-1, -1]

    # 第一次二分，确定右边界
    left, right = 0, len(nums)-1
    while left <= right:
    mid = (left + right) // 2
    if nums[mid] <= target:
        left = mid+1
    else:
        right = mid-1
    end = right

    # 第二次二分，确定左边界
    left, right = 0, len(nums)-1
    while left <= right:
    mid = (left + right) // 2
    if nums[mid] < target:
        left = mid+1
    else:
        right = mid-1
    sta = left

    print(sta, end)
    if end < 0 or sta > len(nums)-1 or nums[sta] != target or nums[end] != target:
    return [-1, -1]
    else:
    return [sta, end]
```

## 细节

写好一个二分搜索就是需要确定三件事:

1. 循环边界条件
2. 调整边界
3. 返回值

这三件事是环环相扣的，先说确定的，调整边界的操作一定是`left=mid+1` or `right=mid-1`,
整数的二分不存在`left=mid`这种操作，仅限于浮点数中。

再说边界条件，我个人喜欢使用带等号的判断，即`while left <= right`, 这纯粹是个人习惯.



### 问题转化
以求右侧下标为例, 可以等价为: 搜索<=target的最大值. 这其实是问题的关键, 只是另外加一个判断说如果求得数
不是target, 返回特殊值就行了. 如果你还是不理解这两个问题为什么是等价的, 那么看完下面的解释应该也能清楚.

先说搜索<=target的最大值的计算方法, 一般的二分搜索, 搜索完成之后, 如果没有找到target, 
`right=left-1`, **且 right 指向比 target 首个小的元素,left 指向首个比 target 大的元素.**

那么代码是不是可以这么写:

```py
left, right = 0, n-1
while left <= right:
    mid = (left + right) // 2
    if nums[mid] == target:
        return mid
    elif nums[mid] < target:
        left += 1
    else:
        right += 1

return right
```

更进一步的说, 如果"把target也看作是小于target", 也就是让循环不停下, 最后平衡的条件也一定是满足 
**right 指向比 target 首个小的元素,left 指向首个比 target 大的元素.**, 只不过此时=target
也被算作是小于target, 走小于target的处理流程.
最终的结果就是right指向的就是target(如果存在),  要不就是比target小的那个数.

于是代码就可以被优化为:
```py
left, right = 0, n-1
while left <= right:
    mid = (left + right) // 2
    if nums[mid] <= target:
        left += 1
    else:
        right += 1

return right
```

### 再审原题

上述这种进一步思考的思想是不是与原题中的要求类似? 我可以有一连串的target, 我要求的是最右边的target在哪,
如果把=target也看作是小于target, 让循环继续跑, 最后right指向的就是最右边的那个target.