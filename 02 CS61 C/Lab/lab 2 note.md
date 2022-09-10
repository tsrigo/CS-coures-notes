---
title: lab 2 note
date: 2022-09-05 周一
tags:
  - c
  - 位运算
  - GDB
  - Valgrind
  - 'Memory_Management'

mindmap-plugin: basic
---
# lab 2 note

## Exercise 1: Bit Operations

```c
/* Returns the Nth bit of X. Assumes 0 <= N <= 31. */
unsigned get_bit(unsigned x, unsigned n) {
    /* YOUR CODE HERE */
    return (x >> n) & 1;
}

/* Set the nth bit of the value of x to v. Assumes 0 <= N <= 31, and V is 0 or 1 */
void set_bit(unsigned *x, unsigned n, unsigned v) {
    /* YOUR CODE HERE */
    *x = (v) 
        ? *x | (1 << n) 
        : *x & ~(1 << n);
}

/* Flips the Nth bit in X. Assumes 0 <= N <= 31.*/
void flip_bit(unsigned *x, unsigned n) {
    /* YOUR CODE HERE */
    get_bit(*x, n) ? set_bit(x, n, 0) : set_bit(x, n, 1);
}
```

要理解好各种操作，将他们组合在一起，特别是中间的`set_bit`

## Exercise 2: Using GDB to find segfaults

```shell
cgdb ./ll_cycle
run
p hare
p hare*
```

## Exercise 3: Valgrind

### Using Valgrind to find segfaults

```shell
valgrind ./ll_cycle
```

The next chunk tells us that we encountered a segfault and repeats the information above.

```bash
==4662== Process terminating with default action of signal 11 (SIGSEGV): dumping core
==4662==  Access not within mapped region at address 0x8
==4662==    at 0x108C4B: ll_has_cycle (ll_cycle.c:9)
==4662==    by 0x108B36: main (test_ll_cycle.c:50)
```

### Invalid reads and memory leaks

```shell
valgrind ./bork hello
```

But back on debugging: A good general rule of thumb to follow when parsing big error logs is to only consider **the first error message** (and ignore the rest), so let's do that:

```bash
==10170== Invalid read of size 1
==10170==    at 0x4C34D04: strlen (in /usr/lib/valgrind/vgpreload_memcheck-amd64-linux.so)
==10170==    by 0x10879F: make_Str (bork.c:22)
==10170==    by 0x108978: translate_to_bork (bork.c:56)
==10170==    by 0x1089F2: main (bork.c:68)
```

The error message states that we are doing an invalid read of size 1. What does this mean? **An invalid read means that your program is reading memory at a place that it shouldn't be (this can cause a segfault, but not always).** Size 1 means that we were attempting to read 1 byte.

Because we're unfamiliar with this ancient codebase and we don't want to read all of it to find the bug, a good process to follow is to start at **high-level details and work our way down** (so basically work our way through the call stack that valgrind provides).

回溯：68行 - 56行 - 22行 一句一句检查



Now let's take a look at the leak summary below. This just states that we lost 8 bytes in 1 block.

```
==29797== LEAK SUMMARY:
==29797==    definitely lost: 8 bytes in 1 blocks
==29797==    indirectly lost: 0 bytes in 0 blocks
==29797==      possibly lost: 0 bytes in 0 blocks
==29797==    still reachable: 0 bytes in 0 blocks
==29797==         suppressed: 0 bytes in 0 blocks
==29797== Rerun with --leak-check=full to see details of leaked memory
```

It tells us to "Rerun with --leak-check=full to see details of leaked memory", so let's do that.

```shell
valgrind --leak-check=full ./bork hello
```

Now Valgrind is telling us the location where the unfree'd block was initially allocated.

## Exercise 4: Memory Management

### *vector_new()

这个函数要理清楚步骤，注释相当详细了

```c
vector_t *vector_new() {
    /* Declare what this function will return */
    vector_t *retval;

    /* First, we need to allocate memory on the heap for the struct */
    retval = malloc(sizeof(vector_t));

    /* Check our return value to make sure we got memory */
    if (retval == NULL) {
        allocation_failed();
    }

    /* Now we need to initialize our data.
       Since retval->data should be able to dynamically grow,
       what do you need to do? */
    retval->size = 1;
    retval->data = malloc(sizeof(int));

    /* Check the data attribute of our vector to make sure we got memory */
    if (retval->data == NULL) {
        free(retval);				//Why is this line necessary?
        allocation_failed();
    }

    /* Complete the initialization by setting the single component to zero */
    retval->data[0] = 0;

    /* and return... */
    return retval; /* UPDATE RETURN VALUE */
}
```

### vector_set

有几点要注意

- `loc >= v-size`时要重新分配空间，注意包含等于的情况，因为数组下标从0开始
- 重新分配空间应该是一个不断迭代直至满足条件的过程。我一开始用了`if`，是不合适的。
- 用`realloc`重新分配空间之后，需要相应的修改`v->size`
- 还需要判断是否分配成功`v->data == NULL`

```c
void vector_set(vector_t *v, size_t loc, int value) {
    /* What do you need to do if the location is greater than the size we have
     * allocated?  Remember that unset locations should contain a value of 0.
     */
    /* YOUR CODE HERE */
    while (loc >= v->size){ // *subtle
        v->data = realloc(v->data, sizeof(int) * (v->size)* 2);
        v->size *= 2;       // *subtle
        if (v->data == NULL){
            allocation_failed();
        }
    }
    v->data[loc] = value;
}
```

