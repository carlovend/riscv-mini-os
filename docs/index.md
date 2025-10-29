<!-- Custom CSS -->
<link rel="stylesheet" href="assets/custom.css">

<div class="hero">
  <h1>RISC‑V Mini OS</h1>
  <p class="subtitle">From bare metal to a working kernel — concise, auditable, and educational.</p>
  <div class="badges">
    <img src="https://img.shields.io/badge/lang-C-blue" alt="C">
    <img src="https://img.shields.io/badge/arch-RISC--V-0A7BDC" alt="RISC-V">
    <img src="https://img.shields.io/badge/QEMU-virt-informational" alt="QEMU virt">
    <img src="https://img.shields.io/badge/status-active-success" alt="status">
    <img src="https://img.shields.io/badge/license-MIT-green" alt="MIT">
  </div>
  <div class="cta">
    <a class="btn primary" href="../">View on GitHub</a>
    <a class="btn" href="m0_setup.md">Start here (M0)</a>
    <a class="btn" href="m1_boot.md">Read M1</a>
    <a class="btn" href="appendix_opensbi_boot.md">OpenSBI Appendix</a>
  </div>
</div>

---

## Roadmap

<div class="grid">
  <a class="card done" href="m0_setup.md">
    <h3>M0 — Setup & Toolchain</h3>
    <p>Cross compiler, QEMU, first run.</p>
    <span class="tag">done</span>
  </a>
  <a class="card done" href="m1_boot.md">
    <h3>M1 — Boot & Linker Script</h3>
    <p>From OpenSBI to C entry, stack, .bss.</p>
    <span class="tag">done</span>
  </a>
  <a class="card" href="m2_paging.md">
    <h3>M2 — Paging & TLB</h3>
    <p>satp, page tables, identity map, faults.</p>
    <span class="tag">next</span>
  </a>
  <a class="card" href="m3_processes.md">
    <h3>M3 — Processes & Context Switch</h3>
    <p>PCB, scheduler (RR), trap return.</p>
  </a>
  <a class="card" href="m4_syscalls.md">
    <h3>M4 — Syscalls</h3>
    <p>ABI, validation, user↔kernel boundary.</p>
  </a>
  <a class="card" href="m5_filesystem.md">
    <h3>M5 — Minimal FS</h3>
    <p>Blocks, inodes, tiny shell integration.</p>
  </a>
</div>

---

## What you’ll learn
- How OpenSBI hands off from M‑mode to your kernel in S‑mode.
- How a linker script controls memory layout and section symbols.
- How to bring up a minimal kernel in C (stack, .bss, entry).
- Fundamentals of paging, traps, scheduling, syscalls, and I/O.

> Each milestone includes: a concise explanation, minimal code, tests, and a short demo/log.

---

## Quickstart

```bash
# Clone & build
git clone https://github.com/<YOUR-USER>/riscv-mini-os.git
cd riscv-mini-os
make

# Run on QEMU (RISC-V 32-bit, virt machine)
./run.sh      # or: make run
```

If you enter the QEMU monitor by accident, press `Ctrl+a` then `c` to toggle back.

---

## Articles
- M0 — Setup & Toolchain → <a href="m0_setup.md">read</a><br/>
- M1 — Boot & Linker Script → <a href="m1_boot.md">read</a><br/>
- Appendix — OpenSBI & Boot Deep Dive → <a href="appendix-opensbi.md">read</a>

---

## About
This project is part of a public learning path in systems and security engineering. The goal is clarity over cleverness: tiny, inspectable components with just enough code to demonstrate real mechanisms.
