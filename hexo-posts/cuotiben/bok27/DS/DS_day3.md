---
title: DS每日一题 Day3
date: 2026-11-07 20:51:00
categories:
  - 错题本
  - bok27
  - DS
tags:
  - 408
  - 每日一题
  - BOK27
  - DS
toc: true
---

## 1. 题目

| 项目 | 内容 |
| :---: | :--- |
| **来源** | BOK27 考研每日一题 |
| **科目** | 数据结构（DS） |
| **涉及知识点** | 顺序表插入、平均移动次数 |

**题目内容**：长度为 $n$ 的顺序表中现有 $n$ 个元素，且存储空间足够。若在合法插入位置中，等概率地选择一个位置插入新元素，则插入操作平均需要移动（ ）个元素。

**选项**：

- A. $\dfrac{n-1}{2}$
- B. $\dfrac{n}{2}$
- C. $\dfrac{n+1}{2}$
- D. $n$

<!-- more -->

## 2. 我的作答

**我的答案**：**B**

**错选**：无（本题作答正确）

## 3. 正确答案与解析

**正确答案**：**B**

**题目解析**：

<span style="display:inline-block;padding:1px 8px;border:1px solid rgba(127,127,127,.35);border-radius:999px;background:rgba(127,127,127,.08);font-size:12px;line-height:20px">deepseek/deepseek-v4-flash 精修于 2026-11-09 16:25</span>

长度为 $n$ 的顺序表共有 $n+1$ 个合法插入位置，即“第 $1$ 个元素之前”到“第 $n$ 个元素之后”。若插入到第 $i$ 个位置（$1\le i\le n+1$），需要后移的元素个数为

$$
n-i+1
$$

各位置等概率，因此平均移动次数为

$$
\begin{aligned}
\bar m
&=\frac{1}{n+1}\sum_{i=1}^{n+1}(n-i+1)\\
&=\frac{1}{n+1}\cdot\frac{n(n+1)}{2}\\
&=\frac{n}{2}
\end{aligned}
$$

**解题关键**：

> 插入有 $n+1$ 个合法位置，平均移动 $\dfrac{n}{2}$ 个元素；删除只有 $n$ 个合法位置，平均移动 $\dfrac{n-1}{2}$ 个元素。两者不要混淆。
