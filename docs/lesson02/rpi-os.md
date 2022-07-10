## 2.1: Processor initialization 

In this lesson, we are going to work more closely with the ARM processor. It has some essential features that can be utilized by the OS. The first such feature is called "Exception levels".

### Exception levels

Each ARM processor that supports ARM.v8 architecture has 4 exception levels. You can think about an exception level (or `EL` for short) as a processor execution mode in which only a subset of all operations and registers is available. The least privileged exception level is level 0. When processor operates at this level, it mostly uses only general purpose registers (X0 - X30) and stack pointer register (SP). EL0 also allows using `STR` and `LDR` commands to load and store data to and from memory and a few other instructions commonly used by a user program.

An operating system should deal with exception levels because it needs to implement process isolation. A user process should not be able to access other process's data. To achieve such behavior, an operating system always runs each user process at EL0. Operating at this exception level a process can only use it's own virtual memory and can't access any instructions that change virtual memory settings. So, to ensure process isolation, an OS need to prepare separate virtual memory mapping for each process and put the processor into EL0 before transferring execution to a user process.

操作系统本身运行在EL1。当运行在这个异常等级时处理器可以访问寄存器，其可以配置虚拟内存的设置和一些系统寄存器。Raspberry Pi OS also will be using EL1.

We are not going to use exceptions levels 2 and 3 a lot, but I just want to briefly describe them so you can get an idea why they are needed. 

EL2被用于当你想要使用一个超级权限时。在这种情况下，主机操作系统运行在EL2，且用户操作系统只能使用EL1。这允许主机系统孤立用户OSes in a similar way how OS isolates user processes.

EL3 is used for transitions from ARM "Secure World" to "Insecure world". This abstraction exist to provide full hardware isolation between the software running in two different "worlds". Application from an "Insecure world" can in no way access or modify information (both instruction and data) that belongs to "Secure world", and this restriction is enforced at the hardware level. 

### Debugging the kernel

