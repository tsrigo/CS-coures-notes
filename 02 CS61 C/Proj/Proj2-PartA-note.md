---
title: Proj2-PartA-note
date: 2022-09-24 周六
tags:
  - riscv
mindmap-plugin: basic
---
# Proj2-PartA-note

## Task 1: Absolute Value (Walkthrough)

代码不重要，重要的是几个基本操作：

### Running Tests

```shell
bash test.sh
bash test.sh test_abs
```

### Using VDB to debug tests via Venus

可以用`ebreak`手动在汇编代码中设置断点，十分方便。

开始debug：`vdb test_abs_one.s`

### Using memcheck to debug tests via `test.sh`

`bash test.sh test_abs -mc`

Memcheck introduces two different flags you can add to your `test.sh` command: 

- `-mc` for normal memcheck, and 
- `-mcv` for a more verbose version

## Task 2: ReLU

通过接下来这几个练习，可以总结出**一些模式**：

### 循环：

```assembly
loop_start:
	li t1, 0 # t1 is the "i" in loop
	...  # some initialization
loop_continue:
	...	 # core code
loop_end:
	...  
	addi t1, t1, 1
	blt loop_continue
```

start也可能会被省去。

#### 遍历数组：

```assembly
loop_start:
    li t1, 0    # t1 is the "i" in loop
	...         # some initialization
loop_continue:
	slli t2, t1, 2      # t2: offset of the array, because of type of int we use slli with 2.
    add t0, t2, a0      # t0: address of a[a1 + t1].
    lw t3, 0(t0)        # t3: content of a[a1 + t1].
    ...                 # other operations
loop_end:
	...  				
	addi t1, t1, 1
	blt loop_continue
```

#### 异常处理：

```assembly
ble a1, x0, error	# check if a1 <= x0
error:
    li a0 36
    j exit
```

还有个很重要**Calling convention**

## Calling convention

完整版：[Appendix: Calling Convention | CS 61C Fall 2022](https://cs61c.org/fa22/projects/proj2/calling-convention/)

几个概念：

- **callee-saved registers**: The registers that a function promises to leave unchanged are the **callee-saved registers** (preserved registers). `s0` through `s11` (saved registers) and `sp` are preserved registers.

- **caller-saved registers**: The registers that a function does not promise to leave unchanged are the **caller-saved registers** (non-preserved registers). `a0` through `a7` (argument registers), `t0` through `t6` (temporary registers), and `ra` (return address) are non-preserved registers.

简单来说，保留寄存器和栈指针由被调用函数负责存档（相当于它们之间的约定：callee保证不会改变这些寄存器）；

临时寄存器由函数调用者来负责存档（毕竟被调用的函数不知道哪些临时寄存器是有用的）

几个建议：

- Always save the value of `ra` at the start of a function and restore it at the end of a function.

- Save the values of all the preserved registers at the start of a function and restore them at the end of the function in the prologue and epilogue.

- Save the values of the non-preserved registers that we rely on after a function is being called; in other words, only non-preserved registers values we rely on across a function call should be saved.

在调用函数之后，只有callee-saved registers是可信的，其它的一律都是垃圾，如果有信息，就一定要保存。



这里还提供了两种测试

Randomizing non-preserved registers、Randomizing preserved registers

## Task 5: Matrix Multiplication

这个函数比较复杂（汇编角度）

首先一定要充分把握函数的执行流程，而不是嗯上代码

我先模拟了一下，对各个参数有一定的把握再写

![image-20220924112103617](https://s2.loli.net/2022/09/24/OL9Ix3ZC7ujg6Jr.png)

## Task 6: Testing

一些注意事项：

Focus:
1. There should be both cases of array1[i] > array2[i] and array1[i] < array2[i].
2. You should check the invalid length input.
3. You should imitate the use of any function rightly from unittests.py.
4. When you are writing other test and want to copy some other codes, you should remember to modify the path of function AssemblyTest() and the argument of t.call().
5. Looking into the assembly code of initialize zero and you will find you need both check the lower and upper boundary.