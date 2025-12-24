---
theme: academic
highlighter: shiki
title: 13-Concurrent Programming
info: |
  ICS 2025 Fall Slides
  Inspired by Arthals
  Presented by Firefly
presenter: false
titleTemplate: '%s'
class: text-center
drawings:
  persist: false
transition: fade-out
mdc: true
layout: cover
---

# 13-Concurrent Programming

信息与计算科学 连昊

---

# 最后提醒
- Proxy Lab截止时间: **12月25日(明天)晚24:00**
- 期末考试时间: **12月29日(下周一)**
  - 把握好最后几天, 自行充分复习

---

# 作业题

Homeworks

12.17 A. `main`函数在创建线程后直接调用了`exit`函数, 这导致创建的所有线程都被终止

12.18 `不安全` `安全` `不安全` (直接模拟画图即可)

12.19 刚刚课上讲过了, 你可以在Arthals的课件中找到代码实现

12.28 都不会导致死锁 (同样的自行画图即可)

12.30 

A. `线程1: a&b 和 a&c` `线程2: b&c` `线程3: a&b`

B. **互斥锁加锁顺序规则: 给定所有互斥操作的一个全序, 如果每个线程都按一个顺序(一般是递增)获得该互斥锁, 并按相反的顺序释放, 则不会导致死锁**. 因此`线程2`和`线程3`违反了规则

C. 交换线程2的`P(c)`和`P(b)`, `V(c)`和`V(b)`, 同时交换线程3的`P(a)`和`P(b)`, `V(a)`和`V(b)`

---
layout: end
---

# Thanks for listening!