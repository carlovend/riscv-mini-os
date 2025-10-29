# M1 – Boot & Linker Script

## Objective
Implement the kernel boot process — from the moment OpenSBI transfers control to your code in S-mode, through setting up the stack and zeroing memory, to finally entering C execution inside `kernel_main()`.

---

## Understanding the Boot Sequence

When you power on a RISC-V system (or start it in QEMU), the CPU begins execution in **Machine Mode (M-mode)** — the most privileged level of the RISC-V architecture.  
At this stage, **OpenSBI** runs first. It is equivalent to a BIOS or UEFI on x86 systems.

### What OpenSBI Does
1. Executes in M-mode right after reset.  
2. Initializes basic hardware (console, CPU state).  
3. Drops privileges to **Supervisor Mode (S-mode)** and jumps to your kernel’s entry address.  
4. Remains available to service privileged requests from the kernel via the **SBI interface**.

From this point onward, your code (the kernel) runs in **S-mode**, and OpenSBI acts as a low-level firmware layer you no longer have to manage directly.

---

## CPU Modes and Privileges

| Mode | Role | Description |
|------|------|-------------|
| **M-mode** | Machine Mode | Highest privilege; runs OpenSBI. |
| **S-mode** | Supervisor Mode | Kernel level; your OS executes here. |
| **U-mode** | User Mode | Applications run here with limited privileges. |

This separation ensures **protection** (user code cannot interfere with kernel code) and **stability** (the kernel arbitrates resources).

---

## The Role of the Linker Script

The linker script defines how sections of the compiled object files (`.text`, `.data`, `.bss`) are laid out in memory.  
Since there is no operating system or default linker configuration in bare-metal environments, you must define this layout manually.

### Example: `linker.ld`

``` ld
ENTRY(boot)

SECTIONS {
    . = 0x80200000;

    .text : {
        KEEP(*(.text.boot));
        *(.text .text.*);
    }

    .rodata : ALIGN(4) {
        *(.rodata .rodata.*);
    }

    .data : ALIGN(4) {
        *(.data .data.*);
    }

    .bss : ALIGN(4) {
        __bss = .;
        *(.bss .bss.* .sbss .sbss.*);
        __bss_end = .;
    }

    . = ALIGN(4);
    . += 128 * 1024; /* Reserve 128KB for the stack */
    __stack_top = .;
}

```

### Key Points
- The kernel starts at address `0x80200000`.  
- `.text.boot` is explicitly kept at the beginning — this contains the `boot()` function.  
- The `__bss` and `__bss_end` symbols mark the start and end of the uninitialized data section, which must be cleared at runtime.  
- The stack is allocated after `.bss`, and its top address (`__stack_top`) will be used to initialize the stack pointer.

---

## Boot Code in C and Inline Assembly

Your kernel’s entry point is the `boot()` function.  
It must:
1. Set the stack pointer.  
2. Jump into C code (`kernel_main`).  

Because the stack does not yet exist when `boot()` runs, the function must be **naked** (no compiler-generated prologue/epilogue) and placed in a **custom section** so that it appears at the beginning of the kernel image.

### Source

```c
typedef unsigned char  uint8_t;
typedef unsigned int   uint32_t;
typedef uint32_t       size_t;

extern char __bss[], __bss_end[], __stack_top[];

void *memset(void *buf, char c, size_t n) {
    uint8_t *p = (uint8_t *) buf;
    while (n--) *p++ = c;
    return buf;
}

void kernel_main(void) {
    memset(__bss, 0, (size_t) __bss_end - (size_t) __bss);
    for (;;); // Infinite loop
}

__attribute__((section(".text.boot")))
__attribute__((naked))
void boot(void) {
    __asm__ __volatile__(
        "mv sp, %[stack_top]\n" // Set up stack pointer
        "j kernel_main\n"       // Jump to kernel_main()
        :
        : [stack_top] "r" (__stack_top)
    );
}

```

### Explanation
- `section(".text.boot")`  
  Ensures the boot function is placed at the very start of the kernel image.  
- `naked`  
  Prevents the compiler from emitting any setup code that would use an uninitialized stack.  
- Inline assembly moves the address of `__stack_top` into the stack pointer (`sp`) and jumps unconditionally to `kernel_main`.

---

## The Kernel Entry Point

`kernel_main()` is the first C function executed with a valid stack.  
Its initial responsibility is to clear the `.bss` section to zero, as required by the C standard.

```c
memset(__bss, 0, (size_t) __bss_end - (size_t) __bss);
```

The linker-defined symbols `__bss` and `__bss_end` are used to determine which range of memory must be cleared.  
Since there are no uninitialized globals yet, this loop currently does nothing, but it is crucial for future stages.

---

## Verifying the Execution

When disassembling `kernel.elf`, you should see:

```c 
80200000 <boot>:
    mv sp, 0x80220000
    j  kernel_main

80200024 <kernel_main>:
    ...
```

At this point, OpenSBI successfully handed control to your kernel, the stack was initialized, and your kernel is running in Supervisor mode.

---

## Lessons Learned

- **OpenSBI** handles all M-mode responsibilities so your kernel can safely run in S-mode.  
- **The linker script** defines the memory map and provides symbolic addresses for stack and sections.  
- **Inline assembly** bridges the gap between low-level hardware setup and C code.  
- **Memory initialization** is essential for predictable kernel behavior.  
- The system is now ready to implement **virtual memory and paging**.

---

## Extras
- [Appendix — OpenSBI and Boot Deep Dive](appendix-opensbi.md)


## References
- [Operating System in 1000 Lines – Boot](https://operating-system-in-1000-lines.vercel.app/en/02-boot)  
- [RISC-V Privileged Architecture Specification](https://riscv.org/technical/specifications/)  
- [Operating Systems: Three Easy Pieces – Address Spaces](https://pages.cs.wisc.edu/~remzi/OSTEP/)  
