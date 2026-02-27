# Topic 2.8 — Bitwise & Shift Operators

---

## 1. Conceptual Explanation

### What Are Bitwise and Shift Operators?

These operators work directly on the **binary representation** of integer values — operating bit by bit. They are applied to `byte`, `short`, `int`, `long`, and `char` types.

**Full list:**

| Category | Operators |
|---|---|
| Bitwise | `&` (AND), `\|` (OR), `^` (XOR), `~` (complement) |
| Shift | `<<` (left shift), `>>` (signed right shift), `>>>` (unsigned right shift) |

> `~` was covered in Topic 2.2. We revisit it here in its bitwise context. `&`, `|`, `^` were introduced in Topic 2.6 — here we go much deeper on their integer behavior.

---

### Binary Representation — Foundation

To understand bitwise operators you must understand how Java stores integers in binary using **two's complement**:

**Positive numbers:** standard binary
```
5  = 00000000 00000000 00000000 00000101  (32-bit int)
12 = 00000000 00000000 00000000 00001100
```

**Negative numbers:** two's complement = flip all bits + 1
```
-1  = 11111111 11111111 11111111 11111111
-5  = 11111111 11111111 11111111 11111011
-12 = 11111111 11111111 11111111 11110100
```

**Key two's complement facts:**
- The **most significant bit (MSB)** is the sign bit: `0` = positive, `1` = negative
- `-x = ~x + 1` (negate = complement then add 1)
- `Integer.MIN_VALUE = 0x80000000` = `10000000 00000000 00000000 00000000`
- `Integer.MAX_VALUE = 0x7FFFFFFF` = `01111111 11111111 11111111 11111111`
- All 1s (`0xFFFFFFFF`) = `-1`

---

### Bitwise AND `&`

Compares each pair of bits. Result bit is `1` only when **both** input bits are `1`:

```
Bit A | Bit B | A & B
  0   |   0   |   0
  0   |   1   |   0
  1   |   0   |   0
  1   |   1   |   1
```

```java
int a = 12;   // 0000 1100
int b =  5;   // 0000 0101
int r = a & b; // 0000 0100 = 4

System.out.println(12 & 5);   // 4
```

**Common uses of `&`:**
- **Check if a bit is set:** `(x & mask) != 0`
- **Check if even/odd:** `(x & 1) == 0` (even), `(x & 1) == 1` (odd)
- **Clear specific bits:** `x & ~mask`
- **Extract lower bits:** `x & 0xFF` (keep lowest 8 bits)

```java
int x = 42;
// Is bit 3 set? (bit counting from 0)
System.out.println((x & (1 << 3)) != 0);   // true — 42 = 101010, bit 3 = 1

// Is x even?
System.out.println((x & 1) == 0);           // true — 42 is even

// Keep only lowest 8 bits (mask to byte range)
System.out.println(x & 0xFF);               // 42 — already fits in byte
System.out.println(300 & 0xFF);             // 44 — 300 = 0x12C, 0x2C = 44
```

---

### Bitwise OR `|`

Result bit is `1` when **at least one** input bit is `1`:

```
Bit A | Bit B | A | B
  0   |   0   |   0
  0   |   1   |   1
  1   |   0   |   1
  1   |   1   |   1
```

```java
int a = 12;   // 0000 1100
int b =  5;   // 0000 0101
int r = a | b; // 0000 1101 = 13

System.out.println(12 | 5);   // 13
```

**Common uses of `|`:**
- **Set specific bits:** `x | mask`
- **Combine flags:** `FLAG_A | FLAG_B | FLAG_C`

```java
// Flag combination pattern
final int READ    = 0b001;   // 1
final int WRITE   = 0b010;   // 2
final int EXECUTE = 0b100;   // 4

int permissions = READ | WRITE;   // 0b011 = 3
System.out.println(permissions);  // 3

// Check if READ permission set:
System.out.println((permissions & READ) != 0);  // true
```

---

### Bitwise XOR `^`

Result bit is `1` when input bits are **different** (exclusive OR):

