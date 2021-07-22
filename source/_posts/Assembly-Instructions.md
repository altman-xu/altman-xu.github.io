---
title: Assembly-Instructions
tags:
  - assembly
toc: true
date: 2021-06-30 09:59:21
categories:
---

|| 字节 | 描述 |
| -- | -- | -- |
| b | 1 bytes | 字节 |
| w | 2 bytes | 字 |
| l | 4 bytes | 双字 |
| q | 8 bytes | 四字 |

## 指令

| 指令 | 效果 | 描述 |
| -- | -- | -- |
| MOV S,D | D<-S | 传送 |
| movb |  | 传送字节 |
| movw |  | 传送字 |
| movb |  | 传送双字 |
| movq |  | 传送四字 |
| movabsq I,R | R<-I | 传送绝对的四字 |


MOV 源和目的类型的5种可能组合

|指令|说明|字节|
|--|--|--|
| movl $0x4050,%eax | Immediate--Register | 4 bytes |
| movw %bp,%sp | Register--Register | 2 bytes |
| movb (%rdi,%rcx),%al | Memory--Register | 1 bytes |
| movb $-17,(%rsp) | Immediate--Memory | 1 bytes |
| movq %rax,-12(%rbp) | Register--Memory | 8 bytes |




| 指令 | 效果 | 描述 |
| -- | -- | -- |
| leaq S,D | D <-&S | 加载有效地址 |
| -- | -- | -- |
| INC D | D <- D+1 | 加1 |
| DEC D | D <- D-1 | 减1 |
| NEG D | D <- -D  | 取负 |
| NOT D | D <- ~D  | 取补 |
| -- | -- | -- |
| ADD S,D | D <- D+S | 加 |
| SUB S,D | D <- D-S | 减 |
| IMUL S,D | D <- D*S | 乘 |
| XOR S,D | D <- D~S | 异或 |
| OR S,D | D <- D|S | 或 |
| AND S,D | D <- D&S | 与 |
| -- | -- | -- |
| SAL k,D | D <- D <<k | 左移 |
| SHL k,D | D <- D <<k | 左移(等同于SAL) |
| SAR k,D | D <- D >>ak | 算术右移 |
| SHR k,D | D <- D >>lk | 逻辑右移 |


## CMP TEST

cmp 和 test 指令不修改任何寄存器的值，只设置条件码

1. cmp 根据两个操作数之差来设置条件码，除了只设置条件码而不更新目的寄存器外，cmp指令与sub指令的行为是一样的。如果两个操作数相等，这些指令会将零标志设置为 1，而其他的标志可以用来确定两个操作数之间的大小关系
2. test 除了只设置条件码而不更新目的寄存器外，指令与 and 指令是一样的。 典型用法是两个操作数是一样的(例如 testq %rax,%rax 用来检查 %rax是负数、零还是正数)，或其中的一个操作数是一个掩码，用来只是哪些位应该被测试

| 指令 | 基于 | 描述 |
| -- | -- | -- |
| CMP S1, S2 | S2-S1 | 比较 |
| cmpb |  | 比较字节 |
| cmpw |  | 比较字 |
| cmpl |  | 比较双字 |
| cmpq |  | 比较四字 |
| -- | -- | -- |
| TEST S1, S2 | S2&S1 | 测试 |
| testb |  | 测试字节 |
| testw |  | 测试字 |
| testl |  | 测试双字 |
| testq |  | 测试四字 |

## 条件码

程序状态字PSW（Program State Word）

除了整数寄存器，CPU还维护一组单个位的条件码寄存器，他们描述了最近的算术或逻辑操作的属性。
可以检测这些寄存器来执行条件分支指令

| 31··· | 22 | 21  | 20 | 19 | 18 | 17 | 16 | 15 | 14 | 13 | 12 | 11 | 10 | 9 | 8 | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
| -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- |
|    |    |    |  ID  |  VIP  |  VIF  |  AC  |  VM  |  O  |  RF  |  NT  |  IOPL  |  OF  |  DF  |  IF  |  TF  |  SF  |  ZF  |  x  |  AF  |  x  |  PF  |  x  |  CF  |


| 条件码 |  | 标志 | 说明 |
| -- | -- | -- | -- |
| CF | Carry Flag| 进位标志 | 最近的操作使最高位产生了进位，可用来检查无符号操作的溢出 |
| ZF | Zero Flag | 零标志 | 最近的操作得出的结果为0 |
| SF | Sign flag | 符号标志 | 最近的操作得到的结果为负数 |
| OF | Overflow  | 溢出标志 | 最近的操作导致一个补码溢出--正溢出或负溢出 |
| PF | Panty Flag| 偶标志  |  |
| AF | Auxiliary | 半进/借位  |  |
| TF | Trap Flag | 陷阱  |  |
| IF | Interrupt | 允许中断  |  |
| DF | Direction | 增量方向  |  |
| VIF |  | 虚拟中断标志位  |  |
| VP |  | 虚拟中断待决标志位  |  |
| IOPL |  | IO特权级别  |  |


条件码同城不会直接读取，常用的使用方法有三种

1. (set指令) 根据条件码的某种组合，讲一个字节设置为0或1
2. (jump指令) 可以条件跳转到程序的某个其他地方
3. (cmove指令) 可以有条件的传送数据

### SET

