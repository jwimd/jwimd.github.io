---
title: A Local Search Algorithm for LIA SMT Solving
author: jwimd
date: 2023-12-07 20:00:00 +0800
categories: [Paper Reading, SMT Solving]
tags: [Paper Reading, SMT Solving, LIA, Local Search, CAV'23]
math: true
mermaid: true
#img_path: /pictures/2023-11-10-Efficient Formal Verification for the Linux Kernel/
#image:
#  path: /vZQxpRRm/favicon.png
#  alt: 早苗
---

## Reference

The method introduced by this article is proposed by S Cai et al in [Local Search for SMT on Linear Integer Arithmetic, *CAV'23*](https://link.springer.com/chapter/10.1007/978-3-031-13188-2_12).

## Preliminary

Before we begin, you need to have some basic knowledge, like [SAT](https://en.wikipedia.org/wiki/Boolean_satisfiability_problem), [SMT](https://en.wikipedia.org/wiki/Satisfiability_modulo_theories), their solving algorithms and any other thing you need to let you understand which I may not mention it below.

We define our learning target: SMT(LIA) as:

**Definition 1.** *Linear Integer Arithmetic (LIA)*: 

1. $P=\{p_1,p_2,...,p_n|p_i\in\{\text{True},\text{False}\},i\le n \}$

   $X=\{x_1,x_2,...,x_n|x_i\in \mathbb{D},i\le n \}$

   

2. 
