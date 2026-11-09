---
title: DS每日一题 Day1
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
| **涉及知识点** | 时间复杂度分析、循环执行次数统计 |

**题目内容**：以下程序段的时间复杂度为（ ）

```c
int s = 0;
for (int i = 1; i <= n; i *= 2)
    for (int j = 0; j < i; ++j)
        ++s;
```

**选项**：

- A. $O(\log n)$
- B. $O((\log n)^2)$
- C. $O(n)$
- D. $O(n\log n)$

<!-- more -->

## 2. 我的作答

**我的答案**：**C**

**错选**：无（本题作答正确）

## 3. 正确答案与解析

**正确答案**：**C**

**题目解析**：

<span style="display:inline-block;padding:1px 8px;border:1px solid rgba(127,127,127,.35);border-radius:999px;background:rgba(127,127,127,.08);font-size:12px;line-height:20px">deepseek/deepseek-v4-flash 精修于 2026-11-09 16:25</span>

外层循环中 $i$ 依次取

$$
1,2,4,\cdots,2^k,\qquad 2^k\le n<2^{k+1}
$$

即 $k=\lfloor\log_2 n\rfloor$。第 $r$ 轮内层循环执行 $i=2^r$ 次，因此总执行次数为

$$
1+2+4+\cdots+2^k=2^{k+1}-1
$$

由 $2^k\le n$ 得 $2^{k+1}-1\le 2n-1$，故 $T(n)=O(n)$。

**易错点**：原解析写成 $1+2+4+\cdots+\log n$ 不准确。末项应是**不超过 $n$ 的最大 2 的幂** $2^k$，而不是 $\log n$；该和是等比数列，结果为 $O(n)$，不是 $O(n\log n)$。

**解题关键**：

> 循环变量 $i$ 每次翻倍，只说明**外层**有 $O(\log n)$ 层；但每层内层的工作量随 $i$ 指数增长，总工作量由最后一项 $n$ 主导，因此是 $O(n)$。
