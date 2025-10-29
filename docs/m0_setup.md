# M0 – Setting Up the Development Environment

## 🎯 Goal
Set up a cross-compilation toolchain and emulator for building a bare-metal RISC-V kernel.

## ⚙️ Steps Followed
Based on the official [Operating System in 1000 Lines guide](https://operating-system-in-1000-lines.vercel.app/en/01-setting-up-development-environment):

1. Installed `riscv64-unknown-elf-gcc` (cross-compiler)  
   → Verified with `riscv64-unknown-elf-gcc --version`
2. Installed `qemu-system-riscv32`  
   → Verified with `qemu-system-riscv32 --version`
3. Cloned the starter code and ran:
   ```bash
   make run
