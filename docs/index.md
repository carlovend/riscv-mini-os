---
layout: home
title: "RISC-V Mini OS"
nav_order: 1
---

# RISC-V Mini OS
**Educational microkernel in C — built step by step following OS in 1000 Lines**

This project documents the creation of a small kernel for RISC-V, starting from scratch.  
The goal is clarity and minimal components that demonstrate real operating system mechanisms.

---

## Roadmap and Articles

This is the linear progression of the project.

1. **[M0 – Setup & Toolchain](m0_setup.md)**  
   Set up the cross-compiler, QEMU, and build environment.

2. **[M1 – Boot & Linker Script](m1_boot.md)**  
   Implement the boot process from OpenSBI, set up the stack, and zero the `.bss` section.

3. **[M2 – UART, `printf` & LibC](m2-printf.md)**  
   Implement a `putchar` driver, a `printf` variadic function, and a minimal `common.h` library.

4. **[M3 – Exception Handling & `panic`](m3-exceptions.md)**  
   Implement the `panic` macro and the full S-Mode exception trap flow (asm entry, `trap_frame`, CSR macros).

5. **[M4 – Memory: The Page Allocator](m4-memory.md)**  
   Define the free memory region in the linker script and implement the `alloc_pages` bump allocator.

6. **[M5 – Processes & Context Switch](m5-processes.md)**  
   Define a Process Control Block (PCB), a round-robin scheduler, and handle trap returns.

7. **[M6 – System Calls](m7_syscalls.md)**  
   Implement the user-mode/kernel-mode boundary, validation, and ABI.

8. **[M7 – Minimal FS](m8_filesystem.md)**  
   Block management, inodes, and integration with a mini-shell.

---

## Appendices

Technical deep-dives on specific topics.

- **[Appendix — OpenSBI & Boot Deep Dive](appendix-opensbi.md)**  
  Details on OpenSBI, the QEMU boot flow, and disassembly analysis.

- **[Appendix — How `printf` prints numbers](appendix-printf-int.md)**  
  Explanation of the algorithm used to convert an `int` into a character string (division and modulo).

- **[Appendix — Fragmentation problem](appendix-fragmentation.md)**  
  Explanation of internal vs. external fragmentation and why paging solves the issue.

---

## Quickstart

```bash
# Clone & build
git clone https://github.com/carlovend/riscv-mini-os.git
cd riscv-mini-os
make

# Run on QEMU (RISC-V 32-bit, virt machine)
./run.sh      # or: make run
