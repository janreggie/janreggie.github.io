---
title: "I completed the Haskell MOOC"
type: post
date: 2020-12-25T20:07:33+08:00
publishdate: 2020-12-25
lastmod: 2020-12-25
draft: false
description: "The University of Helsinki is offering a MOOC for Haskell, which is a great way to introduce experienced programmers into functional programming."
tags:
  - "haskell"
  - "functional programming"
  - "mooc"
categories:
  - "courses"
---

The [Haskell MOOC](https://haskell.mooc.fi/) is an online course on Functional Programming using Haskell.
It is a great way to introduce experienced programmers into functional programming,
and I myself have been introduced to the paradigm properly through this course.

<!--more-->

## Prior Knowledge on Functional Programming

I have always wanted to learn this unfamiliar paradigm for some time now.
I started [Learn You a Haskell for Great Good](http://learnyouahaskell.com/chapters) maybe two years ago,
but unfortunately I abandoned reading it after two chapters at the time.
Most of the code I write is procedural with a bit of passing functions around here and there,
although I have rarely encountered maps, filters, and similar constructs.

While learning JavaScript through freeCodeCamp,
there was [an entire section dedicated to functional programming](https://www.freecodecamp.org/learn/javascript-algorithms-and-data-structures/functional-programming/),
something which I also had to use later when learning React and going through SICP.
I have been exposed to `map`, `filter`, and `reduce`,
and my first reaction when I saw these is how "smart" and clean the expressions were.
Comparing this with the clunkiness of using `for...range` loops in procedural languages.

## Course Material and Exercises

There are a total of eight lectures in the course,
each lecture having a set (or two) of exercises to test one's knowledge.
The reader is expected to follow along with the code,
using a GHCI instance to go through the examples, for instance.

For me, each lecture took around 2 to 3 hours,
although it can differ depending on how fast one can pick up the concepts
using the prior knowledge they have with FP or other paradigms.
The trouble lies in going through the exercises after that amount of time,
making following along with the code very important.

The course doesn't go through much with the theory and maths of functional programming,
and instead focuses on giving examples and showing how the syntax works.
I write that as a complement: that's an excellent way of teaching the language.

As an introduction, it doesn't really go into the nuances of the language.
It only gave a glimpse of what monads are in IO, without explaining what they really are.
Nevertheless, it presents to the reader most of what Haskell is capable of,
and how to read and write programs in a functional manner.
It doesn't go through the process of creating projects using Stack
or in creating test suites to guard correctness,
which, if the learner is interested, should use other resources to do exactly those.

## Next Steps

I've been actually eyeing on this book called "Algorithm Design with Haskell" (2020),
which I hope to get started on soon.
It is actually a sequel to "Thinking Functionally with Haskell" (2014),
although I'm debating whether it might even be worth reading the latter
and just skip to the former.
I took a peek and these books are structured much more formally than the MOOC,
and I'm excited to see what they can bring.

I don't know when the University of Helsinki will release the second part of the course.
I don't think I'll be participating then, though.
They do make [very good online courses](https://www.mooc.fi/en) for other subjects though,
and the best part is thtat they're free.

Am I actually going to use Haskell in the workplace?
Probably not, although having knowhow of functional programming is nice to have.
Just like how I started learning web dev mid-2020,
it's fun to learn new things once in a while.

My solutions for the exercises can be found [here](https://github.com/janreggie/haskell-mooc).
