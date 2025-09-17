---
theme: academic
highlighter: shiki
title: 01-Data
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

# 01-Data

信息与计算科学 连昊

---
layout: image-right
image: /01-Data/survey.jpg
backgroundSize: 60%
---

# 小班课安排

- 同学回课
- 简要的研讨题+作业题讲解
  - 作业截止时间后续修改为**周三18:30**
- 基于同学回课内容, 做重点知识的归纳补充
- 适量的练习题
- **希望大家**
  - 小班课前至少读过一遍对应内容的课本 <br> ~~(否则你可能会比较坐牢)~~
  - 认真准备回课, 提高课堂效率
  - 上交的作业请**署名**

---

# 关于Datalab

- 今年有一定难度, 建议至少**预留2-3天**, 及时提交
- 学会拆解问题, 将直观的思路逐步转化为合法运算的实现
- 注意C语言[运算符优先级](https://blog.csdn.net/yuliying/article/details/72898132)
  - **随时打括号**总是一个保险的策略
- 不会的题, 欢迎与同学/助教/树洞交流思路

---

# 作业题

Homeworks

2.59 

```c
res = (x & 0xff) | (y & (~0xff));
```

2.60

```c
unsigned replace_byte(unsigned x, int i, unsigned char b) {

  unsigned mask = ((unsigned) 0xFF) << (i << 3);
  unsigned pos_byte = ((unsigned) b) << (i << 3); // 请注意b的输入类型是unsigned char

  return (x & (~mask)) | pos_byte;
}
```

--- 

# 作业题

Homeworks

2.71 未对无符号数进行符号拓展

```c
int xbyte(packet_t word, int bytenum) {
  return (int) word << (24 - (bytenum << 3)) >> 24;
}
```

---

# 作业题

Homeworks

2.86 请注意如何理解**值**和**十进制** ~~(翻译问题)~~

| 描述               | 值                                                                      | 十进制                      |
| ------------------ | ----------------------------------------------------------------------- | --------------------------- |
| 最小的正非规格化数 | $\underbrace{00 \cdots 0}_{79个0} 1$                                    | $2^{-16445}$                |
| 最小的正规格化数   | $\underbrace{00 \cdots 0}_{15个0} 11 \underbrace{00 \cdots 0}_{63个0}$  | $2^{16382}$                 |
| 最大的规格化数     | $0 \underbrace{11 \cdots 1}_{14个1} 0 \underbrace{11 \cdots 1}_{64个1}$ | $2^{16383} * (2 - 2^{-63})$ |

---

# 作业题

Homeworks

2.87

| 描述                   | Hex    | M                   | E   | V                     | D          |
| ---------------------- | ------ | ------------------- | --- | --------------------- | ---------- |
| -0                     | 0x8000 | 0                   | -14 | -0                    | -0.0       |
| 最小的大于2的值        | 0x4001 | $\frac{1025}{1024}$ | 1   | $\frac{1025}{512}$    | 2.001953   |
| 512                    | 0x6000 | 1                   | 9   | 512                   | 512.0      |
| 最大的非规格化值       | 0x03FF | $\frac{1023}{1024}$ | -14 | $\frac{1023}{2^{24}}$ | 0.000060   |
| $- \infty$             | 0xFC00 | -                   | -   | $- \infty$            | $- \infty$ |
| 十六进制表示为3BB0的数 | 0x3BB0 | $\frac{123}{64}$    | -1  | $\frac{1025}{128}$    | 0.960938   |

---

# 作业题

Homeworks

2.89

A. 正确的. double的精度能够完美表示所有int范围内的数, 转float时即使出现舍入, 也会是相同的舍入

B. 错误的. $x-y$可能会溢出, 如$x=0, y=T_{min}$

C. 正确的. int范围内的运算不足以使double发生舍入

D. 错误的. $x, y, z$充分大时,它们的乘积会使得double产生舍入. 可以通过循环穷举找到反例.

E. 错误的. 如$x=0, z \ne 0$时, 左侧为NaN.

---

# 重点知识纲要(整型)

重点知识只能保证下限, 请好好熟读课本plz

- 基础概念: 位/字节/字长
  - 留意下32位和64位下C语言类型对应的字节数(课本P27 图2-3)
- 进制转换 (二进制 / 十进制 / 十六进制)
- 字节顺序: 大端法与小端法
  - 网络传输中使用大端法: 方便校验和丢弃数据
- ASCII码
  - 0~9 : 0x30~0x39
  - a~z : 0x61~0x7a
  - A~Z : 0x41~0x5a
  - **使用ASCII码作为字符码在任何系统上都将得到相同的结果**(课本P34)
- 布尔代数和位级运算(以及掩码的使用)
  - 相信通过datalab你能很好的掌握这部分内容(
  - **有趣且重要的思考**: 对于四个位级运算符, 我们是否可以从中选取某几个符号, 便足以表达所有位级运算?

---

# 重点知识纲要(整型)

- 逻辑运算与逻辑短路
- 算数移位和逻辑移位 (请关注课本P41的**两个旁注**)
- 整数编码: B(二进制数) / T(补码表示) / U(无符号数)
  - 补码 / 原码 / 反码
  - 显式强制类型转换  
    - 原则:不改变数的**位表示**,只改变解读位的方式
  - **隐式强制类型转换** (课本P52-P53)
    - 赋值: 总是把右值转化为左值的类型
    - 运算: 总是把有符号数转化到无符号数
  - 关注课本P53旁注, 我们是如何定义$T_{min}$的?
  - 扩展和截断
    - 拓展: 先进行数位扩展, 再进行符号转换

---

# 重点知识纲要(整型)

重点知识只能保证下限, 请好好熟读课本plz

- 整数运算
  - 着重关注各个运算什么时候会**溢出**和**舍入**的情况
  - 有符号数的除法, 关注课本P73, 如何通过引入偏置处理舍入问题

---

# 重点知识纲要(浮点数)

- 浮点表示
  - 符号位(S) + 阶码(E) + 尾数(M)
  - 如何根据E的不同翻译成具体的规格化数/非规格化数/特殊值?
    - 32位 S:E:M = 1:8:23  64位 S:E:M = 1:11:52
    - 请一定要理解**原理**, 考试的时候可能会自由更改E和M的位数
  - 理解非规格数如何实现**平滑转换**和**逐渐下溢**(课本P79)
- 舍入
  - 向零/向上/向下舍入
  - 向偶数舍入: 在多次运算中保证结果的期望正确
  - 实数转浮点数, 向偶数舍入; 浮点数转整型, 向零舍入.
    - 小细节: Intel处理器在浮点转整型溢出时会产生$T_{min}$(课本P86)
- 浮点运算
  - **本质**: 先视为实数运算, 再进行舍入
  - 关注各个运算的**性质**是否成立及不成立情况下的反例

---
layout: image-left
image: /01-Data/Q1.png
backgroundSize: 80%
---

出处: 2019期中 选择题T1

<div class="text-sm" v-click>

1. 由于机器是小端法存储, 而IP地址是大端法存储, 故`0x0a029bfd`会被代码读取为`0xfd9b020a` <br> (注意字节内部不反转)
2. 现需要读取IP的前三个字节,对应读取后数字的后三个字节,应该选用的mask是`0x00ffffff`
3. 而192, 168, 56对应的16进制数分别为`0xc0`, `0xa8`, `0x38`, 反序拼接为`0x0038a8c0`
4. 故本题选 <font color="Red">A</font>

</div>

<div class="text" v-click>

考察重点: 大小端法 + 进制转换

</div>

---
layout: image-left
image: /01-Data/Q2.png
backgroundSize: 80%
---

出处: 2024期中 选择题T1

<div class="text-sm" v-click>

A. 选项的操作相当于将$x$的末位置0, 对正负数分别分析知A选项表达式成立

B. 首先注意到操作`>> 31`, 这意味着只需要关注`x | -x`的最高位, 分析知当且仅当x = 0, `x | -x`的最高位才为0, 验证知该选项成立

C. <font color="Red">错误的.</font> 反例: $x = T_{min}, y = 0$

D. 异或运算满足交换律, 且(~x) - y = -x - 1 - y = (~y) - x, 故该选项成立

</div>

<div class="text" v-click>

考察重点: 整型运算

Hint: 关键在于找反例, 多考虑$T_{max}$, $T_{min}$和0

</div>

---
layout: image-left
image: /01-Data/Q4.png
backgroundSize: 80%
---

出处: 2024期中 选择题T3

<div class="text-sm" v-click>

注意到, 除了NaN外, 每个32位浮点数唯一对应一个实数值, 修改后阶码字段增加, 尾数字段减少, 对应NaN的32位浮点数数量减少, 从而可表示的实数值增加

故本题选 <font color="Red">A</font>

</div>

<div class="text" v-click>

考察重点: 浮点表示

</div>

---
layout: image-left
image: /01-Data/Q5.png
backgroundSize: 80%
---

出处: 2018期中 选择题T1

<div class="text-sm" v-click>

表达范围: int < float < double

精度: float < int < double

故本题选 <font color="Red">B</font>

</div>

<div class="text" v-click>

考察重点: 浮点类型转换

</div>

---
layout: image-left
image: /01-Data/Q6.png
backgroundSize: 80%
---

出处: 2022期中 选择题T2

<div class="text-sm" v-click>

A. c = $\inf$ 时不成立

B. c = NaN 时不成立 (注意, NaN与任何数(包括它自身)比较的结果均为false)

C. 浮点运算不满足结合律

D. 浮点运算满足交换律

故本题选 <font color="Red">D</font>

</div>

<div class="text" v-click>

考察重点: 浮点运算性质

</div>

---
layout: image-left
image: /01-Data/Q7.png
backgroundSize: 80%
---

出处: 2022期中 大题T1

<div class="text-sm" v-click>

1(1). 对应的二进制`10111101`, 符号为负, bias = $2^{3 - 1}$ - 1 = 3 <br> 阶码对应幂次3 - 3 = 0, 尾数对应1.8125, 故答案为<font color="Red">-1.8125</font>

1(2). 规格化数阶码对应幂次至少为1 - 3 = -2, 而$\frac{9}{64} = 2^{-3} * 1.125$, 故其为非规格化数, $\frac{9}{64} = 2^{-4} * 0.0625$, 对应符号位和阶码为0, 尾数为`1010`, 即十六进制为<font color="Red">0x09</font>

2. 二者均为正的规格化数, A阶码更大, 故必然有<font color="Red">A > B</font>
   
3 & 4 & 5. 该浮点数最大能表达$2^{6 - 3} * 1.9325 = 15.46$, 即最大能表达整数<font color="Red">$1111_{2}$</font>, 则表达unsigned char `0x6E`时表达为+inf, 即<font color="Red">$01110000$</font>. 结合这两问知<font color="Red">会发生溢出</font>, 而由于最大只能表示4位二进制数, 且尾数有4位, 故<font color="Red">不会发生舍入</font>
   
6. <font color="Red">更多.</font> 同2024期中T3

7. `0x3c00`对应二进制为`0011 1100 0000 0000`, 实数1的阶码即为bias = $2^{m-1} - 1$, 形如`0111..1`, 故在此<font color="Red">m = 5</font>, <font color="Red">n = 10</font>, 最小负非规格数为`1000 0011 1111 1111`, 即<font color="Red">-1023 / (2^24)</font>

</div>

---
layout: end
---

# Thanks for listening!