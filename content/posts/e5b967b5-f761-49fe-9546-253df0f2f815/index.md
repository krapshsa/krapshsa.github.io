---
title: "用快速冪與矩陣乘法解費氏數列"
date: "2023-09-28T02:58:48Z"
tags: [leetcode]
---

# 用快速冪與矩陣乘法解費氏數列

# 快速冪

可以試著用快速冪解這題：

[LeetCode - The World's Leading Online Programming Learning Platform](https://leetcode.com/problems/powx-n/description/)

## 方法一：Top-Down

以 $X^{13}$ 為例，我可以：

1. 拆成 $X^6 \times X^6 \times X$
2. $X^6$ 可以再拆成 $X^3 \times X^3$
3. $X^3$ 可以再拆成 $X \times X \times X$

用遞迴的方式做，已經拆過的另外半邊就可以被剪枝，來達到降低運算次數。

```C
double fPow(double x, int64_t n) {
    if (n == 0) {
        return 1.0;
    }
    if (n == 1) {
        return x;
    }

    double f = fPow(x, n/2);
    if (n % 2 == 1) {
        return x * f * f;
    } else {
        return f * f;
    }
}

double myPow(double x, int n) {
    if (n == 0) {
        return 1.0;
    }
    if (x == 1.0) {
        return 1.0;
    }

    int64_t ln = (int64_t)n;
    if (ln < 0) {
        return 1.0/fPow(x, -ln);
    } else {
        return fPow(x, ln);
    }
}
```

## 方法二：Bottom-Up

以 $X^{13}$ 為例，我可以：

1. 拆解 $13_{10} = 1101_{2}$ 也就是說 $X^{13}$ 是 $1 \cdot X^{2^3} \times 1 \cdot X^{2^2} \times 0 \cdot X^{2^1} \times 1 \cdot X^{2^0}$
2. 設定一個變數 base，用來代表迭代過程中的 $X^{2^m}$，初始值為 $X$
3. 設定 `result` 為 `1` ，接下來我們要用到上面的拆解，決定要不要乘上 base，決定的方法是看該位數是否為 `1` ，接著只要不斷的 shift 就可以取到下一位。
4. base 為 $X^{2^0}$，`result` 乘以 base
5. base 翻倍為 $X^{2^1}$，但不需要乘
6. base 翻倍為 $X^{2^2}$，`result` 乘以 base
7. base 翻倍為 $X^{2^3}$，`result` 乘以 base

```C
double myPow(double x, int n) {
    if (n == 0) {
        return 1.0;
    }

    int64_t absN = (n < 0) ? -(int64_t)n : n;

    double result = 1.0;
    double base   = x;

    while (absN > 0) {
        if (absN & 1) {
            result *= base;
        }
        base *= base;
        absN >>= 1;
    }

    return (n < 0) ? 1.0 / result : result;
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