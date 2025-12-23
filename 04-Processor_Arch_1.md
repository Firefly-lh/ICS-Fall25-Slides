---
theme: academic
highlighter: shiki
title: 04-Processor Arch I
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

# 04-Processor Architecture I

信息与计算科学 连昊

---

# 关于Arch Lab
- **截止时间: 11月3日晚23:59**
- 前置知识:
  - PartA: 基本的Y86-64指令书写(4.1节)
  - PartB: HCL语言和流水线(4.2-4.5节)
  - PartC: 简单的代码优化(第五章, 可以着重关注下**循环展开**)
- 思路提示
  - 相对容易的满分线: ac = 3 && cpe <= 9
  - 对于ac, `pipe_std`已经实现了ac=4的流水线, 你可以先观察使得ac=4的性能瓶颈部分, 并将其中的运算拆分到其它阶段
  - 对于cpe, 如果你已经实现了ac=3的流水线, 基本的优化便足够获取满分, 如果你希望进一步降低cpe, 你可能需要从网上的一些参考资料中学习些奇技淫巧 
  - 本lab的底层是用Rust书写, 完成时可能会遇到各种奇妙小bug和大规模报错, 请在开工前**仔细阅读writeup**, 有问题欢迎随时咨询助教

---

# 作业题

Homeworks

4.47 A. 主要关注当中的指针运算部分(回忆下**自动伸缩**这一知识点)

```c
void bubble_p(long* data, long count) {
  long *i, *last;
  for (last = data+count-1; last > data; last--) {
    for (i = data; i < last; i++) {
      if (*(i+1) < *i) {
        /* swap adjacent elements */
        long t = *(i+1);
        *(i+1) = *i;
        *i = t;
      }
    }
  }
}
```

