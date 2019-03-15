---
title: Algorithm 笔记
date: 2019-03-14 15:13:54
tags: [算法, Go, Note]
categories: 算法
permalink: Alg-Note
---
# 数组  
## 有序数组的平方  
[题目](https://leetcode-cn.com/problems/squares-of-a-sorted-array/):  
```go
给定一个按非递减顺序排序的整数数组 A，返回每个数字的平方组成的新数组，要求也按非递减顺序排序。

示例 1：
输入：[-4,-1,0,3,10]
输出：[0,1,9,16,100]

示例 2：
输入：[-7,-3,2,3,11]
输出：[4,9,9,49,121]
```

思路：双指针。从前、后开始算平方，比大小。  
```go
func sortedSquares(A []int) []int {
        result := make([]int, len(A))
    top := 0
    end := len(A) - 1

    for i := len(A) - 1; i >= 0; i-- {
        topVal := A[top] * A[top]
        endVal := A[end] * A[end]

        if topVal > endVal {
            result[i] = topVal
            top++
        } else {
            result[i] = endVal
            end--
        }
    }

    return result
}
```

## 按奇、偶顺序排序数组  
[题目](https://leetcode-cn.com/problems/sort-array-by-parity/)：  
```go
给定一个非负整数数组 A，返回一个由 A 的所有偶数元素组成的数组，后面跟 A 的所有奇数元素。

你可以返回满足此条件的任何数组作为答案。

示例：

输入：[3,1,2,4]
输出：[2,4,3,1]
输出 [4,2,3,1]，[2,4,1,3] 和 [4,2,1,3] 也会被接受。
```
思路：构建两个slice，分别append奇数和偶数，然后把两个slice拼起来。

## 斐波那契数列  
[题目](https://leetcode-cn.com/problems/fibonacci-number/):  
```
斐波那契数，通常用 F(n) 表示，形成的序列称为斐波那契数列。该数列由 0 和 1 开始，后面的每一项数字都是前面两项数字的和。也就是：

F(0) = 0,   F(1) = 1
F(N) = F(N - 1) + F(N - 2), 其中 N > 1.
给定 N，计算 F(N)。
```
思路：因为提前给了N，所以可以直接构建长度为N的slice来计算数列，而不是无脑append，花时间。  
```go
    S := make([]int, N)
```
## 数组拆分：求min(ai, bi)和的最大值  
[题目]()：  
```go
给定长度为 2n 的数组, 你的任务是将这些数分成 n 对, 例如 (a1, b1), (a2, b2), ..., (an, bn) ，使得从1 到 n 的 min(ai, bi) 总和最大。

示例 1:

输入: [1,4,3,2]

输出: 4
解释: n 等于 2, 最大总和为 4 = min(1, 2) + min(3, 4).
提示:

n 是正整数,范围在 [1, 10000].
数组中的元素范围在 [-10000, 10000].
```
思路：先排序，再求min的和。因为给了n的个数范围，所以可以构建2n+1容量的slice来桶排序：  
```go
func arrayPairSum(nums []int) int {
    var arr [20001]int
    for _, v := range nums {
        arr[v+10000]++
    }
    sum, flag := 0, true
    for i, v := range arr {
        for v > 0 {
            if flag {
                sum += i - 10000
            }
            v--
            flag = !flag
        }
    }
    return sum
}
```
## 杨辉三角形1：返回前N行  
[题目](https://leetcode-cn.com/problems/pascals-triangle)  
```go
func generate(numRows int) [][]int {
    A := make([][]int, numRows)
    for i := range A {
        A[i] = make([]int, i+1)
        for j, _ := range A[i] {
            switch j {
            case 0:
                A[i][j] = 1
            case i:
                A[i][j] = 1
            default:
                A[i][j] = A[i-1][j-1] + A[i-1][j]
            }
        }
    }
    return A
}
```

## 杨辉三角形2： 返回第N行  
[题目](https://leetcode-cn.com/problems/pascals-triangle-ii/)  
```go
func getRow(rowIndex int) []int {
    i := 0
    ret := make([]int,rowIndex+1)
    for i <= rowIndex {
        if i == 0 {
            ret[0] = 1
        }else if i == 1 {
            ret[1] = 1
        }else{
            tmp := 0
            for index,v := range ret {
                if index > 0 {
                    tmp,ret[index] = ret[index],ret[index] + tmp
                }else{
                    tmp = v
                }
            }
        }
        i ++
    }
    return ret
}
```
## 求众数：出现次数大于n/2的元素  
[题目](https://leetcode-cn.com/problems/majority-element/)：  
```go
给定一个大小为 n 的数组，找到其中的众数。众数是指在数组中出现次数大于 ⌊ n/2 ⌋ 的元素。

你可以假设数组是非空的，并且给定的数组总是存在众数。

示例 1:
输入: [3,2,3]
输出: 3

示例 2:
输入: [2,2,1,1,1,2,2]
输出: 2
```
思路：从第一个数开始count=1，遇到相同的就+1，不同的就-1，减到0时候就重新换个数开始计数：  
```go
func majorityElement(nums []int) int {
    count := 1
    res := nums[0]
    for i := 1; i < len(nums); i++ {
        if nums[i] == res {
            count++
        } else {
            count--
            if count == 0 {
                res = nums[i]
                count = 1
            }
        }
    }
    return res
}
```
## 重塑矩阵  
[题目](https://leetcode-cn.com/problems/reshape-the-matrix/)：  
```go
在MATLAB中，有一个非常有用的函数 reshape，它可以将一个矩阵重塑为另一个大小不同的新矩阵，但保留其原始数据。

给出一个由二维数组表示的矩阵，以及两个正整数r和c，分别表示想要的重构的矩阵的行数和列数。

重构后的矩阵需要将原始矩阵的所有元素以相同的行遍历顺序填充。

如果具有给定参数的reshape操作是可行且合理的，则输出新的重塑矩阵；否则，输出原始矩阵。

示例 1:
输入: 
nums = 
[[1,2],
 [3,4]]
r = 1, c = 4
输出: 
[[1,2,3,4]]
解释:
行遍历nums的结果是 [1,2,3,4]。新的矩阵是 1 * 4 矩阵, 用之前的元素值一行一行填充新矩阵。

示例 2:
输入: 
nums = 
[[1,2],
 [3,4]]
r = 2, c = 4
输出: 
[[1,2],
 [3,4]]
解释:
没有办法将 2 * 2 矩阵转化为 2 * 4 矩阵。 所以输出原矩阵。
注意：

给定矩阵的宽和高范围在 [1, 100]。
给定的 r 和 c 都是正数。
```
思路：用`A[n/c][n%c]`来构建矩阵：  
```go
func matrixReshape(nums [][]int, r int, c int) [][]int {
    if len(nums)*len(nums[0]) != r * c{
        return nums
    }
    A := make([][]int,r)
    for rownum:= range A{
        A[rownum]=make([]int,c)
    }
    var n int
    for rowNum := range nums{
        for i,_ := range nums[rowNum]{
            n= rowNum*len(nums[0])+i
            A[n/c][n%c]=nums[rowNum][i]
        }
    } 
    return A
}
```
## 移除元素  
[题目](https://leetcode-cn.com/problems/remove-element/)：  
```go
给定一个数组 nums 和一个值 val，你需要原地移除所有数值等于 val 的元素，返回移除后数组的新长度。

不要使用额外的数组空间，你必须在原地修改输入数组并在使用 O(1) 额外空间的条件下完成。

元素的顺序可以改变。你不需要考虑数组中超出新长度后面的元素。

示例见上方超链接
```
思路：双指针。发现val时，交换跟末尾的数的值。  
```go
func removeElement(nums []int, val int) int {
    var i=0
    var n= len(nums)
    for i<n{
        if nums[i]==val{
            nums[i]=nums[n-1]
            n--
        }else{
            i++
             }
        
    }
    return n
}
```
## 移动零到数组末尾  
[题目](https://leetcode-cn.com/problems/move-zeroes/)：  
```go
给定一个数组 nums，编写一个函数将所有 0 移动到数组的末尾，同时保持非零元素的相对顺序。
示例:
输入: [0,1,0,3,12]
输出: [1,3,12,0,0]

说明:
必须在原数组上操作，不能拷贝额外的数组。
尽量减少操作次数。
```
思路：双指针。1个指针用来遍历，1个指针用来填非零值，然后把剩下的值改0。  
```go
func moveZeroes(nums []int)  {
    dist := 0
    for _, num := range nums {
        if num != 0 {
            nums[dist] = num
            dist++
        }
    }

    for i := dist; i < len(nums); i++ {
        nums[i] = 0
    }
    
}
```
## 买卖股票的最佳时机  
[题目](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-ii/)：  
```go
给定一个数组，它的第 i 个元素是一支给定股票第 i 天的价格。

设计一个算法来计算你所能获取的最大利润。你可以尽可能地完成更多的交易（多次买卖一支股票）。

注意：你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）。

示例 1:
输入: [7,1,5,3,6,4]
输出: 7
解释: 在第 2 天（股票价格 = 1）的时候买入，在第 3 天（股票价格 = 5）的时候卖出, 这笔交易所能获得利润 = 5-1 = 4 。
     随后，在第 4 天（股票价格 = 3）的时候买入，在第 5 天（股票价格 = 6）的时候卖出, 这笔交易所能获得利润 = 6-3 = 3 。

示例 2:
输入: [1,2,3,4,5]
输出: 4
解释: 在第 1 天（股票价格 = 1）的时候买入，在第 5 天 （股票价格 = 5）的时候卖出, 这笔交易所能获得利润 = 5-1 = 4 。
     注意你不能在第 1 天和第 2 天接连购买股票，之后再将它们卖出。
     因为这样属于同时参与了多笔交易，你必须在再次购买前出售掉之前的股票。

示例 3:
输入: [7,6,4,3,1]
输出: 0
解释: 在这种情况下, 没有交易完成, 所以最大利润为 0。
```
思路：最简单粗暴的方法是遍历，后一位的值大于前一位的值时候，利润就加差值累计。  
还可以寻找峰、谷，然后求峰谷的差的和：  
```go
func maxProfit(prices []int) int {
    if len(prices)==0{
        return 0
    }
    max, min, res, i := prices[0], prices[0], 0, 0
    for i < len(prices)-1 {
        for i < len(prices)-1 && prices[i] >= prices[i+1] {
            i++
        }
        min = prices[i]
        for i < len(prices)-1 && prices[i] <= prices[i+1] {
            i++
        }
        max = prices[i]
        res += max - min
    }
    return res
}
```
## 找数组中消失的数字  
[题目](https://leetcode-cn.com/problems/find-all-numbers-disappeared-in-an-array/):  
```go
给定一个范围在  1 ≤ a[i] ≤ n ( n = 数组大小 ) 的 整型数组，数组中的元素一些出现了两次，另一些只出现一次。

找到所有在 [1, n] 范围之间没有出现在数组中的数字。

您能在不使用额外空间且时间复杂度为O(n)的情况下完成这个任务吗? 你可以假定返回的数组不算在额外空间内。

示例:
输入:
[4,3,2,7,8,2,3,1]
输出:
[5,6]
```
思路：将所有正数为置为相反数。那么，出现正数的位置即为消失的数字。比如，[4,3,2,7,8,2,3,1]  

重置后将为[-4,-3,-2,-7,8,2,-3,-1]  

[8,2] 分别对应的index为[5,6]。  
```go
func findDisappearedNumbers(nums []int) []int {
    for i := 0; i < len(nums); i++ {
        nums[Abs(nums[i])-1] = -Abs(nums[Abs(nums[i])-1])
    }
    res :=[]int{}
    for i:=0;i<len(nums);i++{
        if nums[i]>0{
            res=append(res,i+1)
        }
    }
    return res
}
func Abs(x int) int {
    if x < 0 {
        return -x
    }
    return x
}

```

## 最大自序和  
[题目](https://leetcode-cn.com/problems/maximum-subarray/)：  
```go
给定一个整数数组 nums ，找到一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。

示例:
输入: [-2,1,-3,4,-1,2,1,-5,4],
输出: 6
解释: 连续子数组 [4,-1,2,1] 的和最大，为 6。
```
思路：单指针遍历求和，如果当前的和小于等于0，就直接从下一个数开始重新求和。
```go
func maxSubArray(nums []int) int {
    maxSubSum, sum := nums[0], 0
    
    for i := 0; i < len(nums); i++ {
        if sum > 0 {
            sum += nums[i]
        } else {
            sum = nums[i]
        }
        if sum > maxSubSum {
            maxSubSum = sum
        }
    }
    
    return maxSubSum
}
```

## 合并两个有序数组  
[题目](https://leetcode-cn.com/problems/merge-sorted-array/)：  
```go
给定两个有序整数数组 nums1 和 nums2，将 nums2 合并到 nums1 中，使得 num1 成为一个有序数组。

说明:
初始化 nums1 和 nums2 的元素数量分别为 m 和 n。
你可以假设 nums1 有足够的空间（空间大小大于或等于 m + n）来保存 nums2 中的元素。
示例:
输入:
nums1 = [1,2,3,0,0,0], m = 3
nums2 = [2,5,6],       n = 3
输出: [1,2,2,3,5,6]
```
思路：短数组往大数组里填，从大数组屁股开始填，谁大先填谁。  
如果先把nums1填完了，num2还剩一些没填完，要把这部分剩下的转移到nums1中：  
```go
func merge(nums1 []int, m int, nums2 []int, n int)  {
    for m > 0 && n > 0 {
        if nums1[m-1] > nums2[n-1] {
            nums1[m+n-1] = nums1[m-1]
            m--
        } else {
            nums1[m+n-1] = nums2[n-1]
            n--
        }
    }
    if n > 0 {
        for ; n > 0; n-- {
            nums1[n-1] = nums2[n-1]
        }
    }
}
```



---
*`to be continued...`*