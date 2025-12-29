
# M6 – Processes & Context Switch

## Objective
Transform the kernel from a single-task sequential executor into a **multitasking operating system** capable of managing multiple processes. In this module we implement:

- The **PCB** (Process Control Block)
- The **Context Switch** mechanism
- A cooperative **Round-Robin Scheduler**
- Secure **Kernel Stack** management using the `sscratch` register

---

## 1. Process Control Block (PCB)

A process is an instance of an application with its own execution context and resources. To manage processes, the OS maintains a **Process Control Block (PCB)** containing all essential state.

### Structure Definition

```c
#define PROCS_MAX 8
#define PROC_UNUSED   0
#define PROC_RUNNABLE 1

struct process {
    int pid;
    int state;
    vaddr_t sp;
    uint8_t stack[8192];
};

struct process procs[PROCS_MAX];
```

### Key Insight

The most important field is `sp`. The kernel does **not** store registers individually inside the PCB.  
All CPU state is pushed onto the process's stack during a context switch.  
Restoring `sp` restores the entire execution context.

---

## 2. Context Switch

The **context switch** allows the CPU to suspend one process and resume another by saving and restoring register state.

```c
__attribute__((naked)) void switch_context(uint32_t *prev_sp, uint32_t *next_sp) {
    __asm__ __volatile__(
        "addi sp, sp, -13 * 4\n"
        "sw ra,  0  * 4(sp)\n"
        "sw s0,  1  * 4(sp)\n"
        "sw s1,  2  * 4(sp)\n"
        "sw s2,  3  * 4(sp)\n"
        "sw s3,  4  * 4(sp)\n"
        "sw s4,  5  * 4(sp)\n"
        "sw s5,  6  * 4(sp)\n"
        "sw s6,  7  * 4(sp)\n"
        "sw s7,  8  * 4(sp)\n"
        "sw s8,  9  * 4(sp)\n"
        "sw s9,  10 * 4(sp)\n"
        "sw s10, 11 * 4(sp)\n"
        "sw s11, 12 * 4(sp)\n"
        "sw sp, (a0)\n"
        "lw sp, (a1)\n"
        "lw ra,  0  * 4(sp)\n"
        "lw s0,  1  * 4(sp)\n"
        "lw s1,  2  * 4(sp)\n"
        "lw s2,  3  * 4(sp)\n"
        "lw s3,  4  * 4(sp)\n"
        "lw s4,  5  * 4(sp)\n"
        "lw s5,  6  * 4(sp)\n"
        "lw s6,  7  * 4(sp)\n"
        "lw s7,  8  * 4(sp)\n"
        "lw s8,  9  * 4(sp)\n"
        "lw s9,  10 * 4(sp)\n"
        "lw s10, 11 * 4(sp)\n"
        "lw s11, 12 * 4(sp)\n"
        "addi sp, sp, 13 * 4\n"
        "ret\n"
    );
}
```

---

## 3. Process Creation

A new process starts with a **fabricated stack** that mimics a paused context.

```c
struct process *create_process(uint32_t pc) {
    struct process *proc = NULL;
    int i;

    for (i = 0; i < PROCS_MAX; i++) {
        if (procs[i].state == PROC_UNUSED) {
            proc = &procs[i];
            break;
        }
    }

    uint32_t *sp = (uint32_t *)&proc->stack[sizeof(proc->stack)];

    *--sp = 0;  // s11
    *--sp = 0;  // s10
    *--sp = 0;  // s9
    *--sp = 0;  // s8
    *--sp = 0;  // s7
    *--sp = 0;  // s6
    *--sp = 0;  // s5
    *--sp = 0;  // s4
    *--sp = 0;  // s3
    *--sp = 0;  // s2
    *--sp = 0;  // s1
    *--sp = 0;  // s0
    *--sp = pc; // ra

    proc->pid = i + 1;
    proc->state = PROC_RUNNABLE;
    proc->sp = (uint32_t)sp;

    return proc;
}
```

---

## 4. Scheduler (Round-Robin)

```c
void yield(void) {
    struct process *next = idle_proc;

    for (int i = 0; i < PROCS_MAX; i++) {
        struct process *proc = &procs[(current_proc->pid + i) % PROCS_MAX];
        if (proc->state == PROC_RUNNABLE && proc->pid > 0) {
            next = proc;
            break;
        }
    }

    if (next == current_proc)
        return;

    __asm__ __volatile__(
        "csrw sscratch, %[sscratch]\n"
        :
        : [sscratch] "r" ((uint32_t)&next->stack[sizeof(next->stack)])
    );

    struct process *prev = current_proc;
    current_proc = next;
    switch_context(&prev->sp, &next->sp);
}
```

---

## 5. Kernel Stack & sscratch

The `sscratch` CSR enables **safe trap handling** by switching to a clean kernel stack.

```c
__asm__ __volatile__(
    "csrrw sp, sscratch, sp\n"
    "addi sp, sp, -4 * 31\n"
    "sw ra,  4 * 0(sp)\n"
    "csrr a0, sscratch\n"
    "sw a0,  4 * 30(sp)\n"
    "addi a0, sp, 4 * 31\n"
    "csrw sscratch, a0\n"
    "mv a0, sp\n"
    "call handle_trap\n"
);
```

---

## Summary

This module introduces the core mechanisms required for multitasking on RISC-V:
- Process Control Blocks
- Context switching
- Round-Robin scheduling
- Secure dual-stack trap handling
