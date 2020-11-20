# NASM: Register 寄存器

@[TOC](文章目录)

<!-- TOC -->

- [NASM: Register 寄存器](#nasm-register-寄存器)
  - [简介](#简介)
  - [参考](#参考)
- [正文](#正文)
  - [寄存器分类](#寄存器分类)
  - [数据寄存器：`AX`、`BX`、`CX`、`DX`](#数据寄存器axbxcxdx)
    - [高位 H 和低位 L：`AH`、`BH`、`CH`、`DH`、`AL`、`BL`、`CL`、`DL`](#高位-h-和低位-lahbhchdhalblcldl)
    - [32位 & 64位：`EAX`、`EBX`、`ECX`、`EDX`、`RAX`、`RBX`、`RCX`、`RDX`](#32位--64位eaxebxecxedxraxrbxrcxrdx)
  - [段寄存器：`CS`、`DS`、`SS`、`ES`、`FS`、`GS`](#段寄存器csdsssesfsgs)
  - [指针寄存器：`IP`、`SP`、`BP`](#指针寄存器ipspbp)
    - [32位：`EIP`、`ESP`、`EBP`](#32位eipespebp)
  - [变址寄存器：`SI`、`DI`](#变址寄存器sidi)
  - [控制寄存器：`IP`、`FLAGS`](#控制寄存器ipflags)
- [结语](#结语)

<!-- /TOC -->

## 简介

NASM 是一款基于 80x86 的汇编语言编译程序，它支持多种目标文件格式，实现良好的跨平台和模块化特性。本篇记录在 NASM 中能用到的寄存器类型和相关用法。

## 参考

<table>
  <tr>
    <td>nasm-百度百科</td>
    <td><a href="https://baike.baidu.com/item/nasm/10798233">https://baike.baidu.com/item/nasm/10798233</a></td>
  </tr>
  <tr>
    <td>nasm学习记录-寄存器</td>
    <td><a href="https://blog.csdn.net/aiba1227/article/details/102095719">https://blog.csdn.net/aiba1227/article/details/102095719</a></td>
  </tr>
  <tr>
    <td>汇编语言中的标志位：CF、PF、AF、ZF、SF、TF、IF、DF、OF</td>
    <td><a href="https://blog.csdn.net/weixin_41890599/article/details/99866410">https://blog.csdn.net/weixin_41890599/article/details/99866410</a></td>
  </tr>
</table>

# 正文

## 寄存器分类

存储设备有分为大容量非易失性的外存设备，还有作为 CPU 运行环境的主要场所的内存。但是对于 CPU 内部来说这些都太慢了，我们总是需要一些 CPU 内部的空间来保存需要常访问或是常驻在 CPU 内部的数据(如指令计数器、段基址)，也就是所谓的`寄存器(Register)`。

由于 NASM 作为跨平台的汇编语言编译器，它定义了一套通用的寄存器。NASM 提供的寄存器可以分为五大类：

- 数据寄存器
- 段寄存器
- 指针寄存器
- 变址寄存器
- 控制寄存器

前三种又共同称为通用寄存器。其实寄存器使用时几乎没什么区别，接下来说明的寄存器的意义也仅仅是程序员使用寄存器时候的惯例或是执行特定操作时相关寄存器才会发挥作用(如 `loop` 根据 `cx`、`jmp` 根据 `cs` 等)，否则则执行过程中都能够作为通用寄存器使用，只要在方法结束后或是特定操作前恢复该有的值即可。


本篇主要集中介绍各个寄存器的分类，相关特定指令应用则将在另一篇解说。这边需要注意的是 80x86 是以 `16bits` 为单位的处理器，所以接下来提到的的寄存器除非特别说明，否则都是长度为 16 位的寄存器。接下来我们就来看看这些寄存器到底都能拿来干什么吧

## 数据寄存器：`AX`、`BX`、`CX`、`DX`

看名字就知道，存数据的。通用的数据寄存器有 `AX`、`BX`、`CX`、`DX` 四种

- AX 累加寄存器 (Accumlator)

通常用于保存算中间值的操作数，也是与 I/O 设备交互时与外设传输数据的寄存器

- BX 基址寄存器 (Base)

通常用于内存寻址时保存地址基址的寄存器，可以配合 DI、SI 提供更复杂的寻址模式

- CX 计数寄存器 (Counter)

通常用于保存循环次数(loop 指令会用到)，也可用于保存用于算数运算、位运算的参数等(乘数、位移数等)

- DX 数据寄存器 (Data)

通常就是用于存储数据的，偶尔在大数字的乘除运算时搭配 AX 形成一个操作数

### 高位 H 和低位 L：`AH`、`BH`、`CH`、`DH`、`AL`、`BL`、`CL`、`DL`

四种 16 位寄存器都能被拆分成高 8 位和低 8 位，也就是 `AH`、`BH`、`CH`、`DH` 和 `AL`、`BL`、`CL`、`DL`

这 8 个寄存器并不是新的寄存器，而是取对应 16 位寄存器的部分内容而已，以 AX 的实际存储情形举例

![](https://picures.oss-cn-beijing.aliyuncs.com/img/AX&AH&AL.png)

### 32位 & 64位：`EAX`、`EBX`、`ECX`、`EDX`、`RAX`、`RBX`、`RCX`、`RDX`

同时 NASM 也支持我们编写 32 位甚至 64 位的通用寄存器大小，分别是 `EAX`、`EBX`、`ECX`、`EDX` 和 `RAX`、`RBX`、`RCX`、`RDX`

例如原来的 AX 即为 EAX 的低 16 位如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/RAX&EAX&AX.png)

## 段寄存器：`CS`、`DS`、`SS`、`ES`、`FS`、`GS`

第二个我们来介绍段寄存器，因为后面其他寄存器很多都需要与段寄存器共同使用。NASM 定义的段寄存器有 4+2 种(80386 后多出后面两种，提供更多选择)

- CS 指令段寄存器 (Code)

用于保存当前执行程序的`指令段(code segment)`的起始地址，相当于 `section .text` 的地址

- DS 数据段寄存器 (Data)

用于保存当前执行程序的`数据段(data segment)`的起始地址，相当于 `section .data` 的地址

- SS 栈寄存器 (Stack)

用于保存当前`栈空间(Stack)`的基址，与 SP(偏移量) 相加 -> SS:SP 可找到当前栈顶地址

- ES 额外段寄存器 (Extra)

常用于字符串操作的内存寻址基址，与变址寄存器 DI 共用

- FS、GS 指令段寄存器

80386 额外定义的段寄存器，提供程序员更多的段地址选择

## 指针寄存器：`IP`、`SP`、`BP`

有了前面的段寄存器保存不同区块(section 或称为`段 segment`)，我们还有三个寄存器作为块中指针(用来保存`偏移量 offset`)

- IP 指令指针 (Instruction Pointer)

与 CS 共用，可透过 CS:IP 寻到当前程序执行到的地址

- SP 栈指针 (Stack Pointer)

与 SS 共用，可透过 SS:SP 找到当前`栈顶`地址

- BP 参数指针 (Base Pointer)

与 SS 共用，可透过 SS:BP 找到当前`栈底`地址


有关指针寄存器的寻址如下：

![](https://picures.oss-cn-beijing.aliyuncs.com/img/pointer_usage.png)

### 32位：`EIP`、`ESP`、`EBP`

与通用寄存器类似，指针寄存器也支持 32 位的模式，分别是 `EIP`、`ESP`、`EBP`

## 变址寄存器：`SI`、`DI`

由于寄存器有限，有时候我们并不会直接保存操作数或是操作数的地址，而是保存操作数的某个相对偏移量，透过间接寻址来操作数据。变址寄存器就是在这个过程中负责保存偏移量的部分

- SI 源变址寄存器 (Source Index)

通常用于保存源操作数(字符串)的偏移量，与 DS 搭配使用(DS:SI)

- DI 目的变址寄存器 (Destination Index)

通常用于保存目的操作数(字符串)的偏移量，与 ES 搭配使用(ES:DI)

## 控制寄存器：`IP`、`FLAGS`

其实 IP 也算是一种控制寄存器，决定程序执行时取指令的偏移量，不过由于依旧是一个偏移量所以这边放在指针寄存器的部分来介绍

- FLAGS 标识位

控制寄存器 FLAGS 保存 CPU 运行的状态和一些标识位，每个位代表了不同的含义：

![](https://picures.oss-cn-beijing.aliyuncs.com/img/assembly_flags.png)

- (第0位) CF 进位标志位(Carry Flag)：有进位或借位时置 1
- (第2位) PF 奇偶标志位(Parity Flag)：运算结果 1 的个数为偶数时置 1
- (第4位) AF 辅助进位标志位(Assistent Carry Flag)：低四位运算时的进位标志位
- (第6位) ZF 零标志位(Zero Flag)：运算结果为 0 时置 1
- (第7位) SF 符号标志位(Sign Flag)：运算结果为负时置 1，与结果的最高位相同
- (第8位) TF 陷阱标志位(Trap Flag)：置 1 表示单步调试，每次执行一个指令
- (第9位) IF 中断允许标志位(Interrupt Enable Flag)：置 1 时可响应外部中断
- (第10位) DF 方向标志位(Direction Flag)：CLD 指令将 DF 置 1，STD 将 DF 置 0；置 0 时串行指令每次操作 SI、DI 递减(与字符串存储方向相关)
- (第11位) OF 溢出标志位(Overflow Flag)：运算结果溢出时置 1(过大 or 过小)

# 结语

寄存器是编写汇编程序时非常重要的部件，我们肯定没办法接受每次访问一个数都要访问一次内存，所以大多数操作都要先将数据放到寄存器进行运算后重新写入。

熟记不同寄存器的用途并编写符合规范的代码是良好的编程习惯，不仅提高代码的可读性也比较能避免寄存器没有正常恢复时产生的异常。
