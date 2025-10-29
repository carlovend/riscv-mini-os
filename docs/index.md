# RISC-V Mini OS
**Educational microkernel in C — built step by step following OS in 1000 Lines**

This project documents the creation of a small kernel for RISC-V, starting from scratch. The goal is clarity and creating minimal components to demonstrate real operating system mechanisms.

---

## Roadmap and Articles

This is the linear progression of the project.

1.  **[M0 – Setup & Toolchain](m0_setup.md)**
    * Set up the cross-compiler, QEMU, and build environment.

2.  **[M1 – Boot & Linker Script](m1_boot.md)**
    * Implement the boot process from OpenSBI, set up the stack, and zero the `.bss` section before C entry.

3.  **[M2 – Paging & TLB](m2_paging.md)**
    * Configure `satp`, create page tables, and handle page faults.

4.  **[M3 – Processes & Context Switch](m3_processes.md)**
    * Define a Process Control Block (PCB), a round-robin scheduler, and handle trap returns.

5.  **[M4 – System Calls](m4_syscalls.md)**
    * Implement the user-mode/kernel-mode boundary, validation, and ABI.

6.  **[M5 – Minimal FS](m5_filesystem.md)**
    * Block management, inodes, and integration with a mini-shell.

---

## Appendices

Technical deep-dives on specific topics.

* **[Appendix — OpenSBI & Boot Deep Dive](appendix-opensbi.md)**
    * Details on OpenSBI, the QEMU boot flow, and disassembly analysis.

---

## Quickstart

```bash
# Clone & build
git clone [https://github.com/](https://github.com/)<YOUR-USER>/riscv-mini-os.git
cd riscv-mini-os
make

# Run on QEMU (RISC-V 32-bit, virt machine)
./run.sh      # or: make run