set指令，每条指令根据条件码的组合，将一个字节设置为0或1。
有些指令有同义名，也就是同一条机器指令有别的名字

| 指令 | 同义名 | 效果 | 设置条件 |
| -- | -- | -- | -- |
| sete D | setz | D <- ZF | 相等/零 |
| setne D | setnz | D <- ~ZF | 不等/非零 |
| sets D |  | D <- SF | 负数 |
| setns D |  | D <- ~SF | 非负数 |
| setg D | setnle | D <- ~(SF^OF)&~ZF | 大于(有符号>) |
| setge D | setnl | D <- ~(SF^OF) | 大于等于(有符号>=) |
| setl D | setnge | D <- SF^OF | 小于(有符号<) |
| setle D | setng | D <- (SF^OF)|ZF | 小于等于(有符号<=) |
| seta D | setnbe | D <- ~CF&~ZF | 超过(无符号>) |
| setae D | setnb | D <- ~CF | 超过或相等(无符号>=) |
| setb D | setnae | D <- CF | 低于(无符号<) |
| setbe D | setna | D <- CF\|ZF | 低于或相等(无符号<=) |

### JUMP

当条件满足时，这些指令会调整到一条带标号的目的地

| 指令 | 同义名 | 跳转条件 | 描述 |
| -- | -- | -- | -- |
| jmp Label |  | 1 | 直接跳转 |
| jmp *Operand |  | 1 | 间接跳转 |
| je Label | jz | ZF | 相等/零 |
| jne Label | jnz | -ZF | 不相等/非零 |
| js Label |  | SF | 负数 |
| jns Label |  | ~SF | 非负数 |
| jg Label | jnle | ~(SF^OF)&~ZF | 大于(有符号>) |
| jge Label | jnl | ~(SF^OF) | 大于或等于(有符号>=) |
| jl Label | jnge | SF^OF | 小于(有符号<) |
| jle Label | jng | (SF^OF)\|ZF | 小于或等于(有符号<=) |
| ja Label | jnbe | ~CF&~ZF | 超过(无符号>) |
| jae Label | jnb | ~CF | 超过或相等(无符号>=) |
| jb Label | jnae | CF | 低于(无符号<) |
| jbe Label | jna | CF|ZF | 低于或相等(无符号<=) |

### CMOVE

条件传送指令。当传送条件满足时，指令吧源值S复制到目的地R

| 指令 | 同义名 | 传送条件 | 描述 |
| -- | -- | -- | -- |
| cmove S,R | cmovz | ZF | 相等/零 |
| cmovne S,R | cmovnz | ~ZF | 不相等/非零 |
| cmovs S,R |  | SF | 负数 |
| cmovns S,R |  | ~SF | 非负数 |
| cmovg S,R | cmovnle | ~(SF^OF)&~ZF | 大于(有符号>) |
| cmovge S,R | cmovnl | ~(SF^OF) | 大于或等于(有符号>=) |
| cmovgl S,R | cmovnge | SF^OF | 小于(有符号<) |
| cmovgle S,R | cmovng | (SF^OF)\|ZF | 小于或等于(有符号<=) |
| cmova S,R | cmovnbe | ~CF&~ZF | 超过(无符号>) |
| cmovae S,R | cmovnb | ~CF | 超过或相等(无符号>=) |
| cmovb S,R | cmovnae | CF | 低于(无符号<) |
| cmovbe S,R | cmovna | CF\|ZF | 低于或相等(无符号<=) |



## 条件控制转移 条件数据传送
为了理解为什么基于条件数据传送的代码会比基于条件控制转移的代码性能要好，我们必须了解一些关于现代处理器如何运行的知识。

处理器通过使用流水线(pipelining)来获得高性能，在流水线中，一 条指令的处理要经过一系列的阶段，每个阶段执行所需操作的一小部分(例如，从内存取 指令、确定指令类型、从内存读数据、执行算术运算、向内存写数据，以及更新程序计数 器)。这种方法通过重叠连续指令的步骤来获得高性能，例如，在取一条指令的同时，执 行它前面一条指令的算术运算。要做到这一点，要求能够事先确定要执行的指令序列，这 样才能保持流水线中充满了待执行的指令。当机器 遇到条件跳转(也称为 “分支”)时，只 有当分支条件求值完成之后，才能决定分支往哪边走。处理器采用非常精密的分支预测逻 辑来猜测每条跳转指令是否会执行。只要它的猜测还比较可靠(现代微处理器设计试图达 到 90%以上的成功率)，指令流水线中就会充满着指令。另一方面，错误预测一个跳转， 要求处理器丢掉它为该跳转指令后所有指令已做的工作，然后再开始用从正确位置处起始 的指令去填充流水线。正如我们会看到的，这样一个错误预测会招致很严重的惩罚，浪费 大约 15~30 个时钟周期，导致程序性能严重下降。


条件控制转移 与 条件数据传送 demo 如下:

```zsh
v = test-expr ? then-expr : else-expr;

基于 条件控制转移 伪代码如下:    (then-expr / else-expr 这两个只执行一次)
if (!test-expr)
  goto false;
v = then-expr;
goto done;
false:
  v = else-expr;
done;

基于 条件数据传送 伪代码如下:    (then-expr / else-expr 这两个都会被执行)
v = then-expr;
ve = else-expr;
t = test-expr;
if(!t) v = ve;
```