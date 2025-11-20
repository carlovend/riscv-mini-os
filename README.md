# RISC-V Mini OS

**Educational microkernel in C — built step by step following *Operating System in 1000 Lines*.**

> 📚 **Read the full documentation and tutorial:**
> [**https://carlovend.github.io/riscv-mini-os/**](https://carlovend.github.io/riscv-mini-os/)

---

## 📖 Overview
This project documents the creation of a small kernel for RISC-V, starting from scratch. The goal is clarity and creating minimal components to demonstrate real operating system mechanisms.

Inspired by:
* *Operating System in 1000 Lines*
* *Operating Systems: Three Easy Pieces*

## 📘 Roadmap

This roadmap reflects the step-by-step implementation documented on the website.

* **[M0 – Setup & Toolchain](https://carlovend.github.io/riscv-mini-os/m0_setup.html)**
    * Environment setup, QEMU, and cross-compiler.
* **[M1 – Boot & Linker Script](https://carlovend.github.io/riscv-mini-os/m1_boot.html)**
    * OpenSBI boot, stack setup, and `.bss` zeroing.
* **[M2 – UART, printf & LibC](https://carlovend.github.io/riscv-mini-os/m2-printf.html)**
    * Minimal C library implementation and formatted output.
* **[M3 – Exception Handling & Panic](https://carlovend.github.io/riscv-mini-os/m3-exceptions.html)**
    * Trap handler, `kernel_entry` assembly, and panic macros.
* **[M4 – Memory: The Page Allocator](https://carlovend.github.io/riscv-mini-os/m4-memory.html)**
    * Physical memory management and bump allocator.
* **M5 – Paging & TLB**
    * Virtual memory mapping and page tables.
* **M6 – Processes & Context Switch**
    * Process Control Block (PCB) and scheduler.
* **M7 – System Calls**
    * User/Kernel boundary and ABI.
* **M8 – File System & Shell**
    * Simple file system and user interaction.

---

## 💬 About
This project is part of my learning path in low-level systems and cybersecurity.
