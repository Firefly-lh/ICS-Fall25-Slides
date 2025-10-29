---
theme: academic
highlighter: shiki
title: 05-Processor Arch II
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

# 05-Processor Architecture II

信息与计算科学 连昊

---

# 作业题

Homeworks

4.57 这个改良版流水线实现了对`valA`的加载转发, 因此相较于图4-64给出的HCL公式, 当我们只需要在访存阶段用到`valA`时, 无需进行`stall`, 因此结果如下:

```
E_icode in { IMRMOVQ, IPOPQ } &&
(
  E_dstM == d_srcB ||
  (
    E_dstM == d_srcA && !(D_icode in { IRMMOVQ, IPUSHQ })
  )
);
```

---

# 作业题

Homeworks

6.23 具体计算公式见课本P410.

$T_{avg\ rotation} = \frac{1}{2} \times \frac{60s}{15000RPM} \times 1000ms/s = 2ms$

$T_{avg\ transfer} = \frac{60s}{15000RPM} \times \frac{1扇区}{800磁道} \times 1000ms/s = 0.005ms$

$T_{access} = T_{avg\ seek} + T_{avg\ rotation} + T_{avg\ transfer} = 6.005ms$

---

# 作业题

Homeworks

6.24 

$T_{avg\ rotation} = \frac{1}{2} \times \frac{60s}{15000RPM} \times 1000ms/s = 2ms$

存储一个2MB的文件需要的扇区数: $2MB / 512B = 4000$, 进一步需要的磁道数: $4000 / 1000 = 4$

于是最优情况, 这个文件恰好按顺序分布在4个完整的磁道上:
$$T = T_{avg\ seek} + T_{avg\ rotation} + T_{max\ rotation} \times 4 = 22ms$$

最差情况, 我们每读取一个逻辑块就要寻址一次:
$$T = (T_{avg\ seek} + T_{avg\ rotation}) \times 4000 = 24s$$

---

# 重点知识纲要

- 基础概念: 流水线(分阶段并行完成任务 提高整体效率)
  - 两个基本指标: 吞吐量(单位GIPS), 延迟(单位ps)
    - **一般来说**, 增加流水线阶段数会同时增加延迟(寄存器导致开销)和吞吐量(能并行处理更多指令)
  - 流水线的操作:由时钟周期控制, 每个时钟周期的上升沿更新流水线寄存器
  - 流水线的局限性
    - 不一致的划分: 运行时钟的速率是由最慢的阶段的延迟限制的
    - 流水线过深, 收益反而下降: 寄存器延迟制约流水线吞吐量的提升
    - 处理指令可能相关, 不能独立并行: 数据冒险和控制冒险

---

# 重点知识纲要

- SEQ+
  - **唯一的更改: 更新PC的操作在一个时钟周期开始时执行, 而不是结束时才执行**
    - 根据前一周期产生的控制信号, 动态的计算当前周期的PC <br> (课本P289 图4-39, 思考下`valC` `valM` `valP`分别在哪些指令用到?)
    - 无需硬件寄存器来存放PC
  - 从SEQ到SEQ+的改进称为**电路重定时**, 通常用于平衡流水线系统中各个阶段的延迟

---

# 重点知识纲要

- PIPE-
  - **关键更改: 引入流水线寄存器, 真正能够开始并行不同阶段**
  - 信号的命名
    - 大写前缀对应**流水线寄存器**, 用于存储对应阶段需要用到的状态码字段 **(在对应阶段开始前完成更新)**
    - 小写前缀对应**流水线阶段**, 指的是相应阶段的控制逻辑块产生的状态信号
    - 注意分隔符都是下划线`_`, 和arch lab中略有不同
  - 两个重要模块
    - `Select A`: 不存在同时需要用到`valA`和`valP`的指令, 因此将其合并为`valA`信号存储, 减少流水线寄存器状态数
    - `Select PC`: 分支预测, 用于猜测下一个将要执行的分支, 并提前进行取指, 在PIPE-中我们**总是选择分支**

---

# 重点知识纲要

- 初步认识分支预测
  - 为什么需要分支预测
    - 对于条件转移指令`jxx`, 需要到执行阶段计算完`cnd`后才直到是否需要跳转
    - 对于返回指令`ret`, 需要到访存指令读取完栈顶的值后才能确定返回地址
      - 进一步, 在写回阶段完成后我们才能预测`ret`指令的新PC值
    - 最直接的办法: 对于这些指令, 阻塞流水线到相应阶段完成后再取指下一条指令, **但是严重降低了效率**
      - 因此, 我们不妨可以猜测一个分支, 先继续执行它, 如果发现猜测错误, 再清空重来, 这样节省了猜测正确情况的时间
  - PIPE-中的分支预测
    - **总是选择分支**
    - 对于`ret`指令, 我们不对返回地址做任何预测, 只是暂停处理新指令直到`ret`指令通过写回阶段

---

# 重点知识纲要

