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

We define our learning target: *Linear Integer Arithmetic* as:

**Definition 1.** *propositional atomic LIA formula*: 

$p\in P=\{p_1,p_2,...,p_n\mid p_i\in\{\text{True},\text{False}\},i\le n \}$

**Definition 2.** *arithmetic atomic LIA formula*: 

$\sum_ia_ix_i\bowtie k\ \\ \text{where} \ x_i\in X=\{x_1,x_2,...,x_n\mid x_i\in \mathbb{D},i\le n \},\\ \bowtie \in\{=,\le\},\\ k,a_i \in \mathbb{Q}$

**Definition 3.** *LIA formula*: 

$x\mid x\and y\mid x\or y \mid \lnot x\\ \text{where}\ x,y \in P \cup X$
