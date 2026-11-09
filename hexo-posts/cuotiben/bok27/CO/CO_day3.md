---
title: CO每日一题 Day3
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
| **涉及知识点** | Cache 直接映射、地址划分、Cache 行结构 |

**题目内容**：某计算机按字节编址，主存地址为 $32$ 位。Cache 数据区容量为 $8\text{ KB}$，采用直接映射方式，Cache 块大小为 $32\text{ B}$，并采用回写（Write Back）方式，则一个 Cache 行的总大小为（ ）。

**选项**：

- A. $275\text{ bit}$
- B. $276\text{ bit}$
- C. $277\text{ bit}$
- D. $285\text{ bit}$

<!-- more -->

## 2. 我的作答

**我的答案**：**C**

**错选**：无（本题作答正确）

## 3. 正确答案与解析

**正确答案**：**C**

**题目解析**：

<span style="display:inline-block;padding:1px 8px;border:1px solid rgba(127,127,127,.35);border-radius:999px;background:rgba(127,127,127,.08);font-size:12px;line-height:20px">deepseek/deepseek-v4-flash 精修于 2026-11-09 16:25</span>

**第一步：块内地址位数。**

Cache 块大小 $32\text{ B}=2^5\text{ B}$，按字节编址，故块内地址占

$$
\log_2 32=5\text{ 位}
$$

**第二步：Cache 行数（索引位数）。**

$$
\frac{8\text{ KB}}{32\text{ B}}
=\frac{2^{13}}{2^5}
=2^8=256\text{ 行}
$$

直接映射下，索引位数

$$
\log_2 256=8\text{ 位}
$$

**第三步：Tag 位数。**

$$
32-8-5=19\text{ 位}
$$

**第四步：数据区位数。**

$$
32\text{ B}=32\times8=256\text{ bit}
$$

**第五步：附加位。**

- 有效位（Valid）：$1$ 位；
- 回写方式需要脏位（Dirty）：$1$ 位；
- 直接映射无需替换算法位，不需要 LRU 等标志。

**第六步：Cache 行总大小。**

$$
256+19+1+1=277\text{ bit}
$$

故选 C。

**解题关键**：

> $\text{Cache 行总大小}=\text{数据位数}+\text{Tag 位数}+\text{有效位}+\text{脏位（回写时）}$。直接映射没有替换算法，不要多算替换控制位。

## 4. 知识点补充

<span style="display:inline-block;padding:1px 8px;border:1px solid rgba(127,127,127,.35);border-radius:999px;background:rgba(127,127,127,.08);font-size:12px;line-height:20px">deepseek/deepseek-v4-flash 生成于 2026-11-09 16:25</span>

### 4.1 主存地址划分

直接映射下，主存地址分为三段：

<table style="width:100%;border-collapse:collapse;text-align:center;font-size:14px">
  <tr>
    <td style="width:59.375%;border:1px solid rgba(127,127,127,.45);padding:6px">Tag（19 位）</td>
    <td style="width:25%;border:1px solid rgba(127,127,127,.45);padding:6px">Index（8 位）</td>
    <td style="width:15.625%;border:1px solid rgba(127,127,127,.45);padding:6px">Offset（5 位）</td>
  </tr>
</table>

| 字段 | 位数 | 作用 |
| :---: | :---: | :--- |
| Tag | $19$ | 与 Cache 行中保存的标记比较，判断是否命中 |
| Index | $8$ | 选择 Cache 行（直接映射） |
| Offset | $5$ | 定位块内字节 |

### 4.2 Cache 行的组成

| 字段 | 位数 | 说明 |
| :---: | :---: | :--- |
| Valid | $1$ | 该行内容是否有效 |
| Tag | $19$ | 主存块标记 |
| Data | $256$ | $32\text{ B}$ 的数据 |
| Dirty | $1$ | 回写方式下标记该行是否被修改 |
| **合计** | **$277$** | — |

### 4.3 易混淆概念

> **Cache 行（line）≠ Cache 块（block）**：Cache 块通常只指数据部分（$32\text{ B}$）；Cache 行还包括 Tag、Valid、Dirty 等附加位。题目问“行总大小”时要算附加位。

> **数据区容量 ≠ Cache 总容量**：$8\text{ KB}$ 只是数据区容量，用于求行数；Cache 实际占用的存储还包括 Tag、Valid、Dirty 等，总容量更大。

> **直接映射不需要替换算法位**：直接映射中主存块到 Cache 行的位置固定，不存在“替换哪一行”的选择，因此不需要 LRU、FIFO 等替换控制位；组相联和全相联才需要。

> **回写需要脏位，写直达不需要**：Write Back 必须记录行是否被修改，以便替换时决定是否写回主存；Write Through 每次写都同步写主存，通常不需要脏位。

> **地址位数与编址单位**：按字节编址时，块内地址位数由“块大小（字节）”决定；若按字编址，则要先用 $\text{块大小}\div\text{字长}$ 求块内字数，再取对数。
