

# C 2 MIPS - introduction

In this exercise, we are going to walk through translating a C program to a MIPS program. The following program will print out the nth Fibonacci number.

```c
#include <stdio.h>

int n = 9;

// Function to find the nth Fibonacci number
int main(void) {
    int curr_fib = 0, next_fib = 1;
    int new_fib;
    for (int i = n; i > 0; i--) {
        new_fib = curr_fib + next_fib;
        curr_fib = next_fib;
        next_fib = new_fib;
    }
    printf("%d\n", curr_fib);
    return 0;
}
```

Let's break down how we'll translate this step-by-step. First, we need to re-write the c source code into an [unstructured version](put_a_link_here). 

```c
#include <stdio.h>

int n = 9;

// Function to find the nth Fibonacci number
int main(void) {
    int curr_fib = 0, next_fib = 1;
    int new_fib;
    int i = n;

WHILE:
    if (i == 0) goto ELIHW:
    new_fib  = curr_fib + next_fib;
    curr_fib = next_fib;
    next_fib = new_fib;
    i--;
    goto WHILE;

ELIHW:
    printf("%d\n", curr_fib);
    return 0;
}
```

Next, we need to define the global variable n. In MIPS, global variables are declared under the .data directive. This represents the data segment. It will look like this:

```mips
.data
n: .word 9
```
- `n` is the name of the variable,
- `.word` means that the size of the data is one word,
- `9` is the value that is assigned to `n`.

Let's move on to initializing `curr_fib` and `next_fib`. Since these are local variables in our c code, they will be assigned to some registers of the MIPS cpu. recall that, by convention, the $s0-$s7 registers are used for local variables in MIPS.

```mips
.text
main:
    add  $s0, $0, $0 # curr_fib = 0
    addi $s1, $0,  1 # next_fib = 1
```
Here we have added the `.text` directive. Everything under this directive is our executable code.

Remember that $0 always holds the value 0.

We don't need to do anything to declare `new_fib` (we don't declare variables in MIPS).

Next, let's get to the loop. We'll start with setting up the loop variables. The following code will set `i` to `n`.

```mips
la $t0, n      # load the address of the label n
lw $t0, 0($t0) # get the value that is stored at the address denoted by the label n
```
You can think of the code above as doing something along the lines of

```c
 t0 = &n;
 t0 = *t0;
```

The instruction `la` loads the address of a label into a given register. The first line essentially sets `$t0` to be a pointer to `n`. the second line uses `lw` to dereference `$t0` and sets `$t0` to the value stored at `n`.

Now, you're probably thinking: "Why can't we directly set `$t0` to `n`?" In the `.text` section, there is no way* that we can directly access `n` (Think about it. We can't say `add $t0, n, $0`. The arguments to `and` **must be registers** and `n` **is not** a register). The only way that we can access `n` is by obtaining its address. Once we obtain the address of `n`, we need to dereference it which can be done with `lw`. the instruction `lw` will reach into memory at the address that you specify and load in the value stored at that address. In this case, we specified the address of `n` and added an offset of `0`.

Let's get down to the loop now. First, we'll create the outer structure below:

```mips
WHILE:
    beq $t0, $0, ELIHW  # exit loop once we have completed n iterations
    ...
    ...
    addi $t0, $t0, -1   # decrement counter
    j WHILE             # loop
ELIHW:
```

The first line (`WHILE:`) is a label that we will use to jump back to the beginning of the loop.

The next line (`beq $t0, $0, ELIHW`) specifies our terminating condition. Here, we will jump to another label, `ELIHW`, once $t0 (which is representing `i`) reaches `0`.

The line (`addi $t0, $t0, -1`) decrements `i` at the end of the loop body.

The next instruction jumps back to the start of the loop.

Now, let's add in the loop body.

```mips
WHILE:
    beq $t0, $0, ELIHW  # exit loop once we have completed n iterations
     
    add $t2, $s0, $s1   # new_fib  = curr_fib + next_fib;
    add $s0, $s1, $0    # curr_fib = next_fib;
    add $s1, $t2, $0    # next_fib = new_fib;
     
    addi $t0, $t0, -1   # decrement counter
    j WHILE             # loop
ELIHW:
```
Nothing special here. The corresponding C lines are written in the comments.

Let's print out the nth Fibonacci number!

```mips
ELIHW:
    addi $v0,  $0, 1 # argument to syscall to execute print integer
    addi $a0, $s0, 0 # argument to syscall, the value to be printed
    syscall          # execute 'print integer'
```

Printing is a system call. You'll learn more about these later in the semester, but a system call is essentially a way for your program to interact with the Operating System. To make a system call in MIPS, we use a special instruction called `syscall`. To print out an integer, we need to pass two arguments to `syscall`. The first argument specifies what we want `syscall` to do (in this case, print an integer). To specify that we want to print an integer, we set `$v0` to `1`. The integer that we want to print out is passed as an argument via `a0`.

In C, we are used to functions looking like `afficher(s0)`. In MIPS, we cannot pass arguments in this way. To pass an argument, we need to place it in an argument register (`$a0`-`$a3`). When the function executes, it will look in these registers for the arguments. The first argument should be placed in `$a0`, the second in `$a1`, etc.

Next, let's terminate our program! This also requires a `syscall`!

```mips
addi $v0, $0, 10 # argument to syscall to terminate
syscall          # execute 'terminate'
```
And there you have it! Here's our full program!

```mips
.data
n: .word 9

.text
main:
    add  $s0, $0, $0      # curr_fib = 0
    addi $s1, $0,  1      # next_fib = 1
    la $t0, n             # load the address of the label n
    lw $t0, 0($t0)        # get the value that is stored at the address denoted by the label n
WHILE:
    beq  $t0, $0,  ELIHW  # exit loop once we have completed n iterations     
    add  $t2, $s0, $s1    # new_fib  = curr_fib + next_fib;
    add  $s0, $s1, $0     # curr_fib = next_fib;
    add  $s1, $t2, $0     # next_fib = new_fib;     
    addi $t0, $t0, -1     # decrement counter
    j WHILE               # loop
ELIHW:
    addi $v0,  $0, 1      # argument to syscall to execute print integer
    addi $a0, $s0, 0      # argument to syscall, the value to be printed
    syscall               # execute 'print integer'
    addi $v0, $0, 10      # argument to syscall to terminate
    syscall               # execute 'terminate'
```

