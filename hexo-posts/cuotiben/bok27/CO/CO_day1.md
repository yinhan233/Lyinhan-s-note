---
title: CO每日一题 Day1
date: 2026-11-07 20:52:00
categories:
  - 错题本
  - bok27
  - CO
tags:
  - 408
  - 每日一题
  - BOK27
  - CO
toc: true
---

## 1. 题目

| 项目 | 内容 |
| :---: | :--- |
| **来源** | BOK27 考研每日一题 |
| **科目** | 计算机组成原理（CO） |
| **涉及知识点** | CPU 执行时间、主频、CPI |

**题目内容**：某程序在机器 M1 上运行时间为 $12\text{ s}$，M1 主频为 $3\text{ GHz}$。机器 M2 与 M1 执行该程序的指令条数相同，但平均 CPI 为 M1 的 $1.2$ 倍。若要求该程序在 M2 上运行时间为 $9\text{ s}$，则 M2 主频至少应为（ ）。

**选项**：

- A. $3.6\text{ GHz}$
- B. $4.0\text{ GHz}$
- C. $4.8\text{ GHz}$
- D. $5.4\text{ GHz}$

<!-- more -->

## 2. 我的作答

**我的答案**：**C**

**错选**：无（本题作答正确）

## 3. 正确答案与解析

**正确答案**：**C**

**题目解析**：

<span style="display:inline-block;padding:1px 8px;border:1px solid rgba(127,127,127,.35);border-radius:999px;background:rgba(127,127,127,.08);font-size:12px;line-height:20px">deepseek/deepseek-v4-flash 精修于 2026-11-09 16:25</span>

CPU 执行时间公式：

$$
T=\frac{N\cdot CPI}{f}
$$

其中 $N$ 为指令条数，$CPI$ 为平均每条指令的时钟周期数，$f$ 为主频。

M1 上该程序消耗的时钟周期数为

$$
N\cdot CPI_1=T_1f_1=12\times3\times10^9=3.6\times10^{10}
$$

M2 与 M1 指令条数相同，平均 CPI 为 $1.2CPI_1$，要求 $T_2=9\text{ s}$：

$$
T_2=\frac{N\cdot1.2CPI_1}{f_2}
=\frac{1.2\times3.6\times10^{10}}{f_2}
=9
$$

解得

$$
\begin{aligned}
f_2
&=\frac{1.2\times3.6\times10^{10}}{9}\\
&=4.8\times10^9\text{ Hz}\\
&=4.8\text{ GHz}
\end{aligned}
$$

故选 C。

**倍率速算**：执行时间变为原来的 $\dfrac{9}{12}=0.75$，CPI 变为原来的 $1.2$ 倍，则主频需变为原来的

$$
\frac{1.2}{0.75}=1.6
$$

倍，即 $3\times1.6=4.8\text{ GHz}$。用倍率法可以避免反复处理 $10^9$。

## 4. 知识点补充

<span style="display:inline-block;padding:1px 8px;border:1px solid rgba(127,127,127,.35);border-radius:999px;background:rgba(127,127,127,.08);font-size:12px;line-height:20px">deepseek/deepseek-v4-flash 精修于 2026-11-09 16:25</span>

### 4.1 核心公式

| 概念 | 公式 | 说明 |
| :---: | :--- | :--- |
| 主频 $f$ | — | 每秒时钟周期数，单位 $\text{Hz}$ |
| 时钟周期 $T_{\text{clk}}$ | $T_{\text{clk}}=\dfrac{1}{f}$ | 一个时钟周期持续的时间，单位 $\text{s}$ |
| CPI | $CPI=\dfrac{\text{时钟周期数}}{\text{指令条数}}$ | 平均每条指令所需时钟周期数 |
| CPU 执行时间 | $T=\dfrac{N\cdot CPI}{f}=N\cdot CPI\cdot T_{\text{clk}}$ | $N$ 为指令条数 |
| IPC | $IPC=\dfrac{1}{CPI}$ | 每时钟周期执行的指令数 |
| MIPS | $\text{MIPS}=\dfrac{f}{CPI\times10^6}=\dfrac{N}{T\times10^6}$ | 每秒执行多少百万条指令 |

> 若主频以 $\text{MHz}$ 为单位，则数值上 $\text{MIPS}=\dfrac{f_{\text{MHz}}}{CPI}$，不需要再除以 $10^6$。

### 4.2 易混淆概念

> **主频高 ≠ 程序一定跑得快**：执行时间还取决于指令条数 $N$ 和平均 $CPI$。主频高但 CPI 或指令条数更差时，程序可能更慢。

> **CPI 是平均值**：不同指令的 CPI 不同，程序的平均 CPI 是各指令 CPI 按出现次数（或比例）的加权平均，不能拿某一条指令的 CPI 代表整个程序。

> **MIPS 不能跨指令集直接比较**：不同机器的指令集不同，一条指令完成的工作量不同，MIPS 只适合同一指令集内粗略比较。

> **单位换算**：$1\text{ GHz}=10^9\text{ Hz}$，$1\text{ MHz}=10^6\text{ Hz}$；代入公式时主频要用 $\text{Hz}$，算 MIPS 时注意 $10^6$ 的换算。
