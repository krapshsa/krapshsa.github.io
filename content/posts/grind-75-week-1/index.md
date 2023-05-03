---
title: "Grind 75 Week 1 回顧"
date: "2022-08-20T06:27:00Z"
tags: [leetcode]
---

# Grind 75 Week 1 回顧

---

## 小心得

Week 1 的題目都是 Easy，(除了 53. Maximum Subarray 現在難度被重新評級為 Medium)

雖然不至於寫不出來，但是還是有很容易寫錯的題目。

發現我對於臨界值的掌握有點差：

1. 經常發生越界
2. 不知道可以剪枝 (有時候會遞迴重複項目)

{{< br >}}

## [Two Sum](https://leetcode.com/problems/two-sum)

暴力法，用兩個 Loop 嘗試各種組合

{{< br >}}

## [Valid Parentheses](https://leetcode.com/problems/valid-parentheses)

Stack，成對的就 pop 掉，不然就塞進 Stack 內

{{< br >}}

## [Merge Two Sorted Lists](https://leetcode.com/problems/merge-two-sorted-lists)

相同大小無順序，就把兩個 List 都輪流遍歷就可以了

終止條件是 List 都為空就做完了

{{< br >}}

## [Best Time to Buy and Sell Stock](https://leetcode.com/problems/best-time-to-buy-and-sell-stock)

買點是各個可能的谷底，在從左到右的過程中，如果遇到更低的價格，就更新潛在最佳買點。

但是這題沒有要求是哪個點，所以就計算最大利潤存起來就好。

{{< br >}}

## [Valid Palindrome](https://leetcode.com/problems/valid-palindrome)

題目寫：

>  Alphanumeric characters include letters and numbers.

我漏掉了數字，WA 一次，這題我花比較多時間，因為第一次寫 Go 不知道到底要怎麼判斷字符。

{{< br >}}

## [Invert Binary Tree](https://leetcode.com/problems/invert-binary-tree)

遞迴交換，新的右邊是舊的左邊 `invertTree`，左邊也是同樣道理。

{{< br >}}

## [Valid Anagram](https://leetcode.com/problems/valid-anagram)

用 hasmap 存出現頻率

延伸題目是 unicode 怎麼辦？用 go 寫很容易，做一個 key 是 rune 的 map 就好了

    letterMap := make(map[rune]int)

{{< br >}}

## [Binary Search](https://leetcode.com/problems/binary-search)

心態打擊，逢寫必錯

推薦看一下 Carl 大大的手撕二分法

[手把手带你撕出正确的二分法 | 二分查找法 | 二分搜索法 | LeetCode：704. 二分查找_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1fA4y1o715)

學習重點：

- 循環不變量
- 區間

這題我使用左閉右閉區間，也就是終止條件應該是 `left ≤ right`

並解用過的區間就不應該再用了，不然會導致無窮的做下去

{{< br >}}

## [Flood Fill](https://leetcode.com/problems/flood-fill)

我把指定的點的上下左右丟進 queue 裡面，再拿出來處理。

要注意邊界，以及不要重複做。

{{< br >}}

## [Maximum Subarray](/25ae9972307340bca1b30ddf7ae7fb41)

一個解的左邊元素以及右邊元素只有可能是：

1. 空的
2. 和是負的 → 若是正的，可以加起來形成更大的 subarray

想法是從左到右找，並且紀錄：

1. 最大的和 → 可能解
2. 現在的和 → 可能可以貢獻到解 → 一旦為負就表示不可能是解一部分 → 重置要貢獻的和

Edge Case：全部都是負的

WA

    func maxSubArray(nums []int) int {
    	maxSum := nums[0]
    	curSum := 0
    
    	for i := 0; i < len(nums); i++ {
    		curSum += nums[i]
    		if curSum < 0 {
    			curSum = 0
    			continue
    		}
    
    		if curSum > maxSum {
    			maxSum = curSum
    		}
    	}
    
    	return maxSum
    }

AC1

    func maxSubArray(nums []int) int {
    	maxSum := nums[0]
    	curSum := 0
    
    	for i := 0; i < len(nums); i++ {
    		curSum += nums[i]
    		if curSum > maxSum {
    			maxSum = curSum
    		}
    		if curSum < 0 {
    			curSum = 0
    		}
    	}
    
    	return maxSum
    }

AC2

    func maxSubArray(nums []int) int {
    	maxSum := nums[0]
    	curSum := nums[0]
    
    	for i := 1; i < len(nums); i++ {
        if curSum < 0 {
    			curSum = nums[i]
        } else {
          curSum += nums[i]
        }
    
    		if curSum > maxSum {
    			maxSum = curSum
    		}
    	}
    
    	return maxSum
    }

## [Lowest Common Ancestor of a Binary Search Tree](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-search-tree)

給兩個點 p, q 在過程中，如果我找到一個點：

1. 其中一個是 p 或 q → 高的是 ancestor
2. p, q 在不同邊 → 該點是 ancestor

另外一個想法是：

1. 都在左側 → 左邊那個點是可能的 ancestor，往左找
2. 都在右側 → 右邊那個點是可能的 ancestor，往右找
3. 其他：找到了

    func lowestCommonAncestor(root, p, q *TreeNode) *TreeNode {
    	lowest := root
    
    	for true {
    		if lowest.Val > p.Val && lowest.Val > q.Val {
    			lowest = lowest.Left
    		} else if lowest.Val < p.Val && lowest.Val < q.Val {
    			lowest = lowest.Right
    		} else {
    			break
    		}
    	}
    
    	return lowest
    }

{{< br >}}

## [Balanced Binary Tree](https://leetcode.com/problems/balanced-binary-tree)

遞迴：DFS 回傳值為是否 balanced 以及深度

{{< br >}}

## [Linked List Cycle](https://leetcode.com/problems/linked-list-cycle)

NeetCode 講得很清楚

[Linked List Cycle - Floyd's Tortoise and Hare - Leetcode 141 - Python](https://www.youtube.com/watch?v=gBTe7lFR3vc)

沒有 cycle → 最後會走到 nil

有 cycle → 快慢指針相遇

這題的延伸題是求解 cycle 的開端