```
Bit A | Bit B | A ^ B
  0   |   0   |   0
  0   |   1   |   1
  1   |   0   |   1
  1   |   1   |   0
```

```java
int a = 12;   // 0000 1100
int b =  5;   // 0000 0101
int r = a ^ b; // 0000 1001 = 9

System.out.println(12 ^ 5);   // 9
```

**Common uses of `^`:**
- **Toggle bits:** `x ^ mask` (flips the mask bits in x)
- **Swap without temp variable:** `a ^= b; b ^= a; a ^= b;`
- **Detect difference:** if `a ^ b == 0` then `a == b`

```java
// XOR swap
int x = 5, y = 10;
x ^= y;    // x = 15 (5^10)
y ^= x;    // y = 5  (10^15)
x ^= y;    // x = 10 (15^5)
System.out.println(x + " " + y);  // 10 5

// Toggle a bit
int flags = 0b1010;
flags ^= 0b0010;     // toggle bit 1
System.out.println(Integer.toBinaryString(flags));  // 1000
```

---

### Bitwise Complement `~`

Flips every bit. As established in Topic 2.2: `~x = -(x+1)`:

```java
System.out.println(~0);    // -1
System.out.println(~5);    // -6
System.out.println(~-1);   //  0
```

---

### Left Shift `<<`

Shifts bits to the **left** by `n` positions. Vacated right bits filled with `0`. Bits shifted off the left end are discarded.

