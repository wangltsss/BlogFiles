---
title: Latex技巧总结(更新中)
date: 2022-01-21 21:15:59
categories: Dictionaries
tags:
- LaTeX
mathjax: true
---

# 前言
这篇文章主要记录一些用LaTeX写作时常用的环境以及数学公式，一来方便自己需要时查阅，二来也给有需要的人提供一点便利。

**注意**，这篇文章假设你了解LaTeX的基本知识，所以没有LaTeX的详细教学。
<!--more-->

# 文本类

## Description环境

**大象**&nbsp;&nbsp;&nbsp;&nbsp;一种动物。</br>

```
\begin{description}
	\item{大象} 一种动物。
\end{description}
```

---

## 列表

### 有序列表

1. 第一项
2. 第二项
3. 第三项

```
\begin{enumerate}
	\item[] 第一项
	\item[] 第二项
	\item[] 第三项
\end{enumerate}
```

### 无序列表
形如:</br>

* subsubsub
* supsupsup
* puspuspus

```
\begin{itemize}
	\item[] subsubsub
	\item[] supsupsup
	\item[] puspuspus
\end{itemize}
```
&nbsp;&nbsp;&nbsp;&nbsp;[]内可指定自定义行符。

---

# 数学类

## Arithmetic Operations

| Code | 效果 |
| :----: | :----: |
| `\sqrt[5]{x}` | $\sqrt[5]{x}$ |
| `\lvert x \rvert` | $\lvert x \rvert$ |
| `+` | $+$ |
| `-` | $-$ |
| `\times` | $\times$ |
| `\div` | $\div$ |

## 角标

| Code | 效果 |
| :----: | :----: |
| `_aX` | $_aX$ |
| `^bX` | $^bX$ |
| `X^c` | $X^c$ |
| `X_d` | $X_d$ |

## 划线

| Code | 效果 |
| :----: | :----: |
| `\overline{abcddd}` | $\overline{abcddd}$ |
| `\underline{abcddd}` | $\underline{abcddd}$ |


## 微积分

### 极限

| Code | 效果 |
| :----: | :----: |
| `\lim_{n \to \infty}n` | $\lim_{n \to \infty}n$ |

### 微分

| Code | 效果 |
| :----: | :----: |
| `\mathrm{d}x` | $\mathrm{d}x$ |
| `\frac{\mathrm{d}y}{\mathrm{d}x}` | $\frac{\mathrm{d}y}{\mathrm{d}x}$ |
| `\partial{x}` | $\partial{x}$ |
| `f‘(x)` | $f’(x)$ |

### 级数

| Code | 效果 |
| :----: | :----: |
| `\sum_{k=1}^N k^2` | $\sum_{k=0}^N k$ |


### 积分

| Code | 效果 |
| :----: | :----: |
| `\int_{-\infty}^{\infty}` | $\int_{-\infty}^{\infty}$ |
| `\iint_{-\infty}^{\infty}` | $\iint_{-\infty}^{\infty}$ |
| `\iiint_{-\infty}^{\infty}` | $\iiint_{-\infty}^{\infty}$ |



































