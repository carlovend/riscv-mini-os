# M4 – Memory Management: The Page Allocator

## Objective
To implement the kernel's first dynamic memory manager. We will define a large pool of free RAM (the heap) in the linker script and write a simple **Bump Allocator** that hands out memory in fixed 4KB blocks called **pages**.

---

## 1. Defining the Heap (Linker Script)

The kernel needs a dedicated region of memory to allocate from. Instead of hardcoding an address, we let the **linker** reserve a large block *after* all kernel sections.

Add the following to `kernel.ld`:

```ld
    . = ALIGN(4096);
    __free_ram = .;
    . += 64 * 1024 * 1024; /* 64MB */
    __free_ram_end = .;
```

### What this does:

- **ALIGN(4096)** – ensures the heap starts on a 4KB boundary.
- **__free_ram** – symbol marking the beginning of the heap.
- **64MB reserved** – we carve out a contiguous 64MB block for dynamic allocation.
- **__free_ram_end** – symbol marking the end of the heap.

Because this is controlled by the linker, we guarantee the heap never overlaps with kernel code, data, or the stack.

---

## 2. The Concept of Pages

Operating systems almost never allocate memory byte‑by‑byte.  
Instead, they use fixed‑size blocks called **pages**, normally:

- **4 KiB (4096 bytes)**

### Why pages?

- The RISC‑V MMU works in pages.
- Eliminates **external fragmentation** (the “Swiss-cheese problem”).
- Makes memory management much simpler and faster.

*(For a deeper explanation, see the Appendix — Fragmentation.)*

---

## 3. The Bump Allocator

This is the simplest physical memory allocator.

It works like a tape dispenser:

- Keep a pointer to the next free address.
- When asked for `n` pages, return the current pointer.
- Move the pointer forward by `n * PAGE_SIZE`.

### Source Code

```c
extern char __free_ram[], __free_ram_end[];

paddr_t alloc_pages(uint32_t n) {
    static paddr_t next_paddr = (paddr_t) __free_ram;
    paddr_t paddr = next_paddr;

    next_paddr += n * PAGE_SIZE;

    if (next_paddr > (paddr_t) __free_ram_end)
        PANIC("out of memory");

    memset((void *) paddr, 0, n * PAGE_SIZE);
    return paddr;
}
```

---

## 4. Code Analysis

### The “Static Cursor”

```c
static paddr_t next_paddr = (paddr_t) __free_ram;
```

`static` ensures the value persists across calls.  
This acts as the allocator’s *cursor* through free RAM.

### Allocation Steps

1. **paddr = next_paddr**  
   The address we will return.

2. **Increase pointer**  
   ```c
   next_paddr += n * PAGE_SIZE;
   ```

3. **Safety Check**  
   If we exceed `__free_ram_end`, the kernel is out of physical RAM → fatal error.

4. **Zero the Memory**  
   ```c
   memset((void *) paddr, 0, n * PAGE_SIZE);
   ```
   Ensures the allocated memory contains no leftover data.

---

## Summary

In this module we implemented:

- A 64MB kernel heap defined in the linker script  
- Page-based memory management  
- A simple bump allocator (`alloc_pages`)  
- Safety checks and zeroing of allocated memory  

This allocator will later support:

- Page tables
- User process memory
- Kernel stacks
- Allocating PCB structures