**Effect:** `x << n` = `x * 2^n` (for values that don't overflow)

```java
System.out.println(1 << 0);   // 1   (1 * 1)
System.out.println(1 << 1);   // 2   (1 * 2)
System.out.println(1 << 2);   // 4   (1 * 4)
System.out.println(1 << 3);   // 8   (1 * 8)
System.out.println(1 << 10);  // 1024
System.out.println(1 << 31);  // -2147483648 (Integer.MIN_VALUE — overflow!)

System.out.println(5 << 1);   // 10  (5 * 2)
System.out.println(5 << 2);   // 20  (5 * 4)
System.out.println(-1 << 1);  // -2  (all 1s shift left, new 0 on right)
```

**Visual:**
```
5 = 00000000 00000000 00000000 00000101
5 << 1:
    00000000 00000000 00000000 00001010  = 10
```

---

### Signed Right Shift `>>`

Shifts bits to the **right** by `n` positions. Vacated left bits filled with the **sign bit** (0 for positive, 1 for negative). This **preserves the sign**.

**Effect:** `x >> n` = `x / 2^n` (floor division, rounds toward negative infinity)

```java
System.out.println(10 >> 1);   // 5   (10 / 2)
System.out.println(10 >> 2);   // 2   (10 / 4, floor)
System.out.println(-10 >> 1);  // -5  (-10 / 2)
System.out.println(-1 >> 1);   // -1  (all 1s — sign bit extends)
System.out.println(-1 >> 31);  // -1  (still all 1s)
```

**Visual:**
```
-8 = 11111111 11111111 11111111 11111000
-8 >> 1:
     11111111 11111111 11111111 11111100  = -4 (sign bit extended)
```

> **Key distinction:** `>>` vs `/` for negative odd numbers:
> ```java
> System.out.println(-7 >> 1);   // -4  (floor toward -∞)
> System.out.println(-7 / 2);    // -3  (truncate toward 0)
> ```
> `>>` performs **floor division** (toward negative infinity). `/` truncates toward zero.

---

### Unsigned Right Shift `>>>`

Shifts bits to the **right** by `n` positions. Vacated left bits filled with **`0`** regardless of sign. This **does NOT preserve the sign** — always produces non-negative result for large shifts.

```java
System.out.println(10 >>> 1);   // 5   — same as >> for positive numbers
System.out.println(-1 >>> 1);   // 2147483647 (Integer.MAX_VALUE)
// -1 = 11111111 11111111 11111111 11111111
// >>> fills with 0: 01111111 11111111 11111111 11111111 = Integer.MAX_VALUE

System.out.println(-1 >>> 0);   // -1  (shift by 0 — no change)
System.out.println(-1 >>> 31);  // 1
System.out.println(-1 >>> 32);  // -1  (shift by 32 on int = shift by 0!)
```

**Visual:**
```
-8 = 11111111 11111111 11111111 11111000
-8 >>> 1:
     01111111 11111111 11111111 11111100  = 2147483644 (large positive)
```

---

### Shift Amount Reduction — Critical Rule

For `int` types, the shift amount is **masked to 5 bits** (mod 32):
For `long` types, the shift amount is **masked to 6 bits** (mod 64):

```java
// int shifts — mod 32
System.out.println(1 << 32);    // 1  — same as 1 << 0 (32 % 32 = 0)
System.out.println(1 << 33);    // 2  — same as 1 << 1 (33 % 32 = 1)
System.out.println(1 << 64);    // 1  — 64 % 32 = 0
System.out.println(-1 >>> 32);  // -1 — same as >>> 0

// long shifts — mod 64
long l = 1L;
System.out.println(l << 64);    // 1L — same as << 0 (64 % 64 = 0)
System.out.println(l << 65);    // 2L — same as << 1
```

> **Exam trap:** `1 << 32` is NOT `0` — it is `1` (same as `1 << 0`). Candidates expect the bits to be completely shifted out. But the shift amount is first reduced mod 32 for ints.

---

### Negative Shift Amounts

Negative shift amounts also get masked — they become large positive shifts:

```java
// -1 as 5-bit shift amount:
// -1 in binary (5 bits) = 11111 = 31
System.out.println(1 << -1);    // same as 1 << 31 = Integer.MIN_VALUE
System.out.println(1 >> -1);    // same as 1 >> 31 = 0
System.out.println(-1 >>> -1);  // same as -1 >>> 31 = 1
```

---

### Shift Operator Behavior Summary

| Expression | Result | Why |
|---|---|---|
| `1 << 0` | 1 | No shift |
| `1 << 1` | 2 | ×2 |
| `1 << 31` | -2147483648 | Overflow into sign bit |
| `1 << 32` | 1 | 32 % 32 = 0, no shift |
| `-1 >> 1` | -1 | Sign bit extended |
| `-1 >>> 1` | 2147483647 | 0 fills MSB |
| `-1 >>> 0` | -1 | No shift |
| `10 >> 1` | 5 | ÷2 |
| `-10 >> 1` | -5 | ÷2 preserving sign |
| `-7 >> 1` | -4 | Floor division |
| `-7 / 2` | -3 | Truncate toward zero |

---

### Type Promotion in Shift Operations

Just like arithmetic operators, shift operators promote operands:
- `byte`, `short`, `char` operands are promoted to `int` before shifting
- The **shift amount** (right operand) is always treated as `int`

```java
byte b = 1;
// byte result = b << 1;   // COMPILE ERROR — result is int
int result = b << 1;       // OK — 2

// Shift amount type doesn't change result type:
int x = 1;
long shift = 3L;
int r = x << shift;        // OK — shift amount converted to int (3)
// Actually: shift amount is always masked to int (lower 5 bits for int LHS)
```

---

### Compound Shift Assignment

```java
int x = 8;
x <<= 1;    // x = 16 (8 << 1)
x >>= 2;    // x = 4  (16 >> 2)
x >>>= 1;   // x = 2  (4 >>> 1)
System.out.println(x);  // 2

byte b = 8;
b <<= 1;    // OK — implicit cast: b = (byte)(8 << 1) = (byte)16 = 16
System.out.println(b);  // 16
```

---

### Practical Applications of Bitwise/Shift Operators

#### 1. Checking Even/Odd

```java
System.out.println((x & 1) == 0 ? "even" : "odd");
// Much faster than x % 2 == 0
```

#### 2. Powers of Two

```java
int powerOf2 = 1 << n;   // 2^n
// Check if x is a power of 2:
boolean isPowerOf2 = (x > 0) && (x & (x - 1)) == 0;
```

#### 3. Bit Masking and Flags

```java
int BOLD      = 1 << 0;   // 0001
int ITALIC    = 1 << 1;   // 0010
int UNDERLINE = 1 << 2;   // 0100

int style = BOLD | UNDERLINE;  // 0101 = 5

// Check
System.out.println((style & BOLD) != 0);      // true
System.out.println((style & ITALIC) != 0);    // false
System.out.println((style & UNDERLINE) != 0); // true

// Toggle
style ^= ITALIC;     // 0111 = 7 (add italic)
style ^= BOLD;       // 0110 = 6 (remove bold)

// Clear a bit
style &= ~UNDERLINE;  // clear underline
```

#### 4. Extract Bytes from Integer

```java
int value = 0x12345678;
byte b0 = (byte)(value & 0xFF);           // lowest byte: 0x78
byte b1 = (byte)((value >> 8) & 0xFF);   // second byte: 0x56
byte b2 = (byte)((value >> 16) & 0xFF);  // third byte:  0x34
byte b3 = (byte)((value >> 24) & 0xFF);  // highest byte: 0x12
```

---

## 2. Code Examples

### Example 1 — All Bitwise Operators

```java
public class BitwiseAll {
    public static void main(String[] args) {
        int a = 0b1010;   // 10
        int b = 0b1100;   // 12

        System.out.println(a & b);  // 0b1000 = 8  (AND)
        System.out.println(a | b);  // 0b1110 = 14 (OR)
        System.out.println(a ^ b);  // 0b0110 = 6  (XOR)
        System.out.println(~a);     // -11           (complement)

        // Verify ~a = -(a+1)
        System.out.println(~a == -(a + 1));  // true

        // With negatives
        System.out.println(-1 & 0xFF);   // 255 — mask to 8 bits
        System.out.println(-1 & 0x0F);   // 15  — mask to 4 bits
    }
}
```

---

### Example 2 — Left Shift Deep Dive

```java
public class LeftShiftDemo {
    public static void main(String[] args) {
        // Basic powers of 2
        for (int i = 0; i <= 10; i++) {
            System.out.println("1 << " + i + " = " + (1 << i));
        }
        // 1, 2, 4, 8, 16, 32, 64, 128, 256, 512, 1024

        // Overflow
        System.out.println(1 << 30);   // 1073741824 (2^30)
        System.out.println(1 << 31);   // -2147483648 (Integer.MIN_VALUE)

        // The 32-mod rule
        System.out.println(1 << 32);   // 1 — same as 1 << 0
        System.out.println(1 << 33);   // 2 — same as 1 << 1

        // Negative shift amounts (mod 32)
        System.out.println(1 << -1);   // Integer.MIN_VALUE — same as 1 << 31
    }
}
```

---

### Example 3 — Signed vs Unsigned Right Shift

```java
public class RightShiftDemo {
    public static void main(String[] args) {
        // Positive number — same for >> and >>>
        System.out.println(100 >> 2);    // 25
        System.out.println(100 >>> 2);   // 25

        // Negative number — DIFFERENT
        System.out.println(-100 >> 2);   // -25 — sign preserved
        System.out.println(-100 >>> 2);  // 1073741799 — 0 fills MSBs

        // The extreme: -1
        System.out.println(-1 >> 1);     // -1  — all 1s
        System.out.println(-1 >>> 1);    // Integer.MAX_VALUE = 2147483647

        // All 1s shifted right with >>> fills with 0
        System.out.println(-1 >>> 31);   // 1
        System.out.println(-1 >>> 32);   // -1 (32 mod 32 = 0 — no shift!)

        // >> vs / for negatives:
        System.out.println(-7 >> 1);     // -4 (floor division)
        System.out.println(-7 / 2);      // -3 (truncation toward zero)
    }
}
```

---

### Example 4 — Bit Flag Pattern

```java
public class BitFlags {
    static final int READ    = 1 << 0;  // 001
    static final int WRITE   = 1 << 1;  // 010
    static final int EXECUTE = 1 << 2;  // 100

    static void printPermissions(int perms) {
        System.out.println("Read:    " + ((perms & READ)    != 0));
        System.out.println("Write:   " + ((perms & WRITE)   != 0));
        System.out.println("Execute: " + ((perms & EXECUTE) != 0));
    }

    public static void main(String[] args) {
        int perms = READ | EXECUTE;    // 101 = 5
        printPermissions(perms);       // Read:true, Write:false, Execute:true

        perms |= WRITE;                // add WRITE: 111 = 7
        System.out.println(perms);     // 7

        perms &= ~EXECUTE;             // remove EXECUTE: 011 = 3
        System.out.println(perms);     // 3

        perms ^= READ;                 // toggle READ: 010 = 2
        System.out.println(perms);     // 2
    }
}
```

---

### Example 5 — Extract and Combine Bytes

```java
public class ByteExtraction {
    public static void main(String[] args) {
        int value = 0x1A2B3C4D;

        // Extract each byte
        int b0 = value & 0xFF;              // 0x4D = 77
        int b1 = (value >> 8) & 0xFF;       // 0x3C = 60
        int b2 = (value >> 16) & 0xFF;      // 0x2B = 43
        int b3 = (value >> 24) & 0xFF;      // 0x1A = 26

        System.out.printf("b0=0x%X b1=0x%X b2=0x%X b3=0x%X%n",
                          b0, b1, b2, b3);

        // Reconstruct the int
        int reconstructed = (b3 << 24) | (b2 << 16) | (b1 << 8) | b0;
        System.out.println(value == reconstructed);  // true
    }
}
```

---

### Example 6 — Common Bit Tricks

```java
public class BitTricks {
    public static void main(String[] args) {
        // Even/odd check
        for (int i = 0; i <= 5; i++) {
            System.out.println(i + " is " + ((i & 1) == 0 ? "even" : "odd"));
        }

        // Power of 2 check
        int[] nums = {1, 2, 3, 4, 8, 16, 24, 32};
        for (int n : nums) {
            boolean isPow2 = (n > 0) && (n & (n - 1)) == 0;
            System.out.println(n + " isPowerOf2: " + isPow2);
        }

        // Count set bits (Brian Kernighan's algorithm)
        int n = 0b10110110;   // 6 bits set
        int count = 0;
        while (n != 0) {
            n &= (n - 1);     // clear lowest set bit
            count++;
        }
        System.out.println("Set bits: " + count);  // 5 (not 6, let me recount)
        // 10110110 = bits 1,2,4,5,7 set = 5 bits

        // Multiply/divide by powers of 2
        System.out.println(5 << 3);   // 5 * 8 = 40
        System.out.println(40 >> 3);  // 40 / 8 = 5
    }
}
```

---

### Example 7 — Shift Mod Rule

```java
public class ShiftModRule {
    public static void main(String[] args) {
        // int shifts mod 32
        System.out.println(1 << 32);    // 1  (32%32=0)
        System.out.println(1 << 33);    // 2  (33%32=1)
        System.out.println(1 << 64);    // 1  (64%32=0)
        System.out.println(1 << 100);   // 16 (100%32=4)

        // long shifts mod 64
        System.out.println(1L << 64);   // 1L (64%64=0)
        System.out.println(1L << 65);   // 2L (65%64=1)

        // Practical implication:
        int ALL_ONES = -1;
        System.out.println(ALL_ONES >>> 32);  // -1 (no shift — mod 32 = 0)
        System.out.println(ALL_ONES >>> 31);  //  1
        System.out.println(ALL_ONES >>> 1);   //  Integer.MAX_VALUE
    }
}
```

---

### Example 8 — `>>` vs `/` for Negative Numbers

```java
public class ShiftVsDivision {
    public static void main(String[] args) {
        // For positive: >> and / are equivalent
        System.out.println(10 >> 1);   // 5
        System.out.println(10 / 2);    // 5

        // For negative with even result: same
        System.out.println(-10 >> 1);  // -5
        System.out.println(-10 / 2);   // -5

        // For negative with fraction: DIFFERENT
        System.out.println(-7 >> 1);   // -4 (floor toward -infinity)
        System.out.println(-7 / 2);    // -3 (truncate toward 0)

        System.out.println(-1 >> 1);   // -1 (floor of -0.5 = -1)
        System.out.println(-1 / 2);    //  0 (truncate of -0.5 = 0)

        System.out.println(-3 >> 1);   // -2 (floor of -1.5 = -2)
        System.out.println(-3 / 2);    // -1 (truncate of -1.5 = -1)
    }
}
```

---

## 3. Exam-Style Questions

---

**Q1 — MCQ**
What is the output?
```java
System.out.println(5 & 3);
System.out.println(5 | 3);
System.out.println(5 ^ 3);
```
- A. `1` / `7` / `6`
- B. `1` / `6` / `7`
- C. `7` / `1` / `6`
- D. `4` / `7` / `3`

**Answer: A**
`5=0101, 3=0011`. `5&3=0001=1`. `5|3=0111=7`. `5^3=0110=6`.

---

**Q2 — MCQ**
What is the output?
```java
System.out.println(1 << 32);
```
- A. `0`
- B. `1`
- C. `2`
- D. `4294967296`

**Answer: B**
For `int`, shift amount is masked mod 32. `32 % 32 = 0`. `1 << 0 = 1`.

---

**Q3 — MCQ**
What is the output?
```java
System.out.println(-1 >>> 1);
System.out.println(-1 >> 1);
```
- A. `0` / `-1`
- B. `Integer.MAX_VALUE` / `-1`
- C. `-1` / `0`
- D. `Integer.MAX_VALUE` / `Integer.MAX_VALUE`

**Answer: B**
`-1 = all 1s`. `>>> 1` fills MSB with 0 → `0111...1` = `Integer.MAX_VALUE = 2147483647`. `>> 1` fills MSB with sign (1) → still all 1s = `-1`.

---

**Q4 — MCQ**
What is the output?
```java
System.out.println(-7 >> 1);
System.out.println(-7 / 2);
```
- A. `-4` / `-4`
- B. `-3` / `-3`
- C. `-4` / `-3`
- D. `-3` / `-4`

**Answer: C**
`>> 1` = floor division toward -∞: `-7/2 = -3.5` → floor = `-4`. Integer `/` truncates toward 0: `-7/2 = -3.5` → truncate = `-3`.

---

**Q5 — Select All That Apply**
Which statements are true about the `>>>` operator? (Choose all that apply)

- A. It always fills vacated bits with `0`
- B. It preserves the sign of the operand
- C. For positive numbers, `>>>` produces the same result as `>>`
- D. `x >>> 0` always equals `x`
- E. `-1 >>> 32` equals `-1` for `int`
- F. `>>>` can be applied to `double` values

**Answer: A, C, D, E**
B is false — `>>>` does NOT preserve sign. F is false — `>>>` only for integral types. A: always fills with 0 ✅. C: positive numbers have 0 MSB anyway ✅. D: shift by 0 is no-op ✅. E: `-1 >>> 32` → 32%32=0 → no shift → `-1` ✅.

---

**Q6 — MCQ**
What is the output?
```java
int x = 0xFF;
byte b = (byte)(x & 0x7F);
System.out.println(b);
```
- A. `255`
- B. `127`
- C. `-1`
- D. `-128`

**Answer: B**
`0xFF = 255`. `255 & 0x7F = 255 & 127`. `11111111 & 01111111 = 01111111 = 127`. Cast to byte: 127 fits → `127`.

---

**Q7 — MCQ**
```java
int n = 8;
System.out.println(n & (n - 1));
```
- A. `8`
- B. `7`
- C. `0`
- D. `1`

**Answer: C**
`n=8=1000`. `n-1=7=0111`. `1000 & 0111 = 0000 = 0`. This is the "power of 2" check — any power of 2 ANDed with itself minus 1 gives 0.

---

**Q8 — MCQ**
What is the output?
```java
int x = 1;
x <<= 3;
x >>= 1;
x >>>= 1;
System.out.println(x);
```
- A. `4`
- B. `2`
- C. `1`
- D. `8`

**Answer: B**
`x=1`. `<<=3`: `x=8`. `>>=1`: `x=4`. `>>>=1`: `x=2`.

---

**Q9 — MCQ**
```java
System.out.println(1 << 31);
System.out.println(-1 << 1);
```
- A. `2147483648` / `-2`
- B. `-2147483648` / `-2`
- C. `-2147483648` / `2`
- D. `2147483648` / `2`

**Answer: B**
`1 << 31` shifts the 1 bit into the sign position → `Integer.MIN_VALUE = -2147483648`. `-1 = all 1s`. `-1 << 1 = 11...10 = -2`.

---

**Q10 — Select All That Apply**
Which produce `0` as output? (Choose all that apply)
```java
A. 5 & 0
B. 5 | 0
C. 5 ^ 5
D. 0 >> 5
E. 0 << 31
F. -1 >>> 32
```

**Answer: A, C, D, E**
A: `5 & 0 = 0` ✅. B: `5 | 0 = 5` ❌. C: `5 ^ 5 = 0` (same values XOR = 0) ✅. D: `0 >> 5 = 0` ✅. E: `0 << 31 = 0` ✅. F: `-1 >>> 32` = `-1 >>> 0` = `-1` ❌ (mod 32 rule).

---

**Q11 — MCQ**
What is the output?
```java
int a = 12, b = 10;
a ^= b;
b ^= a;
a ^= b;
System.out.println(a + " " + b);
```
- A. `12 10`
- B. `10 12`
- C. `0 0`
- D. `22 22`

**Answer: B**
XOR swap algorithm. After: `a=10, b=12`.

---

## 4. Trick Analysis

### Trap 1: `1 << 32` is `1`, not `0`
For `int`, shift amounts are reduced modulo 32. `32 % 32 = 0`, so `1 << 32 = 1 << 0 = 1`. Candidates expect the bit to disappear off the left end. This is the single most tested shift trap on OCP.

---

### Trap 2: `>>` vs `>>>` only matters for negative numbers
For positive numbers, both give the same result — the MSB is already 0, so whether you fill with 0 or the sign bit doesn't change anything. The exam uses positive numbers to check if candidates know when the operators actually differ.

---

### Trap 3: `-7 >> 1` is `-4`, not `-3`
`>>` performs **floor division** (toward negative infinity). `/` performs **truncation** (toward zero). `-7 >> 1 = -4` because floor(-3.5) = -4. `-7 / 2 = -3` because truncate(-3.5) = -3. These give different results for negative odd numbers.

---

### Trap 4: `-1 >>> 31` is `1`, `-1 >>> 32` is `-1`
`-1 = all 1s`. `>>> 31` → leaves one 1-bit = 1. `>>> 32` → mod 32 = 0 → no shift → still -1. Candidates expect `-1 >>> 32 = 0` (all bits shifted out). The mod-32 rule prevents this.

---

### Trap 5: `~x` result is `-(x+1)`, NOT `-x`
`~5 = -6`, not `-5`. The complement flips bits, which in two's complement is `-(x+1)`. True negation (`-x`) requires `~x + 1`. Many candidates confuse bitwise complement with arithmetic negation.

---

### Trap 6: `5 ^ 5 = 0` — XOR with itself always 0
Any value XORed with itself is 0. This is used in "find the unique element" problems (XOR all elements — duplicates cancel). The exam uses this property in code snippets where `a ^= a` appears.

---

### Trap 7: Shift operators don't work on `float` or `double`
`5.0 << 1` is a COMPILE ERROR. Shift operators are only for integral types. The exam presents floating-point shift to test this.

---

### Trap 8: `byte b = 1; b <<= 1;` compiles but `b = b << 1` does not
Like all compound operators, `<<=` includes an implicit narrowing cast. `b << 1` produces `int`; assigning to `byte` needs explicit cast. `b <<= 1` handles it implicitly.

---

### Trap 9: `n & (n-1)` clears the lowest set bit
`n & (n-1)` is 0 only when `n` is a power of 2 (or 0). This is a classic bit trick — if the exam shows this expression and asks what the output is for a power of 2, the answer is `0`.

---

### Trap 10: Shift amount sign and promotion
The shift amount (right operand) is always `int`. If you use a `long` as the shift amount for an `int` left operand, only the lower 5 bits matter (and the long is still allowed — it just gets masked). If you use a negative value, it gets masked too (e.g., `-1` masked to 5 bits = `11111` = 31).

---

## 5. Summary Sheet

### Bitwise Operators Truth Table (per bit)

| A | B | `A & B` | `A \| B` | `A ^ B` | `~A` |
|---|---|---|---|---|---|
| 0 | 0 | 0 | 0 | 0 | 1 |
| 0 | 1 | 0 | 1 | 1 | 1 |
| 1 | 0 | 0 | 1 | 1 | 0 |
| 1 | 1 | 1 | 1 | 0 | 0 |

---

### Shift Operators Comparison

| Operator | Name | Fills Vacated Bits With | Preserves Sign? | Works On |
|---|---|---|---|---|
| `<<` | Left shift | `0` on right | No (can overflow) | Integral only |
| `>>` | Signed right shift | Sign bit (0 or 1) | ✅ Yes | Integral only |
| `>>>` | Unsigned right shift | Always `0` | ❌ No | Integral only |

---

### Shift Amount Reduction

| Operand Type | Shift Amount Masked To | Effective Modulo |
|---|---|---|
| `byte`, `short`, `char`, `int` | Lower 5 bits | mod 32 |
| `long` | Lower 6 bits | mod 64 |

---

### Shift Effect on Value

| Operation | Mathematical Effect |
|---|---|
| `x << n` | `x × 2^n` (may overflow) |
| `x >> n` | `x ÷ 2^n` (floor, preserves sign) |
| `x >>> n` | `x ÷ 2^n` (treating x as unsigned) |

---

### Common Bit Patterns

| Expression | Result | Use Case |
|---|---|---|
| `x & 1` | `0` (even) or `1` (odd) | Parity check |
| `x & (x-1)` | `0` if x is power of 2 | Power-of-2 test |
| `x & 0xFF` | Lower 8 bits of x | Mask to byte |
| `x \| mask` | Set bits in mask | Set bits |
| `x & ~mask` | Clear bits in mask | Clear bits |
| `x ^ mask` | Flip bits in mask | Toggle bits |
| `x ^ x` | `0` | Clear/cancel |
| `~x` | `-(x+1)` | Complement |
| `1 << n` | `2^n` | Power of 2 |

---

### Key Rules to Memorize

| Rule | Detail |
|---|---|
| `~x` formula | Always `-(x+1)` |
| `1 << 32` (int) | `1` — not 0 (mod 32 rule) |
| `>>` vs `/` negative | `>>` floors, `/` truncates toward 0 |
| `-1 >> n` | Always `-1` (sign bit extends) |
| `-1 >>> 1` | `Integer.MAX_VALUE` |
| `-1 >>> 32` | `-1` (32%32=0, no shift) |
| Shift on float/double | COMPILE ERROR |
| Compound `<<=` on byte | OK — implicit cast |
| `b = b << 1` on byte | COMPILE ERROR — no implicit cast |
| `x ^ x` | Always `0` |
| `x & (x-1)` | Clears lowest set bit |

---

### `>>` vs `>>>` for Negative Numbers

| Expression | `>>` Result | `>>>` Result |
|---|---|---|
| `-1 >> 1` | `-1` | `2147483647` |
| `-8 >> 1` | `-4` | `2147483644` |
| `-1 >> 31` | `-1` | `1` |
| `-1 >>> 32` | `-1` | `-1` (mod 32 = no shift) |

---
