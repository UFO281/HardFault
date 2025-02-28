[TOC]

# HardFault 分析教程
Teach you how to locate HardFault and analyze its causes（教你如何定位HardFault，并分析HardFault的原因）

## 1. 定位HardFault产生处

### 1.1 C语言版本
>把你的void HardFault_Handler(void) 替换成下摆你这段代码，然后在hardfault处打断点，单步执行就可以找到问题原因处

```c
// Return the R13(Stack Pointer)
__ASM unsigned int __Get_SP_Value(void)
{
    mov r0, r13 //R13(SP)
    bx lr
}

void (*f1)(void);
unsigned int result;

void HardFault_Handler(void)
{
    //实际操作Arm Coretex-M0核心的是+0x1C，M3核的是+0X14
    result = __Get_SP_Value()+0x14;
    f1 = *( (unsigned int*)result );
    f1();
    while (1);
}
```
>![alt text](image-5.png)

---

### 1.2 汇编语言版本
```armasm
；把你的起始.s文件内的HardFault_Handler代码换成下边这个

HardFault_Handler\
                PROC
                add   r0, sp, #0x14     
                ldr   r1, [r0]          
                blx   r1                
                B       .
                ENDP
```
>![alt text](image-6.png)
---
## 2. 分析HardFault原因

>分析产生HardFault的原因（需要具体代码具体分析仔细崩溃处的代码的问题）
hardfault产生的原因有很多，例如访问非法地址了，函数指针变量为NULL，然后直接运行了，总共包含以下几种

1. 内存访问错误

- 访问了未映射的内存区域。

- 对只读内存区域执行写操作。

- 访问了无效的内存地址（如NULL指针解引用）。

2. 堆栈溢出：

- 如果任务或中断的堆栈空间不足，可能导致堆栈溢出，进而破坏内存或触发HardFault。

3. 未对齐的内存访问：

- 某些STM32系列要求特定的数据类型在内存中对齐。未对齐的访问可能触发HardFault。

4. 错误的异常处理：

- 如果在异常处理程序中再次发生异常，且没有正确处理，可能会导致HardFault。

5. 中断优先级配置错误：

- 如果配置了错误的中断优先级，可能导致不可屏蔽中断（NMI）或HardFault。

6. 非法指令或状态：

- 执行了未定义的指令。

- 尝试切换到无效的处理器模式。

7. 外设配置错误：

- 错误地配置或使用了外设，如未启用时钟就访问外设寄存器。

8. 电源问题：

- 不稳定的电源可能导致处理器行为异常，从而触发HardFault。

9. 软件错误：

- 逻辑错误、数组越界、类型转换错误等编程错误也可能导致HardFault。

>但是这些太笼统了，太废话了，具体是哪个呢？？ 那还得看HardFault状态寄存器（HFSR）以及其他相关的故障状态寄存器，如配置管理故障状态寄存器（CFSR）、内存管理故障状态寄存器（MMFSR）、总线故障状态寄存器（BFSR）和使用故障状态寄存器（UFSR）。
下图是HardFault状态寄存器，引发hardfault了可以看看这个寄存器值，然后再仔细检查下代码
---
### 2.1 Chinese version
>![alt text](image.png)
>![alt text](image-1.png)

### 2.1 English version
>![alt text](image-2.png)
>![alt text](image-4.png)
>![alt text](image-3.png)