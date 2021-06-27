---
title: "1411: Number of Ways to Paint N × 3 Grid"
type: post
date: 2021-06-27T20:51:50+08:00
publishdate: 2021-06-27
lastmod: 2021-06-27
draft: false
math: true
description: "This post details my solution on Leetcode problem 1411, and how combinatorics can be used to"
tags:
  - "combinatorics"
  - "leetcode-hard"
categories:
  - "leetcode"
---

In this post, I will talk about my solution to Problem #1411 in LeetCode,
on the [number of ways to paint an N×3 grid](https://leetcode.com/problems/number-of-ways-to-paint-n-3-grid/),
and how a problem on dynamic programming can be solved using some combinatorics to reach a very quick 0 ms solution in linear time.

<!--more-->

This is my first post regarding a [LeetCode](https://leetcode.com/) problem,
and I don't do write-ups of any of my solutions
but this problem is especially peculiar for me.
When I discovered this solution, I wanted to share it so badly with the rest of the audience.
I might post more solutions to LeetCode problems that I am very interested in in the future.

## Problem statement

An $n\times3$ grid is provided, where each cell is to be painted either red, green, or yellow,
and no two cells directly adjacent to each other is to be painted the same color.
How many possible "paintings" are there?

As $n$ gets larger, the number of paintings get considerably so.
The answer must be computed modulo $10^9+7$.

For example, a $1\times3$ grid has $12$ paintings:

{{< figure
  src="1x3b.svg"
  caption="All 12 paintings of a $1\times3$ grid" >}}

A $2\times3$ grid has $54$ paintings:

{{< figure
  src="2x3b.svg"
  caption="All 54 paintings of a $2\times3$ grid" >}}

The number of paintings get very large quickly: a $7\times3$ grid has $106494$ paintings.

## Solution

### Six possible ways to assign colors

Ultimately, there are only two types of rows in such a color scheme:

1. the first and third cell having the same color, and
2. the first and third cell having different colors.

Let us call the first one a **121-row** and the second one a **123-row**.
The numbers indicate what the colors of such a row would look like.

{{< figure
  src="1x3-numberings.svg"
  caption="A 121-row and a 123-row, and the possible colors that can be assigned to them." >}}

At $n=1$, we would have one 121-row and one 123-row,
which would lead to a total of $(1+1)\times6 = 12$ paintings.

### 123-numberings and 121-numberings

From here on, let us use numbers instead of colors in describing a painting,
and let us assign the colors red, green, and yellow to the numbers 1, 2, and 3 respectively.
Let us call a **numbering** this representation of numbers on a painting,
where $x$ numberings yield $6x$ paintings.

Let us call a numbering a **123-numbering** if the last row that it contains is a 123-row
and a **121-numbering** if the last row it contains is a 121-row.
Remember that a "123-row" and a "121-row" does *not* mean that their digits are such,
rather they represent the relationship between the colors of the left and right cells.

{{< figure
  src="123-121-numberings.svg"
  caption="123-numberings and 121-numberings for $n=1$ and $n=2$." >}}

At $n=1$, there is only one 123-numbering and one 121-numbering.
At $n=2$, there are four 123-numberings (first row) and five 121-numberings (second row).
We also observe that, by extending a single row,
a 123-numbering will generate *two* 123-numberings and *two* 121-numberings,
whereas a 121-numbering will generate *two* 123-numberings and *three* 121-numberings.

Using the observation above,
we can say that, at $n=3$, we would get $4\times2+5\times2=18$ 123-numberings
and $4\times2+5\times3=23$ 121-numberings.

{{< figure
  src="3x3-numberings.svg"
  caption=`All numberings for $n=2$ and $n=3$.
           At $n=2$, there are four 123-numberings and five 121-numberings,
           and they generate 16 and 25 numberings listed in the upper and lower parts of the image.
           We can count that there are indeed $18$ 123-numberings (purple background) and $23$ 121-numberings (pink background).` >}}

Suppose that at some $n=i$, let $a_i$ be the number of 123-numberings and $b_i$ be the number of 121-numberings.
From there, we can say the following:

$$
\begin{align}
  a_i &= {2a_{i-1} + 2b_{i-1}}\\\\\\
  b_i &= 2a_{i-1} + 3b_{i-1}\\\\\\
  a_1 &= 1\\\\\\
  b_1 &= 1
\end{align}
$$

At a specific $n$, the total number of numberings would be $a_n+b_n$, and the total number of *paintings* will be $6\(a_n+b_n\)$.

### Code

All of the above work led me to finding this solution:

```c++
int numOfWays(int n) {

    constexpr long _M{1000000007};
    long a{1}, b{1}; // a: 123, b: 121
    for (int ii{1}; ii < n; ++ii) {
        long na{2*a+2*b}, nb{2*a+3*b};
        a = na % _M;
        b = nb % _M;
    }

    return (int)((6*(a+b))%_M);
}
```

This took me an *hour* to think through.
And I thought I needed to do some wild dynamic programming.

### Improving runtime to logarithmic time

The above solution runs at linear time. This can be further improved.

Let us consider the equations above at $n=n$, but frame it using matrices:

$$
\begin{align}
\begin{bmatrix}
  a_n\\\\\\
  b_n\end{bmatrix}
&=\begin{bmatrix}
    2 & 2\\\\\\
    2 & 3
    \end{bmatrix}
\times
  \begin{bmatrix}
    a_{n-1}\\\\\\
    b_{n-1}\end{bmatrix}
\\\\\\
&= {\begin{bmatrix}
    2 & 2\\\\\\
    2 & 3
    \end{bmatrix}}^2
  \times
  \begin{bmatrix}
    a_{n-2}\\\\\\
    b_{n-2}\end{bmatrix}
\\\\\\
& \vdots
\\\\\\
&= {\begin{bmatrix}
    2 & 2\\\\\\
    2 & 3
    \end{bmatrix}}^{n-1}
  \times
  \begin{bmatrix}
    a_{1}\\\\\\
    b_{1}\end{bmatrix}
\\\\\\
\begin{bmatrix}
  a_n\\\\\\
  b_n\end{bmatrix}
&= {\begin{bmatrix}
    2 & 2\\\\\\
    2 & 3
    \end{bmatrix}}^{n-1}
  \times
  \begin{bmatrix}
    1\\\\\\
    1\end{bmatrix}
\end{align}
$$

The power of the square matrix above can be done in $O(\log n)$ time by considering the following:

$$
{\begin{bmatrix}
  w & x\\\\\\
  y & z\end{bmatrix}}^n
=\begin{cases}
\left(
    {\begin{bmatrix}
    w & x\\\\\\
    y & z\end{bmatrix}}^2
  \right)^{\frac{n}{2}} &\text{if }n\text{ is even}\\\\\\
\begin{bmatrix}
    w & x\\\\\\
    y & z\end{bmatrix}
  \times
  {\begin{bmatrix}
    w & x\\\\\\
    y & z\end{bmatrix}}^{n-1} &\text{if }n\text{ is odd}
\end{cases}
$$

Using these computations, we can solve this problem using matrices:

```c++
class Solution {
    static constexpr long _M{1000000007};

public:

    int numOfWays(int n) {
        if (n == 1) { return 12; }
        long matrix[4]{2,2,2,3};
        exp(matrix, n-1);
        return (int)((6*(matrix[0]+matrix[1]+matrix[2]+matrix[3]))%_M);
    }
    
private:

    // A matrix `long m[4]` is represented as {w,x,y,z}
    void exp(long matrix[4], int n) {

        if (n == 1) { return; }
        if (n % 2 == 0) {
            sq(matrix);
            exp(matrix, n/2);
            return;
        }

        long m[4]; // copy of the matrix
        memcpy(m, matrix, sizeof m);
        exp(matrix, n-1);
        mul(m, matrix);
    }

    // mul multiplies a to b and saves the result in b
    void mul(long a[4], long b[4]) {
        long c[4];
        c[0] = (a[0]*b[0] + a[1]*b[2]) % _M;
        c[1] = (a[0]*b[1] + a[1]*b[3]) % _M;
        c[2] = (a[2]*b[0] + a[3]*b[2]) % _M;
        c[3] = (a[2]*b[1] + a[3]*b[3]) % _M;
        memcpy(b, c, sizeof c);
    }

    // sq squares a matrix in-place
    void sq(long a[4]) {
        mul(a,a);
    }
};
```

Would this be considered overkill, though?
Yes.
LeetCode reports the runtime of the linear-time solution to be 0 ms already,
so making the solution run in logarithmic time will do no benefit.
Nonetheless, it's a good exercise.
