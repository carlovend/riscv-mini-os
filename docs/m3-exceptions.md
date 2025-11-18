# M3 – Kernel Panic & Exception Handling

## Objective
To implement the kernel’s safety and trap-handling mechanisms:
- a `PANIC` macro for fatal errors,
- a complete exception (trap) handler for CPU events such as system calls, page faults, and illegal instructions.

---

# 1. Kernel Panic — The Emergency Brake

A **kernel panic** is the operating system’s safety mechanism for unrecoverable internal errors.  
It is the kernel-level equivalent of an application crash (such as a segmentation fault).

Since **nothing exists above the kernel** to recover from such failures, the kernel must:

1. **Print a diagnostic message** containing the filename and line number.  
2. **Freeze the system** by entering an infinite loop (`while (1) {}`) to prevent further data corruption.

## The PANIC Macro

```c
#define PANIC(fmt, ...)     do {         printf("PANIC: %s:%d: " fmt "\n", __FILE__, __LINE__, ##__VA_ARGS__);         while (1) {}     } while (0)
```

---

# 2. The Assembly Bridge: `kernel_entry`

When the CPU triggers an exception, we **cannot** jump directly to a C function.  
The interrupted program’s state must first be saved, and the stack must be placed in a safe, known configuration.

The job of `kernel_entry` is to:

- Save all registers
- Call the C handler
- Restore registers and return using `sret`

---

## Act 1: Saving the Context

Key steps:

- `csrw sscratch, sp` – save the original stack pointer  
- `addi sp, sp, -4 * 31` – allocate space for 31 registers  
- `sw ra ... sw s11` – store all registers  
- `csrr a0, sscratch` – recover original SP and save it as final slot  

---

## Act 2: Call the C Handler

- `mv a0, sp` – pass pointer to trap frame  
- `call handle_trap`

---

## Act 3: Restore the Context

- `lw` all registers back  
- `lw sp` original sp  
- `sret` return from exception  

---

# 3. The trap_frame Structure & CSRs

```c
struct trap_frame {
    uint32_t ra;
    uint32_t gp;
    uint32_t tp;
    uint32_t t0;
    // ...
    uint32_t s11;
    uint32_t sp;
} __attribute__((packed));
```

CSR macros:

```c
#define READ_CSR(reg) ({ unsigned long x; asm volatile("csrr %0, " #reg : "=r"(x)); x; })
#define WRITE_CSR(reg, v) asm volatile("csrw " #reg ", %0" :: "r"(v))
```

---

# 4. C Trap Handler

```c
void handle_trap(struct trap_frame *f) {
    uint32_t scause = READ_CSR(scause);
    uint32_t sepc   = READ_CSR(sepc);
    uint32_t stval  = READ_CSR(stval);

    PANIC("unexpected trap scause=%x, sepc=%x, stval=%x",
          scause, sepc, stval);
}
```

---

# Summary

This module introduces:

- Kernel panic
- Full trap entry/exit flow
- CSR helpers
- C-level trap handling

