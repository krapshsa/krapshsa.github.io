---
title: "用快速冪與矩陣乘法解費氏數列"
date: "2023-09-28T02:58:48Z"
tags: [leetcode]
---

# 用快速冪與矩陣乘法解費氏數列

# 快速冪

[LeetCode - The World's Leading Online Programming Learning Platform](https://leetcode.com/problems/powx-n/description/)

```Go
func fPow(x float64, n int) float64 {
    if n == 1 {
        return x
    }

    p := fPow(x, n/2)

    if n % 2 == 0 {
        return p * p
    } else {
        return p * p * x
    }
}

func myPow(x float64, n int) float64 {
    if n == 0 || x == 1.0 {
        return 1.0
    }

    if n < 0 {
        return 1/fPow(x, -1*n)
    }

    return fPow(x, n)
}
```

{{< br >}}

# 矩陣解費氏數列

首先，我們知道費氏數列的定義是：

$F(n)=F(n−1)+F(n−2)$

其中 $F(0)=0$ 和 $F(1)=1$

考慮以下的矩陣乘法：

$$\\begin{bmatrix} 1 & 1 \\\\ 1 & 0 \\\\ \\end{bmatrix} \\times \\begin{bmatrix} F(n) \\\\ F(n-1) \\\\ \\end{bmatrix} = \\begin{bmatrix} F(n-1) + F(n) \\\\ F(n) \\\\ \\end{bmatrix} = \\begin{bmatrix} F(n+1) \\\\ F(n) \\\\ \\end{bmatrix} $$

也就是說：

$$\\begin{bmatrix} 1 & 1 \\\\ 1 & 0 \\\\ \\end{bmatrix} ^{n-1} \\times \\begin{bmatrix} F(1) \\\\ F(0) \\\\ \\end{bmatrix} = \\begin{bmatrix} F(n) \\\\ F(n-1) \\\\ \\end{bmatrix} $$

再者：

$$\\begin{bmatrix} 1 & 1 \\\\ 1 & 0 \\\\ \\end{bmatrix} ^{n-1} \\times \\begin{bmatrix} F(2) \\\\ F(1) \\\\ \\end{bmatrix} = \\begin{bmatrix} F(n+1) \\\\ F(n) \\\\ \\end{bmatrix} $$

可以整理成：

$$\\begin{bmatrix} 1 & 1 \\\\ 1 & 0 \\\\ \\end{bmatrix} ^{n-1} \\times \\begin{bmatrix} F(2) & F(1) \\\\ F(1) & F(0) \\\\ \\end{bmatrix} = \\begin{bmatrix} F(n+1) & F(n) \\\\ F(n) & F(n-1) \\\\ \\end{bmatrix} $$

$$\\therefore \\begin{bmatrix} 1 & 1 \\\\ 1 & 0 \\\\ \\end{bmatrix} ^{n-1} \\times \\begin{bmatrix} 1 & 1 \\\\ 1 & 0 \\\\ \\end{bmatrix} = \\begin{bmatrix} F(n+1) & F(n) \\\\ F(n) & F(n-1) \\\\ \\end{bmatrix} $$

$$\\therefore \\begin{bmatrix} 1 & 1 \\\\ 1 & 0 \\\\ \\end{bmatrix} ^{n} \\times = \\begin{bmatrix} F(n+1) & F(n) \\\\ F(n) & F(n-1) \\\\ \\end{bmatrix} $$

## 矩陣快速冪

```Go
type Matrix [2][2]int

func multiply(matrix1, matrix2 Matrix) Matrix {
    return Matrix{
        {
            matrix1[0][0] * matrix2[0][0] + matrix1[0][1] * matrix2[1][0],
            matrix1[0][0] * matrix2[0][1] + matrix1[0][1] * matrix2[1][1],
        },
        {
            matrix1[1][0] * matrix2[0][0] + matrix1[1][1] * matrix2[1][0],
            matrix1[1][0] * matrix2[0][1] + matrix1[1][1] * matrix2[1][1],
        },
    }
}

func fastPow(matrix Matrix, n int) Matrix {
    if n == 1 {
        return matrix
    }

    m := fastPow(matrix, n/2)
    if n % 2 == 0 {
        return multiply(m, m)
    } else {
        return multiply(multiply(m, m), matrix)
    }
}

func fib(n int) int {
    if n == 0 {
        return 0
    }
    if n == 1 {
        return 1
    }

    m := fastPow(Matrix{{1,1}, {1,0}}, n)

    return m[0][1]
}
```

{{< br >}}

# 參考資料

[快速冪 & 矩陣乘法 - HackMD](https://hackmd.io/@fdhscpp110/matix_fast_pow)