---
title: "1411: Number of Ways to Paint N × 3 Grid"
type: post
date: 2021-06-26T13:13:50+08:00
publishdate: 2021-06-26
lastmod: 2021-06-26
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
and how a problem on dynamic programming can be solved using some combinatorics to reach a quick $O(n)$ solution.

<!--more-->

This is my first post regarding a [LeetCode](https://leetcode.com/) problem,
and I don't do write-ups of any of my solutions
but this problem is especially peculiar for me.
When I discovered this solution, I wanted to share it so badly with the rest of the audience.
I might post more solutions to LeetCode problems that I am very interested in in the future, who knows.

## Problem statement

An $n\times3$ grid is provided, where each cell is to be painted either red, green, or yellow,
and no two cells directly adjacent to each other is to be painted the same color.
How many possible "paintings" are there?

As $n$ gets larger, the number of paintings get considerably so as well.
The answer must be computed modulo $10^9+7$.

For example, a $1\times3$ grid has $12$ paintings:

{{< figure
  src="1x3b.svg"
  caption="All 12 paintings of a $1\times3$ grid" >}}

A $2\times3$ grid has $54$ paintings:

{{< figure
  src="2x3b.svg"
  caption="All 54 paintings of a $2\times3$ grid" >}}

The number of paintings get large quickly: a $7\times3$ grid has $106494$ paintings.

## Solution

### Code

This is the solution that I came up with to this quite elusive problem:

```c++
class Solution {

  static constexpr long _M{1000000007};

public:
  int numOfWays(int n) {

    // a: 121-paintings, b: 123-paintings
    long a{1}, b{1};
    for (int ii{1}; ii < n; ++ii) {
      long na{3*a+2*b}, nb{2*a+2*b};
      a = na % _M;
      b = nb % _M;
    }

    return (int)((6*(a+b))%_M);
  }
```

That's it. But, okay, let's trace the steps that I took in order to get here.

### Six possible ways to assign colors

Ultimately, there are only two types of rows in such a color scheme:

1. the first and third cell having the same color, and
2. the first and third cell having different colors.

Let us call the first one a **121-row** and the second one a **123-row**.
The numbers indicate what the colors of such a row would look like.

There are $3! = 6$ ways to assign three colors to these numbers.
Hence, a 121-row and a 123-row can have six combinations each.

{{< figure
  src="1x3-numbered.svg"
  caption="There are six possible ways to assign colors to the numbers 1, 2, and 3." >}}

### 123-paintings and 121-paintings

Let us call a painting a **123-painting** if the last row that it contains is a 123-row
and a **121-painting** if the last row it contains is a 121-row.

Now,
