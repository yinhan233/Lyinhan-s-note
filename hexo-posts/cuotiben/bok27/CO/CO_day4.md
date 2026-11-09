---
title: CO每日一题 Day4
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
| **涉及知识点** | IEEE 754 单精度、非规格化数、小端存储 |

**题目内容**：某计算机按字节编址，采用小端方式存储数据，`float` 型数据采用 IEEE 754 单精度格式。已知变量 $x$ 的值为 IEEE 754 单精度格式能表达的最大非规格化数。若 $x$ 从地址 $1000\text{H}$ 开始存放，则地址 $1002\text{H}$ 中存放的内容为（ ）。

**选项**：

- A. $00\text{H}$
- B. $7\text{FH}$
- C. $\text{FFH}$
- D. $80\text{H}$

<!-- more -->

## 2. 我的作答

<div style="margin:0 0 12px;padding:8px 14px;border-left:4px solid #e53935;border-radius:6px;background:rgba(229,57,53,.08);color:#e53935;font-weight:700">✘ 本题作答错误</div>

**我的答案**：<span style="color:#e53935;font-weight:700">C</span>

**错选**：<span style="color:#e53935;font-weight:700">C</span>

## 3. 正确答案与解析

**正确答案**：**B**

**题目解析**：

<span style="display:inline-block;padding:1px 8px;border:1px solid rgba(127,127,127,.35);border-radius:999px;background:rgba(127,127,127,.08);font-size:12px;line-height:20px">deepseek/deepseek-v4-flash 生成于 2026-11-09 20:35</span>

IEEE 754 单精度共 $32$ 位：

| 位域 | 符号位 | 阶码 | 尾数 |
| :---: | :---: | :---: | :---: |
| 位数 | $1$ | $8$ | $23$ |

非规格化数（subnormal）的阶码全为 $0$，且尾数不全为 $0$。最大非规格化数取：

- 符号位 $S=0$（正数）；
- 阶码 $E=0000\,0000$；
- 尾数 $M=111\cdots111$（$23$ 个 $1$）。

位模式为

$$
0\ 0000\,0000\ 111\,1111\,1111\,1111\,1111\,1111_2
=\text{0x007FFFFF}
$$

其真值为

$$
(1-2^{-23})\times2^{-126}
$$

按**小端方式**存储时，低地址存低字节，因此从地址 $1000\text{H}$ 开始：

| 地址 | $1000\text{H}$ | $1001\text{H}$ | $1002\text{H}$ | $1003\text{H}$ |
| :---: | :---: | :---: | :---: | :---: |
| 内容 | $\text{FFH}$ | $\text{FFH}$ | **$\text{7FH}$** | $00\text{H}$ |

所以地址 $1002\text{H}$ 中存放的是 $\text{7FH}$，选 B。

**错因分析**：选项 C 的 $\text{FFH}$ 出现在 $1000\text{H}$ 和 $1001\text{H}$，不是 $1002\text{H}$。若把存储方式误当成大端，$1002\text{H}$ 才会是 $\text{FFH}$；本题明确是小端，因此第三字节是 $\text{7FH}$。

**图解**：

<span style="display:inline-block;padding:1px 8px;border:1px solid rgba(127,127,127,.35);border-radius:999px;background:rgba(127,127,127,.08);font-size:12px;line-height:20px">deepseek/deepseek-v4-flash 生成于 2026-11-09 20:35</span>

| 步骤 | 结果 |
| :--- | :--- |
| 最大非规格化数位模式 | $0\ 0000\,0000\ 111\cdots111_2$ |
| 32 位十六进制 | $\text{0x007FFFFF}$ |
| 小端字节序列（低 → 高） | $\text{FFH},\ \text{FFH},\ \text{7FH},\ 00\text{H}$ |
| 地址 $1002\text{H}$ | **$\text{7FH}$** |

**解题关键**：

> 非规格化数看“阶码全 0、尾数非 0”；最大非规格化数让尾数全 1。写完 `0x007FFFFF` 后，小端按“低字节在低地址”展开。

## 4. 知识点补充

<span style="display:inline-block;padding:1px 8px;border:1px solid rgba(127,127,127,.35);border-radius:999px;background:rgba(127,127,127,.08);font-size:12px;line-height:20px">deepseek/deepseek-v4-flash 生成于 2026-11-09 20:35</span>

### 4.1 规格化数与非规格化数

| 类型 | 阶码 $E$ | 尾数 $M$ | 真值 |
| :--- | :---: | :---: | :--- |
| 规格化数 | $1\sim254$ | 任意 | $(-1)^S\times(1.M)\times2^{E-127}$ |
| 非规格化数 | $0$ | 非 $0$ | $(-1)^S\times(0.M)\times2^{-126}$ |
| 零 | $0$ | $0$ | $0$ |

- 最小正非规格化数：$2^{-149}$，位模式 `0x00000001`。
- 最大正非规格化数：$(1-2^{-23})\times2^{-126}$，位模式 `0x007FFFFF`。
- 非规格化数的阶码固定为 $-126$，没有隐含的整数位 $1$。

### 4.2 小端与大端

| 方式 | 低地址存放 | 本例 `0x007FFFFF` 的字节序列 |
| :--- | :--- | :--- |
| 小端 | 最低有效字节 | $\text{FF},\text{FF},\text{7F},00$ |
| 大端 | 最高有效字节 | $00,\text{7F},\text{FF},\text{FF}$ |

小端是“低字节放低地址”，所以 $1000\text{H}$ 是 $\text{FFH}$，$1002\text{H}$ 是 $\text{7FH}$。

### 4.3 易混淆概念

> **非规格化数 vs 规格化数**：非规格化数阶码全 $0$，尾数没有隐含的 $1$；规格化数阶码非 $0$，尾数有隐含的 $1$。不要把“最大非规格化数”当成“最大规格化数”。

> **小端 vs 大端**：小端低地址放低字节，大端低地址放高字节。本题若按大端展开，$1002\text{H}$ 会是 $\text{FFH}$，正好是干扰项 C。

> **$\text{FFH}$ 的位置**：`0x007FFFFF` 的低两个字节都是 $\text{FFH}$，所以 $1000\text{H}$、$1001\text{H}$ 都是 $\text{FFH}$；$1002\text{H}$ 是第三个字节 $\text{7FH}$，不要看到两个 $\text{FF}$ 就选 C。

> **最大非规格化数 vs 最小非规格化数**：前者尾数全 $1$，后者尾数最低位为 $1$（`0x00000001`）；两者阶码都为 $0$。