- PIPE-存在的问题: 流水线冒险
  - **数据冒险(Data hazard)**: 一条指令的操作数被它前面三条指令中的任一条改变, 则会发生数据冒险
    - 为什么是三条: 在译码阶段读取的操作数, 在三个周期后的写回阶段结束时, 才重新写回到寄存器文件中
    - 解决方法
      - **暂停(Stall)**: 停止流水线中的一条/多条指令, 直到冒险条件不再满足.
        - 处理方法: 要将一条指令阻塞在某一阶段, 就在下一阶段插入一个**气泡(bubble)**, 气泡可以理解为一个自动产生的`nop`指令, **其不会改变寄存器, 内存, 条件码或程序状态** (注意区分下暂停和气泡)
      - **转发(forwarding)/旁路(bypassing)**: 在写回阶段才写回寄存器文件中的值, **若在执行阶段已被计算得到**, 我们通过电路直接将其向前传送到译码阶段, 避免暂停
      - 但如果这个值**在访存阶段才从内存中读取得到(即加载了一个数据)**, 即使转发也晚了一个周期, 这种情况称为**加载/使用冒险**, 为此我们不得不暂停流水线一个周期, 这种解决方案称为**加载互锁**
        - 好消息: 只有`mrmovq`后立即使用对应的寄存器才会导致加载/使用冒险
  - **控制冒险(Control hazard)**: 一条指令要确定下一条指令的跳转地址(只会发生在`jxx`和`ret`)
    - 解决方案: 分支预测 + 消除预测错误的指令(bubble)

---

# 重点知识纲要

- PIPE
  - **主要更改: 完善了数据转发结构`Sel + Fwd A`和`Fwd B`, 处理了数据冒险问题(数据转发 + 加载互锁)**
  - **重点关注PIPE的各阶段对应的HCL和结构图(4.5.7节)**
- 异常处理
  - 三类异常: 执行了`halt`指令; 输入了非法指令/功能码组合的指令; 尝试读写一个非法地址
  - 三个细节
    - 可能有多条指令会引起异常: **流水线中越深的指令引起的异常, 优先级越高**
    - 需要能够取消分支预测错误导致的异常
    - 导致异常指令之后的所有指令都不能够影响系统状态
  - 为此, 我们在流水线寄存器中引入了stat状态码, 并根据异常对应设置其字段, **系统应该禁止异常指令之后的所有指令更新数据内存或条件码寄存器**

---

# 重点知识纲要

- 流水线控制逻辑
  - 对于PIPE, 我们只需要处理以下几个问题: 加载/使用冒险; `ret`指令; 预测错误的分支; 异常
  - **重点: 理解并记忆上述几类特殊控制的发生条件(课本P316 图4-64), 解决方法(课本P318 图4-66 以及P319的两张表格) 以及它们对应的HCL代码**
- 性能分析和未完成的工作: 简单了解, 关注下CPI的计算即可
- 比较经典且困难的题目: 数据转发数的计算和代码运行周期数的计算(自行参考往年题拔高)

---
layout: image-left
image: /05-Processor Arch II/Q1.png
backgroundSize: 90%
---

出处: 2021期中 选择题8

<div class="text-sm" v-click>

A. 该流水线的延迟为$40 + 30 + 40 + 40 + 20 = 170ms$, 吞吐量为$1000 / 170 \approx 5.882 GIPS$ **(不要忽略寄存器的延迟)**

B. 考虑D-E-F-G这条路, 要么插D-E, 要么E-F, 要么F-G. 如果是D-E, 那么为了使每条极大路径上都恰有一个新插入的寄存器, 考虑D-E-F-C, 那么F-C之
间也不能插入了. 但是A-E-F-G上又必须有一个, 所以只能插A-E. 这样之后不管是插在A-B还是B-C所得方案都是合法的. 对称地, 如果是F-G, 结果也是如此.
最后考虑E-F的情况, 此时A-E, D-E, F-C, F-G之间均不能再插入, 但A-B和B-C任选一个插入得到的方案都合法, 因此一共有2+2+2=6种.

C. 注意到对于A-E-F-C这条路, 任两个相邻逻辑单元的延迟为70ms, 插入寄存器不能完全拆分A-E, E-F, F-C这三组逻辑单元, 因此至少有一组为90ms.

D. 只要stall足够多次, 流水线就退化为SEQ, 显然可以. <br> <font color="Red">本题选D.</font>

</div>

---
layout: image-left
image: /05-Processor Arch II/Q2.png
backgroundSize: 90%
---

出处: 2024期中 大题3 <br> **(图(a)即课本P302 图4-52)**

<div class="text-sm" v-click>

(1) `D_valP` `e_valE` `m_valM` `M_valE` `W_valM` `W_valE` `d_rvalA` **(理清逻辑 看图说话 注意最后一个空)**

</div>

<div class="text-sm" v-click>

(2) `Memory(访存)` `m_valM` `1` **(基本知识 考察数据转发)**

</div>

<div class="text-sm" v-click>

(3) `Memory(访存)` `Execute(执行)` `M / M_valA` 
    <br> **(之所以可以这么做 是因为`pushq`在执行阶段不需要用到寄存器的值)**

</div>

---
layout: image-left
image: /05-Processor Arch II/Q3.png
backgroundSize: 70%
---

出处: 2024期中 大题3

<div class="text-sm" v-click>

(4)  `M_dstM` `m_valM` `E_valA` **(依然是看图说话)**

</div> 

<div class="text-sm" v-click>

(5)  `stall`  `stall`  `bubble`  `normal` **(直接考察课本知识点)**

</div> 

---
layout: image-left
image: /05-Processor Arch II/Q4.png
backgroundSize: 90%
---

出处: 2024期中 选择题9

<div class="text-sm" v-click>

第4行使用了`%rax`和`%rbx`, 分别在第二行和第三行被修改, 各使用一次数据转发.

**第5行首先分支预测跳转**, 而后执行第九行, 用到了`%rbx`, 其在第三行被修改, 理解为一次/两次数据转发.

第十行执行完Fetch(取指)阶段后发现分支预测错误, bubble无需进行数据转发, 而后跳转到`L3`, 执行`ret`指令.

<font color="Red">本题选B或C.</font>

</div> 

---
layout: end
---

# Thanks for listening!