Next thing that I want to do is to figure out which Exception level we are currently using. But when I tried to do this, I realized that the kernel could only print some constant string on a screen, but what I need is some analog of [printf](https://en.wikipedia.org/wiki/Printf_format_string) function. With `printf` I can easily display values of different registers and variables. Such functionality is essential for the kernel development because you don'tIn this case host operating system runs at EL2 and guest operating systems can only use EL 1. have any other debugger support and `printf` becomes the only mean by which you can figure out what is going on inside your program.

For the RPi OS I decided not to reinvent the wheel and use one of  [existing printf implementations](http://www.sparetimelabs.com/tinyprintf/tinyprintf.php) This function consists mostly from string manipulations and is not very interesting from a kernel developer point of view. The implementation that I used is very small and don't have external dependencies, that allows it to be easily integrated into the kernel. The only thing that I have to do is to define `putc`  function that can send a single character to the screen. This function is defined [here](https://github.com/s-matyukevich/raspberry-pi-os/blob/master/src/lesson02/src/mini_uart.c#L59) and it just uses already existing `uart_send` function. Also, we need to initialize the `printf` library and specify the location of the `putc` function. This is done in a single [line of code](https://github.com/s-matyukevich/raspberry-pi-os/blob/master/src/lesson02/src/kernel.c#L8).

### 找到现在的异常等级

Now, when we are equipped with the `printf` function, we can complete our original task: figure out at which exception level the OS is booted. A small function that can answer this question is defined [here](https://github.com/s-matyukevich/raspberry-pi-os/blob/master/src/lesson02/src/utils.S#L1) and looks like this.

```
.globl get_el
get_el:
    mrs x0, CurrentEL
    lsr x0, x0, #2
    ret
```

这里我们使用 `mrs` 指令去从 `CurrentEL` 系统寄存器读取值进 `x0` 寄存器。然后我们右移2位，这是因为在 `CurrentEL` 寄存器中的前两位是保留的且总是0值。且最终在寄存器 `x0` 中，我们有一个整数值指出现在的异常等级。Now the only thing that is left is to display this value, like [this](https://github.com/s-matyukevich/raspberry-pi-os/blob/master/src/lesson02/src/kernel.c#L10).

```
    int el = get_el();
    printf("Exception level: %d \r\n", el);
```

If you reproduce this experiment, you should see `Exception level: 3` on the screen.

### 改变现在的异常级别

在ARM架构中，需要一个已经运行在更高级别的软件的参与才能增加一个程序的异常级别。This makes a perfect sense: 要不然，任何程序将能逃出它被分配的EL且访问其它程序的数据。只有当一个异常被生成的时候，现在的EL才能被改变。

This can happen if a program executes some illegal instruction (for example, tries to access memory location at a nonexisting address, or tries to divide by 0) 
当然一个应用能运行`svc`指令来有目的的生成一个异常。
 Hardware generated interrupts are also handled as a special type of exceptions. 无论何时一个异常被生成，下列的步骤都会发生
  (In the description I am assuming that the exception is handled at EL `n`, were `n` could be 1, 2 or 3).

1. 现在的指令的地址被保存在`ELR_ELn`寄存器中(它被称为`Exception link register`)
2. 现在的处理器状态被保存在`SPSR_ELn`寄存器中 (`Saved Program Status Register`)
3. 一个异常处理函数被执行，做它需要做的任何工作。
4. 异常处理函数调用`eret`指令。这个指令从`SPSR_ELn`恢复处理器状态且从保存在`ELR_ELn`寄存器中保存的地址开始重新执行。

实际上，这个过程要复杂一点，因为异常处理函数也需要保存所有通用目的寄存器的状态且之后恢复回去， but we will discuss this process in details in the next lesson. For now, we need just to understand the process in general and remember the meaning of the `ELR_ELn` and `SPSR_ELn` registers.

一个需要知道的重要的事情是，异常处理函数没有义务返回到异常发生的位置。`ELR_ELn`和`SPSR_ELn`都是可写的且异常处理函数可以修改它们如果它想的话。
 We are going to use this technique to our advantage when we try to switch from EL3 to EL1 in our code.

### 转到EL1

严格地说，我们的操作系统没有义务切换到EL1，但是EL1对于我们来说是个自然的选择，因为这个级别
has just the right set of privileges to implement all common OS tasks. It also will be an interesting exercise to see how switching exceptions levels works in action. Let's take a look at the [source code that does this](https://github.com/s-matyukevich/raspberry-pi-os/blob/master/src/lesson02/src/boot.S#L17).

```
master:
    ldr    x0, =SCTLR_VALUE_MMU_DISABLED
    msr    sctlr_el1, x0        

    ldr    x0, =HCR_VALUE
    msr    hcr_el2, x0

    ldr    x0, =SCR_VALUE
    msr    scr_el3, x0

    ldr    x0, =SPSR_VALUE
    msr    spsr_el3, x0

    adr    x0, el1_entry        
    msr    elr_el3, x0

    eret                
```

像你可以看到的一样，代码大多数由配置某些寄存器组成。
 Now we are going to examine those registers one by one. In order to do this we first need to download [AArch64-Reference-Manual](https://developer.arm.com/docs/ddi0487/ca/arm-architecture-reference-manual-armv8-for-armv8-a-architecture-profile). This document contains the detailed specification of the `ARM.v8` architecture. 

#### SCTLR_EL1, System Control Register (EL1), Page 2654 of AArch64-Reference-Manual.

```
    ldr    x0, =SCTLR_VALUE_MMU_DISABLED
    msr    sctlr_el1, x0        
```

Here we set the value of the `sctlr_el1` system register. `sctlr_el1` is responsible for configuring different parameters of the processor, when it operates at EL1. 比如，它控制cache是否使能，对于我们来说最重要的是，MMU是否打开。 `sctlr_el1` is accessible from all exception levels higher or equal than EL1 (you can infer this from `_el1` postfix) 

`SCTLR_VALUE_MMU_DISABLED` constant is defined [here](https://github.com/s-matyukevich/raspberry-pi-os/blob/master/src/lesson02/include/arm/sysregs.h#L16) Individual bits of this value are defined like this:

* `#define SCTLR_RESERVED                  (3 << 28) | (3 << 22) | (1 << 20) | (1 << 11)` Some bits in the description of `sctlr_el1` register are marked as `RES1`. Those bits are reserved for future usage and should be initialized with `1`.
* `#define SCTLR_EE_LITTLE_ENDIAN          (0 << 25)` Exception [Endianness](https://en.wikipedia.org/wiki/Endianness). This field controls endianess of explicit data access at EL1. We are going to configure the processor to work only with `little-endian` format.
* `#define SCTLR_EOE_LITTLE_ENDIAN         (0 << 24)` Similar to previous field but this one controls endianess of explicit data access at EL0, instead of EL1. 
* `#define SCTLR_I_CACHE_DISABLED          (0 << 12)` Disable instruction cache. We are going to disable all caches for simplicity. You can find more information about data and instruction caches [here](https://stackoverflow.com/questions/22394750/what-is-meant-by-data-cache-and-instruction-cache).
* `#define SCTLR_D_CACHE_DISABLED          (0 << 2)` Disable data cache.
* `#define SCTLR_MMU_DISABLED              (0 << 0)` Disable MMU. MMU must be disabled until the lesson 6, where we are going to prepare page tables and start working with virtual memory.

#### HCR_EL2, Hypervisor Configuration Register (EL2), Page 2487 of AArch64-Reference-Manual. 

```
    ldr    x0, =HCR_VALUE
    msr    hcr_el2, x0
```

We are not going to implement our own [hypervisor](https://en.wikipedia.org/wiki/Hypervisor). Stil we need to use this register because, among other settings, 它控制在EL1时的执行状态. 执行状态必须是`AArch64`和非`AArch32`. This is configured [here](https://github.com/s-matyukevich/raspberry-pi-os/blob/master/src/lesson02/include/arm/sysregs.h#L22).

#### SCR_EL3, Secure Configuration Register (EL3), Page 2648 of AArch64-Reference-Manual.

```
    ldr    x0, =SCR_VALUE
    msr    scr_el3, x0
```

This register is responsible for configuring security settings. For example, it controls whether all lower levels are executed in "secure" or "nonsecure" state. It also controls execution state at EL2. [here](https://github.com/s-matyukevich/raspberry-pi-os/blob/master/src/lesson02/include/arm/sysregs.h#L26) we set that EL2  will execute at `AArch64` state, and all lower exception levels will be "non secure". 

#### SPSR_EL3, Saved Program Status Register (EL3), Page 389 of AArch64-Reference-Manual.

```
    ldr    x0, =SPSR_VALUE
    msr    spsr_el3, x0
```

This register should be already familiar to you - we mentioned it when discussed the process of changing exception levels. `spsr_el3`包含处理器状态, 其将会在我们执行`eret`指令后被恢复。
It is worth saying a few words explaining what processor state is. Processor state includes the following information:

* **Condition Flags** 这些标志包含先前被执行的操作的信息: whether the result was negative (N flag), zero (A flag), has unsigned overflow (C flag) or has signed overflow (V flag). 这些标志的值能被用在一个条件分支指令中。For example, `b.eq` instruction will jump to the provided label only if the result of the last comparison operation is equal to 0. The processor checks this by testing whether Z flag is set to 1.

* **Interrupt disable bits** 这些位允许使能/关闭不同类型的中断。

* Some other information, required to fully restore the processor execution state after an exception is handled.

当EL3出现异常时，通常`spsr_el3`被自动保存。
 However this register is writable, so we take advantage of this fact and manually prepare processor state. `SPSR_VALUE` is prepared [here](https://github.com/s-matyukevich/raspberry-pi-os/blob/master/src/lesson02/include/arm/sysregs.h#L35) and we initialize the following fields:

* `#define SPSR_MASK_ALL        (7 << 6)` 当我们改变EL到EL1后，所有的中断类型将被masked(或者关闭，其是一样的).
* `#define SPSR_EL1h        (5 << 0)` At EL1 we can either use our own dedicated stack pointer or use EL0 stack pointer. `EL1h` mode means that we are using EL1 dedicated stack pointer. 

#### ELR_EL3, Exception Link Register (EL3), Page 351 of AArch64-Reference-Manual.

```
    adr    x0, el1_entry        
    msr    elr_el3, x0

    eret                
```

`elr_el3` holds the address, to which we are going to return after `eret` instruction will be executed. Here we set this address to the location of `el1_entry` label.

### Conclusion

That is pretty much it: when we enter `el1_entry` function the execution should be already at EL1 mode. Go ahead and try it out! 

##### Previous Page

1.5 [Kernel Initialization: Exercises](../../docs/lesson01/exercises.md)

##### Next Page

2.2 [Processor initialization: Linux](../../docs/lesson02/linux.md)