B. 略, 在Arch Lab中你也需要自己实现一个冒泡排序的ys版本, 你可以在[这个网址](https://dreamanddead.github.io/CSAPP-3e-Solutions/chapter4/4.47/)获取参考.

---

# 作业题

Homeworks

4.48 如下实现了简单的比较交换逻辑, 注释部分标注了对应汇编代码的理解

```
L4:
  mrmovq 8(%rax), %r9    // a in %r9
  mrmovq (%rax), %r10    // b in %r10
  rrmovq %r9, %r8        // a -> %r8
  subq %r10, %r8         // a-b -> %r8 (if a<b?)
  cmovl %r9, %r11        // if a<b then a -> %r11 (tmp = a)
  cmovl %r10, %r9        // if a<b then b -> %r9 (a = b)
  cmovl %r11, %r10       // if a<b then tmp -> %r10 (b = tmp)
  rmmovq %r9, 8(%rax)
  rmmovq %r10, (%rax)
  irmovq $8, %r8
  addq %r8, %rax
  jmp L5
```

---

# 作业题

Homeworks

4.49 由于只允许使用一次条件传送, 你需要通过尽量少传送的方法交换a和b的值, 这里采用的解决方案是先求较小值, 而后利用两数之和还原最大值的方式. ([这个网站](https://dreamanddead.github.io/CSAPP-3e-Solutions/chapter4/4.49/)记录了另一个可行的方法, 用到了三异或交换的巧妙性质)

```
L4:
  mrmovq 8(%rax), %r9   // a in %r9
  mrmovq (%rax), %r10   // b in %r10
  rrmovq %r9, %r8       // a -> %r8
  rrmovq %r10, %r11     // b -> %r11 (min = b)
  subq %r10, %r8        // a-b -> %r8 (if a<b?)
  cmovl %r9, %r11       // if a<b then a -> %r11 (min = a)
  rrmovq %r9, %r12      // a -> %r12 (max)
  addq %r10, %r12   
  subq %r11, %r12       // max = a + b - min
  rmmovq %r12, 8(%rax)  // max -> data[i+1]
  rmmovq %r11, (%rax)   // min -> data[i]
  irmovq $8, %r8
  addq %r8, %rax
  jmp L5
```

---

# 作业题

Homeworks

4.51 **重点题型, 关注课本4.3.1给出的书写规范**

| phase | iaddq V,rB |
| :--: | :--: |
| Fetch(取指) | icode:ifun $\leftarrow M_1$\[PC\] <br> rA:rB $\leftarrow M_1$\[PC + 1\] <br> valC = $\leftarrow M_8$\[PC + 2\] <br> valP $\leftarrow$ PC + 10 |
| Decode(译码) | valB $\leftarrow$ R\[rB\] |
| Execute(执行) | valE $\leftarrow$ valB + valC <br> set CC |

---

# 作业题

Homeworks

4.51

| phase | iaddq V,rB |
| :--: | :--: |
| Memory(访存) | |
| Write(写回) | R\[rB\] $\leftarrow$ valE|
| PC(更新PC) |PC $\leftarrow$ valP|

---

# 重点知识纲要
- 基本概念: 指令集体系结构(ISA, 一个处理器支持的指令和指令的字节级编码)
  - 主要作用: 汇编语言和机器码的互译; 计算机软件和硬件的中继桥梁
- Y86-64 ISA(我们主要学习的ISA, X86-64的子集)
  - 基本组成: 各种状态单元, 指令集和它们的编码, 一组编程规范和异常事件处理
  - 程序员可见状态: Y86-64指令能够读取或修改的处理器状态
    - 15个程序寄存器(**没有%r15**) + 3个条件码(**没有CF**) + PC(程序计数器) + Stat(状态码) + DMEM(内存)
  - Y86-64指令及其编码 (课本4.1.2和4.1.3节, 重点掌握)
    - 小细节
      - 在内存传送指令`xxmovq`中, 内存引用只能使用基址加立即数偏移量的形式, 没有第二变址寄存器
      - Y86-64指令集只支持对8字节立即数进行操作, 所有指令编码的常数值均为8字节, 且**采用小端法编码**
      - `rrmovq`和条件传送共用同样的指令代码, 可将其视为"无条件传送"
      - 注意课本P247图4-4的**寄存器指示符字节和寄存器的对应顺序**, 没有%r15后我们余出`0xF`用于表示无寄存器
      - 分支指令和调用指令的目的都是**绝对地址编码**, 而不是PC-relative编码

---

# 重点知识纲要

- 关注课本P248和P249的两个旁注 ~~(陆老师很爱考RISC和CISC的比较)~~
- 重要题型: Y86-64 汇编指令和字节码的互译
  - 字节码 -> 汇编指令: 从前往后读, 先根据指令编码确定指令长度, 而后相应分段, 逐段翻译
- Y86-64状态: 主要掌握课本P251 图4-5中四个状态码对应的名字和含义, 具体的异常处理流会在第8章讨论
- 小细节(4.1.6节): Y86-64在执行`pushq %rsp`时, 压入的是`%rsp`的旧值, 执行`popq %rsp`时, 弹出的是`M[%rsp]`的值, 等价于`mrmovq (%rsp), %rsp`
- 硬件控制语言HCL **(要求掌握基本的HCL书写)**
  - 逻辑门: 掌握不同的逻辑门和不同运算方式的对应关系, 注意**逻辑门只对单个位的数(0,1)进行操作**
  - 组合电路(注意课本P257对组合电路构建的限制), 理解课本P258图4-10和4-11对应的组合电路如何运作
    - 区分组合逻辑电路和C语言逻辑表达式: 输出持续响应输入变化; 只针对位值0/1运算; 没有短路求值规则
    - 注意多路复用器和HCL中情况表达式的对应关系
      - 情况表达式顺序求值, 第一个求值为1的情况会被选中, 因此不要求不同的选择表达式互斥
    - 关注课本P261图4-15 ALU的结构(输入和输出)
  - 字级电路: 逐位实现, 抽象为字级

---

# 重点知识纲要

- 存储器和时钟
  - 时钟: 周期性的信号, 控制何时将新值加载的存储设备中
  - 时钟寄存器/寄存器/硬件寄存器 **(注意和软件层面上寄存器的区分)**
    - 在时钟周期的上升沿将输出更新为输入, 用于在组合电路的不同部分间传输信号
    - 一般用于存储PC, CC, Stat等程序员可见状态
  - 随机访问寄存器/内存 **(这才涵盖了软件层面上的寄存器)**
    - 寄存器文件/程序寄存器: 两个读端口 + 一个写端口, 允许在同时对寄存器文件进行读写

---

# 重点知识纲要

- Y86-64的顺序实现(SEQ)
  - **注: 这部分是理解Pipeline的基础, 很重要, 我这里只列举了部分重点和细节, 推荐大家结合A神的课件回到课本进行复习**
  - 指令处理的六个阶段: 取指(Fetch) -> 译码(Decode) -> 执行(Execute) -> 访存(Memory) -> 写回(Write back) -> 更新PC(PC update)
    - 取指: 根据PC从对应的内存地址中读出完整指令(icode, ifun, rA, rB, valC), 并根据指令长度计算valP(用于最后更新PC)
    - 译码: 根据取指阶段读出的rA和rB(如有), 从寄存器文件中读出valA和valB
    - 执行: 在需要计算的情况(`opq` `rmmovq` `mrmovq` `pushq` `popq`), ALU将计算得到valE, 同时设置条件码(`set CC`)并决定是否需要选择分支(`Cnd <- Cond(CC, ifun)`)
    - 访存: 将数据写入内存(`rmmovq` `pushq` `call`) 或从内存读取数据(`mrmovq` `popq`), 读出的值为valM
    - 写回: 将至多两个结果写入寄存器文件
    - 更新PC: 将PC更新为下一条指令的地址(根据指令的不同, 更新的值也不同)

---

# 重点知识纲要

- 关于指令执行的六个阶段, 你需要重点掌握如下内容:
  - 各个指令在各个阶段都进行了哪些计算(课本4.3.1节详细列出了大部分指令的书写方式作为参考)
  - 基本了解SEQ对应的硬件结构(课本4.3.2节)
    - 重点关注各个阶段用到的硬件, 值的传递方式, **控制信号(名字和作用)**
    - 细节: PC是SEQ中唯一的时钟寄存器
  - SEQ的时序(简单了解)
    - 在每个时钟周期时, PC, Cond, DMEM和寄存器文件都执行什么操作?(课本P275 第二段)
    - **原则: 从不回读**
  - SEQ各阶段的HCL代码(可以参考A神的slides)

---
layout: image-left
image: /04-Processor Arch I/Q1.png
backgroundSize: 80%
---

出处: 2024期末 大题T1(1)

<div class="text-sm" v-click>

1. $M_1$ \[PC + 1\]
2. valP $\leftarrow$ PC + 2
3. valE $\leftarrow$ (-1) + valB 或 valE $\leftarrow$ valB + (-1) <br> **(重点: 没有 valB - 1)**
4. if(Cnd) R\[ rB \] $\leftarrow$ valE

</div>

---
layout: image-left
image: /04-Processor Arch I/Q2.png
backgroundSize: 80%
---

出处: 2024期中 选择题T11

<div class="text-sm" v-click>

1. 第一空: 这四个指令访问的内存地址在Execute阶段通过偏移量计算得到, 故为`valE`
2. 第二空: 这两个指令访问的内存地址直接存在寄存器中, 在Decode阶段读取, 故为`valA`
3. 第三空: 这两个指令将数据写入内存, 被写入的数据存在寄存器中, 在Decode阶段读取, 故为`valA`
4. 第四空: `call`指令将返回地址(即当前代码地址)写入内存, 故在`valP`
   
<br>
<font color="Red">本题选D.</font>

</div>

---
layout: image-left
image: /04-Processor Arch I/Q3.png
backgroundSize: 80%
---

出处: 2022期中 选择题T11

<div class="text-sm" v-click>

没啥好说的, 看书即可. 

大体记忆: RISC简单(Reduced), CISC复杂(Complex)

<br>
<font color="Red">本题选B.</font>

</div>

---
layout: image-left
image: /04-Processor Arch I/Q4.png
backgroundSize: 80%
---

出处: 2022期中 选择题T12

<div class="text-sm" v-click>
同样没啥好说的, 看书即可, 关注三个逻辑门的形状. 

<br>
<font color="Red">本题选D.</font>

</div>

---
layout: end
---

# Thanks for listening!