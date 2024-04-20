---
title: operator浅谈
date: 2023-10-04 16:54:43
tags: [skill,c++]
mathjax: true
---

# operator浅谈

## 前言

$operator$，即c++的重载运算符。

可以理解为定义了一个符号的运算规则，用来更方便处理<u>高精度</u>等特定运算。

## 使用须知

$=$赋值运算符、$[]$下标运算符、$()$函数调用运算符、$->$成员访问运算符。

**只能作为类成员重载，不能作为友元函数。**

还有，重载后无优先级，可以打括号。

Finally，你不应该重载 $\&\&\;\|\|\;\&$ 运算符，他们在程序中有重要的作用。

## 使用

你应该把他定义在结构体里，他不会对结构体外的运算符产生影响。

```c++
{结构体名} operator {运算符}({运算符}){ 
{重载规则}
}   
```

## 例题

单讲不好讲，上一道例题：高精度a+b

```c++
#include <bits/stdc++.h>
using namespace std;
struct high {
  int len, o[1005];
  void clear() {
    len = 0;
    memset(o, 0, sizeof(o));
  }
  void in() {
    char ch[1005];
    scanf("%s", ch), len = strlen(ch);
    for (int i = 0; i < len; i++)
      o[len - i] = (ch[i] - '0');
  }
  void print() {
    for (int i = len; i >= 1; i--)
      printf("%d", o[i]);
  }
};
high operator+(high a, high b) {
  high c;
  c.clear();
  while (c.len <= a.len || c.len <= b.len) {
    c.len++;
    c.o[c.len] += (a.o[c.len] + b.o[c.len]);
    c.o[c.len + 1] += (c.o[c.len] / 10);
    c.o[c.len] %= 10;
  }
  while (!c.o[c.len] && c.len > 1)
    c.len--;
  return c;
}
int main() {
  high x, y;
  x.in(), y.in();
  high z = x + y;
  z.print();
}
```

应该好理解。


