---
title: 卡字符串hash
date: 2024-05-03 19:34:04
tags: [string,hash]
---
# 卡字符串hash
## 前言
某天看到同学打**字符串hash**，发现还在用**腐朽**的**模数hash**。

为了帮助他改进码风，我决定**卡炸hash**。

## 一、大模数哈希
### 1.前言
我们知道，要想卡掉 hash，本质上就是让两个字符串有相同的 hash 值，我们称为**哈希碰撞**。

假设 hash 中每个值生成概率相同，则碰撞概率取决于两个因素：
- 取值空间大小
- hash 的计算次数

这个问题在数学中有一个经典的问题，叫[生日问题](https://baike.baidu.com/item/%E7%94%9F%E6%97%A5%E6%82%96%E8%AE%BA/2715290)。

简单来说，就是在一个有 d 人的班中，有人生日相同的概率为多少。

实际上，这个问题的答案并没有想象中的小，下面是部分概率：
| 人数  |   概率   |
| :---: | :------: |
|  23   |  $50\%$  |
|  50   |  $97\%$  |
|  70   | $99.9\%$ |

### 2.计算冲突概率
我们设 hash 的取值空间为 d，计算次数为 n。
则 hash 不冲突的概率为：

$$
\overline{p}(n,d) = 1 \cdot \left (1 - \frac{1}{d} \right) \cdot \left ( 1- \frac{2}{d}\right) \cdots \left ( 1- \frac{n-1}{d}\right)
$$

经过亿点点的数学计算，他可以变为：

$$
\frac{d!}{d^n\left(d-n\right)!}
$$

则 hash 冲突概率为：

$$
p(n,d)=1-\frac{d!}{d^n\left(d-n\right)!}
$$

但是，这个太丑了，计算也太复杂了。

根据泰勒公式：

$$
\exp(x) = \sum_{k=0}^{\infty}\frac{x^k}{k!}=1+x+\frac{x^2}{2}+\frac{x^3}{6}+\frac{x^4}{24}+\cdots
$$

当 $x$ 为一个极小的值，那么 $\exp(x)\approx1+x$

将上面 hash 不冲突的原始公式带入：

$$
\overline{p}(n,d) = 1 \cdot \exp(-\frac{1}{d}) \cdot \exp(-\frac{2}{d}) \cdots \exp(-\frac{n-1}{d})
$$

化简得：

$$
\overline{p}(n,d) = \exp(-\frac{n(n-1)}{2d})
$$

则 hash 冲突的概率为：

$$
p(n,d) = 1-\exp(-\frac{n(n-1)}{2d})
$$

### 3.卡hash
言归正传，注意这个公式：

$$
p(n,d) = 1-\exp(-\frac{n(n-1)}{2d})
$$

我们只要使 d（取值空间）比模数大，并找一个 n（计算次数），使得 $p(n,d)$ 尽量为 1，就完成了。

例：

对于字符集为大小写字母及其数字，我们设模数为 $10^9+7$：

$\log_{62}10^9+7\approx6$

$p(10^6,6^{62})\approx$

所以共有 $10^6$ 个字符串，每个字符串长度为 6 就是个不错的数据。

## 二、自然溢出hash
我们知道，这种 hash 是形如 $hs = hs \cdot base + s[i]$，分情况讨论。
### 1.base 为偶数
这种简单，构造两个长度相同且大于64，并只有首字母不同的字符串，形如：

aaa...a

baa...a

### 2.base 为奇数
定义 $!s_i$ 为 $s_i$ 的倒串。

例如：
$s_i=abcda$

则：$!s_i=adcba$

设 $s_1 = a$

之后不断构造 $s_i=s_{i-1}+!s_{i-1}$

然后取 $s_{12}$ 中所有长度至多为 $2^{11}$ 的子串即可。

参考：

[毒瘤养成记1: 如何卡hash](https://www.cnblogs.com/Hs-black/p/12219270.html)

[哈希碰撞与生日攻击](https://www.ruanyifeng.com/blog/2018/09/hash-collision-and-birthday-attack.html)