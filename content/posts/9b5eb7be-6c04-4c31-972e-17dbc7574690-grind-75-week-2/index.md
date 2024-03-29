---
title: "Grind 75 Week 2 回顧"
date: "2022-08-23T16:25:00Z"
tags: [leetcode]
---

# Grind 75 Week 2 回顧

---

## 小心得

有一些題目第一次寫，寫不出來了。例如 [Climbing Stairs](https://leetcode.com/problems/climbing-stairs)

{{< br >}}

## [Implement Queue using Stacks](https://leetcode.com/problems/implement-queue-using-stacks)

- Stack: FILO
- Queue: FIFO
1. 需要倒轉 Stack 的內容
    1. 把 Stack pop 的內容 push 到另外一個 Stack 就完成了倒轉 → 所以需要兩個 Stack
2. 要保證順序性，所以倒轉的過程只能發生在用來放倒轉元素的 Stack 為空的時候

{{< br >}}

## [First Bad Version](https://leetcode.com/problems/first-bad-version)

又是 Binary Search，要小心邊界問題以及重複問題

可以去看 Week 1 的 Binary Search， Carl 的影片很清楚

個人認為左閉右開區間很難懂，還是使用左閉右閉，即 `left` `right` 都是搜索範圍

{{< br >}}

## [Ransom Note](https://leetcode.com/problems/ransom-note)

map 解

{{< br >}}

## [Climbing Stairs](https://leetcode.com/problems/climbing-stairs)

第 n 階的上樓方法是：

- 第 (n-1) 階 + 1 → 上到第 (n-1) 階的方法數
- 第 (n-2) 階 + 2 → 上道第 (n-2) 階的方法數
- 第 (n-2) 階 + 1 + 1 → 已經被算過了不重複算

{{< br >}}

## [Longest Palindrome](https://leetcode.com/problems/longest-palindrome)

map 紀錄有多少成對的字元

回文中間可以是單獨的字元，只要判斷一下最後成對的加起來個數有沒有跟原始字串一樣長，

- 是 → 解
- 否 → 可以加一

{{< br >}}

## [Reverse Linked List](https://leetcode.com/problems/reverse-linked-list)

要記得下一個跟前一個，比兩兩交換稍微複雜一點

{{< br >}}

## [Majority Element](https://leetcode.com/problems/majority-element)

solution 很值得一看，有很多種想法介紹而且都很詳細

[Majority Element - LeetCode](https://leetcode.com/problems/majority-element/solution/)

Boyer-Moore Voting Algorithm

存在絕對多數的情況下，倆倆不同的元素湮滅，最後剩下的就是眾數

{{< br >}}

## [Add Binary](https://leetcode.com/problems/add-binary)

開一個比輸入稍大的 slice，從最後一個 bit 開始加

{{< br >}}

## [Diameter of Binary Tree](https://leetcode.com/problems/diameter-of-binary-tree)

這題我錯了很多次，後來發現兩個點之間的最長路線是可以不通過頂點的。

DFS 紀錄最長的左與右

{{< br >}}

## [Middle of the Linked List](https://leetcode.com/problems/middle-of-the-linked-list)

用快慢指針，快指針走到底，慢指針就在中間

{{< br >}}

## [Maximum Depth of Binary Tree](https://leetcode.com/problems/maximum-depth-of-binary-tree)

DFS

{{< br >}}

## [Contains Duplicate](https://leetcode.com/problems/contains-duplicate)

map

{{< br >}}