---
title: Proj02-partB-note
date: 2022-09-29 周四
tags:
  - riscv
mindmap-plugin: basic
---
# Proj02-partB-note

## Task 7: Read Matrix

### 寄存器的保存

#### callee的视角

Read Matrix本身作为一个函数，得保存好**callee-saved registers**

#### caller的视角

**Read Matrix**会调用其它函数，因此在调用前得保存好**caller-saved registers**

由于a0, a1, a2是经常用的寄存器，所以基本所有函数调用都得保存它们，另外ra必须保存。

```assembly
    # prologue of xxx
    addi sp, sp, -16
    sw ra, 0(sp)
    sw a0, 4(sp)
    sw a1, 8(sp)
    sw a2, 12(sp)
    # epilogue of xxx
    lw ra, 0(sp)
    lw a0, 4(sp)
    lw a1, 8(sp)
    lw a2, 12(sp)
    addi sp, sp, 16
```

## Calling Convention

虽然概念搞清楚了，但是在敲代码时还是犯了不少错：

### 没有保存全部临时寄存器

#### 错误：

```assembly
# prologue of malloc
addi sp, sp, -8
sw ra, 0(sp)
sw a0, 4(sp)
# call malloc
li a0, 8
jal malloc
    # check error
beq a0, x0, mallocError
    # save result
add s1, a0, x0
# epilogue of malloc
lw ra, 0(sp)
lw a0, 4(sp)
addi sp, sp, 8
```

#### 改正

注意要搞清楚，如果函数参数只有一个a0，并不代表着你只需要保留a0。我们并不能确定callee的内部会修改哪些临时寄存器，因此只要某个寄存器之后要用，我们就一定要保存。

e.g. 虽然malloc只有一个参数，但是我们得把要用的临时寄存器都保留（a0,a1,... 是临时寄存器不是保留寄存器！）

```assembly
    # prologue of malloc
    addi sp, sp, -16
    sw ra, 0(sp)
    sw a0, 4(sp)
    sw a1, 8(sp)
    sw a2, 12(sp)
    # call malloc
    li a0, 8
    jal malloc
        # check error
    beq a0, x0, mallocError
        # save result
    add s1, a0, x0
    # epilogue of malloc
    lw ra, 0(sp)
    lw a0, 4(sp)
    lw a1, 8(sp)
    lw a2, 12(sp)
    addi sp, sp, 16
```

### 使用未恢复的寄存器

#### 错误

```assembly
    # prologue of fread
    addi sp, sp, -16
    sw ra, 0(sp)
    sw a0, 4(sp)    # The file descriptor of the file we want to read from, previously returned by fopen.
    sw a1, 8(sp)    # A pointer to the buffer where the read bytes will be stored. 
    sw a2, 12(sp)   # The number of bytes to read from the file.
    # call fread
    add a0, s0, x0
    add a1, s1, x0
    li a2, 8
    jal fread
        # check error
    bne a0, a2, freadError  # If a0 differs from the argument provided in a2, then we should raise an error.
    # epilogue of fread
    lw ra, 0(sp)
    lw a0, 4(sp)
    lw a1, 8(sp)
    lw a2, 12(sp)
    addi sp, sp, 16
```

#### 改正

如果 a0 与 a2 中提供的参数不同，那么应该报错。

但如同上面解释的那样，a2是临时寄存器，经过函数调用，值是无法保证的，就算是参数也不例外。

因此我们得重新加载a2: 在 `bne a0, a2, freadError` 前添加 `li a2, 8`



## 几个小点

- 第二次调用fread不会读取行和列
- 最后记得把矩阵的地址赋值给a0



## Task 8: Write Matrix

和task7差不多，主要是注意细节，理清楚逻辑，尤其是步骤二。

步骤二的步骤：

1. 调用malloc生成buffer，并将其（地址）保存至s1
2. 使用sw将a2, a3（行，列）保存至buffer
3. 调用fwrite

## Task 9: Classify

写的太久，反而失了不少印象，赶紧把一些余温记录下来。

- row and col是需要存储在malloc分配的buffer里的。分配后的指针作为参数传给read_matrix进行赋值，之后即可通过lw来访问。（注意长和宽并不一定是28，而应根据读取的结果而定）

- 使用malloc为矩阵分配内存时，注意malloc是以byte为单位的。

- 区分"A pointer to the filepath string of m0"和"A pointer to the matrix in memory (stored as an integer array)"：每个矩阵有两种存在形式，一种是矩阵的文件位置，一种是int型数组，分别对应了两种指针，使用不同的函数时要注意选择正确的参数

- 处理好指针与其值的关系：a1是一个数组，存储着几个字符串filepath的指针，因此如果需要访问filepath，应该要使用lw进行读取; 但注意即使是访问这几个字符指针(write_matrix)，也需要要用lw读取a1，而不是简单的a1 + 4, a1 + 8之类。毕竟数组名本身就是指针。

这些东西如果在开始敲代码之前就搞清楚，可能就会走很多弯路，因此还是得多想，尽量不要等出bug了再来debug时补这些错误；不过换句话说也是这些bug加深了我对这些概念的理解吧；也许最好的状态应该是debug时发现的真的是理解比较薄弱造成的。

