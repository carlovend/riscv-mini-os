# Appendix — How `printf` Prints Numbers (%d)

This appendix explains one of the fundamental concepts behind low-level I/O functions: how a single integer like `345` becomes the sequence of printable characters `'3'`, `'4'`, `'5'`.

---

## 1. The Core Problem: Numbers vs Characters

The CPU does **not** see the number `345` as three symbols.  
It sees it as a single integer value stored in binary.

The only printing function we typically have at low level is:

```c
putchar(char ch);
```

This function can print **one character at a time**, not an entire number.  
Therefore, we must convert an integer (a *quantity*) into individual *characters*.

Goal: convert the number `345` into the characters `'3'`, `'4'`, `'5'`, and print them individually.

---

## 2. The Strategy: Extracting Digits from Left to Right

Getting the **last digit** is trivial:

```
345 % 10 = 5
```

But `printf` must print digits **from left to right**, so we need the **first** digit first.  
To achieve this, the algorithm uses two phases:

1. **Phase 1 — Find the correct divisor** (hundreds, thousands, etc.)
2. **Phase 2 — Peel off digits from left to right**

---

## 3. Phase 1: Finding the Divisor

To print `345`, we first need to know that the leftmost digit is in the **hundreds** place.  
We find this using a loop:

```c
unsigned int divisor = 1;
while (magnitude / divisor > 9)
    divisor *= 10;
```

### Example Trace (magnitude = 345)

| Step | divisor | magnitude/divisor | Condition (>9?) | Action |
|------|---------|-------------------|------------------|--------|
| Start | 1 | 345 | Yes | divisor = 10 |
| Loop 2 | 10 | 34 | Yes | divisor = 100 |
| Loop 3 | 100 | 3 | No | stop |

**Final divisor = 100**  
This lets us extract the first digit:

```
345 / 100 = 3
```

---

## 4. Phase 2: Peeling and Printing Digits

Once we know the divisor, we can peel digits from left to right:

```c
while (divisor > 0) {
    putchar('0' + magnitude / divisor);
    magnitude %= divisor;
    divisor /= 10;
}
```

### Example Trace (magnitude = 345, divisor = 100)

#### Loop 1 — divisor = 100
- `345 / 100 = 3` → prints `'3'`
- `magnitude = 345 % 100 = 45`
- `divisor = 100 / 10 = 10`

#### Loop 2 — divisor = 10
- `45 / 10 = 4` → prints `'4'`
- `magnitude = 45 % 10 = 5`
- `divisor = 10 / 10 = 1`

#### Loop 3 — divisor = 1
- `5 / 1 = 5` → prints `'5'`
- `magnitude = 5 % 1 = 0`
- `divisor = 1 / 10 = 0`

Loop stops.  
Final output: **3 4 5**

---

## 5. Summary

- Computers store numbers as binary quantities, not multiple characters.  
- To print a number, we must convert it into characters.  
- We find the most significant digit by finding a divisor (1, 10, 100, 1000...).  
- Then we peel digits off from left to right using division and modulo.  
- This is the core of how `printf("%d")` works internally.

