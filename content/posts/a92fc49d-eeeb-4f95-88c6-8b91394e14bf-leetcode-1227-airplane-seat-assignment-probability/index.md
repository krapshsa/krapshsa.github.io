---
title: "LeetCode 1227. Airplane Seat Assignment Probability"
date: "2023-10-04T08:51:09Z"
tags: [leetcode]
---

# LeetCode 1227. Airplane Seat Assignment Probability

[LeetCode - The World's Leading Online Programming Learning Platform](https://leetcode.com/problems/airplane-seat-assignment-probability/description/)

當有 `k` 個位置時，有哪些可能？

1. 第 `1` 個人坐到他的位置 (機率 $\frac 1 k$)，則大家都照位置坐，所以 `k` 可以坐到他的位置 (乘 1)
2. 第 `1` 個人坐到第 `k` 個人的位置 (機率 $\frac 1 k$)，則 `k` 坐不到他的位置 (乘 0)
3. 第 `1` 個人坐到第 `2` 個人的位置 (機率 $\frac 1 k$)，則相當於第二個人是找不到票的人，有 `k-1` 個位置。要嘛他去坐第 `1` 個位置，或者他去搶別人的位置： $P_{k-1}$
4. 第 `1` 個人坐到第 `3` 個人的位置 (機率 $\frac 1 k$)，則相當於第三個人是找不到票的人，有 `k-2` 個位置 (大家順序上飛機，第 `2` 個人一定坐在 `2`。要嘛他去坐第 `1` 個位置，或者他去搶別人的位置： $P_{k-2}$

所以機率如下：

$P_k = \frac 1 k (1+P_{k-1} + ... + P_2)$

{{< br >}}

接著

$P_{k+1} = \frac 1 {k+1} (1+P_k+P_{k-1} + ... + P_2)$

$\because k P_k = (1+P_{k-1}+...P_2)$

$\therefore P_{k+1} = \frac 1 {k+1} (1+P_k+k(P_{k-1} + ... + P_2))$

$\therefore P_{k+1} = \frac 1 {k+1} (1+P_k+kP_k)$

$\therefore  P_{k+1} = \frac 1 {k+1} (1+k)P_k$

$\therefore  P_{k+1} = P_k$

{{< br >}}

所以程式碼就變成：

```Go
func nthPersonGetsNthSeat(n int) float64 {
    if n == 1 {
        return 1.0
    }

    return 0.5
}
```