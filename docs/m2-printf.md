# M2 – UART, `printf` & LibC

## Objective
To implement a minimal C environment (`common.h`) and a formatted output function (`printf`). In a bare‑metal kernel, we have no C standard library. We must create our own essential functions for string formatting, memory manipulation, and basic types before we can proceed.

---

## 1. The `common.h` Survival Kit

Before we can write `printf`, we must first define the basic types and macros that C code normally gets from the standard library. This file, `common.h`, will be our "survival kit" included in all kernel files.

### Defining Standard Types

We cannot rely on `int` or `long` to have a specific size, so we must define **fixed‑width integer types**. We also create clear aliases for memory addresses.

| Typedef | Purpose |
|--------|---------|
| `typedef int bool;` | Simple boolean type. |
| `typedef unsigned char uint8_t;` | 8‑bit unsigned integer. |
| `typedef unsigned short uint16_t;` | 16‑bit unsigned integer. |
| `typedef unsigned int uint32_t;` | 32‑bit unsigned integer. |
| `typedef unsigned long long uint64_t;` | 64‑bit unsigned integer. |
| `typedef uint32_t size_t;` | Type used for memory sizes. |
| `typedef uint32_t paddr_t;` | Physical address alias. |
| `typedef uint32_t vaddr_t;` | Virtual address alias. |

We also define basic constants normally expected:

```
#define true 1
#define false 0
#define NULL ((void *) 0)
```

### Leveraging Compiler Built‑ins

Instead of writing architecture‑specific helpers, we use compiler intrinsics:

```
#define align_up(value, align) __builtin_align_up(value, align)
#define offsetof(type, member) __builtin_offsetof(type, member)
#define va_list __builtin_va_list
```

These let us handle alignment, offsets, and variadic functions.

---

## 2. Implementing `printf`

With `common.h` in place, we can implement `printf`.  
This is a **variadic function**, meaning it can accept a variable number of arguments.

### Variadic Function Macros

- `va_list vargs;` — declares a special iterator for extra arguments.
- `va_start(vargs, fmt);` — initializes the list.
- `va_arg(vargs, type);` — extracts the next argument of the given type.
- `va_end(vargs);` — cleans up.

### `printf.c` Source Code

```c
#include "common.h"

// Forward declaration for putchar
void putchar(char ch);

void printf(const char *fmt, ...) {
    va_list vargs;
    va_start(vargs, fmt);

    while (*fmt) {
        if (*fmt == '%') {
            fmt++; // Skip '%'
            switch (*fmt) {
                case '\0': // '%' at end
                    putchar('%');
                    goto end;

                case '%': // Literal '%'
                    putchar('%');
                    break;

                case 's': { // String
                    const char *s = va_arg(vargs, const char *);
                    while (*s) {
                        putchar(*s++);
                    }
                    break;
                }

                case 'd': { // Decimal
                    int value = va_arg(vargs, int);
                    unsigned int magnitude = value;

                    if (value < 0) {
                        putchar('-');
                        magnitude = -magnitude;
                    }

                    unsigned int divisor = 1;
                    while (magnitude / divisor > 9)
                        divisor *= 10;

                    while (divisor > 0) {
                        putchar('0' + magnitude / divisor);
                        magnitude %= divisor;
                        divisor /= 10;
                    }
                    break;
                }

                case 'x': { // Hexadecimal
                    unsigned int value = va_arg(vargs, unsigned int);
                    for (int i = 7; i >= 0; i--) {
                        unsigned int nibble = (value >> (i * 4)) & 0xf;
                        putchar("0123456789abcdef"[nibble]);
                    }
                    break;
                }
            }
        } else {
            putchar(*fmt);
        }
        fmt++;
    }

end:
    va_end(vargs);
}
```

---

# 3. Dissecting the `printf` Logic

The core of the implementation is a loop that walks through the format string (`fmt`) one character at a time.

## 1. Normal Characters vs. `%`

- **If the character is NOT `%`**:  
  We print it directly:

  ```
  putchar(*fmt);
  ```

- **If the character *is* `%`**:  
  This signals a format specifier.  
  We increment `fmt` and handle the specifier with a `switch`.

---

## 2. Handling Format Specifiers

### `%%`
Prints a literal `%`.

---

### `%s` — String

```
const char *s = va_arg(vargs, const char *);
```

We print all characters until we hit the terminating `\0`.

---

### `%d` — Decimal

```
int value = va_arg(vargs, int);
```

Steps:

1. Handle negative values.  
2. Find the largest power of 10.  
3. Extract digits from left to right.

---

### `%x` — Hexadecimal

```
unsigned int value = va_arg(vargs, unsigned int);
```

Loop 8 times (32‑bit hex), printing one nibble at a time.

---

## 3. Final LibC Prototypes

```
void *memset(void *buf, char c, size_t n);
void *memcpy(void *dst, const void *src, size_t n);
char *strcpy(char *dst, const char *src);
int strcmp(const char *s1, const char *s2);
void printf(const char *fmt, ...);
```

---

## Lessons Learned

With `common.h` and a custom `printf`, we now have the foundation of a **kernel‑space C library**, enabling:

- predictable fixed‑width types  
- compiler‑powered helpers for low‑level tasks  
- formatted debugging output over UART  

This logging ability is essential for the next step: **exception handling and kernel panics**.
