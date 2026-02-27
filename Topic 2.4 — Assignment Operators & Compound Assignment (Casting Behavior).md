# Topic 2.4 — Assignment Operators & Compound Assignment
## (Casting Behavior)

---

## 1. Conceptual Explanation

### What Are Assignment Operators?

Assignment operators store a value into a variable. Java has two categories:

1. **Simple assignment** — `=`
2. **Compound assignment** — `+=`, `-=`, `*=`, `/=`, `%=`, `&=`, `|=`, `^=`, `<<=`, `>>=`, `>>>=`

The distinction between these two categories is more significant than it appears — especially around **implicit casting behavior** — and this is tested heavily on the OCP exam.

---

### Simple Assignment `=`

The basic assignment operator evaluates the right-hand side and stores the result in the left-hand variable.

```java
int x = 10;          // literal assigned to int
double d = 3.14;     // literal assigned to double
String s = "hello";  // reference assigned
int y = x;           // value of x copied to y
```

#### Assignment is an Expression

In Java, assignment is an **expression** that returns the assigned value. This allows chaining:

```java
int a, b, c;
a = b = c = 10;    // right-to-left: c=10, b=10, a=10
System.out.println(a + " " + b + " " + c);  // 10 10 10

// Assignment in conditions (legal but discouraged):
int x;
if ((x = getValue()) > 0) {   // assigns AND tests in one step
    System.out.println(x);
}
```

> **Exam trap:** `a = b = c = 10` — assignment is right-to-left associative. `c` gets `10` first, then `b` gets the result of `c = 10` (which is `10`), then `a` gets the result of `b = 10` (which is `10`).

---

### Widening in Simple Assignment

Java allows **implicit widening** in assignments — assigning a smaller type to a larger type:

```java
byte b = 10;
short s = b;      // byte → short (widening)
int i = s;        // short → int
long l = i;       // int → long
float f = l;      // long → float (may lose precision!)
double d = f;     // float → double
```

Widening conversions are automatic — no cast required. But some widening conversions **can lose precision**:

```java
long bigLong = 1_234_567_891_234L;
float f = bigLong;   // compiles — widening, but may lose precision
System.out.println(bigLong);  // 1234567891234
System.out.println((long)f);  // 1234567880704 — different! precision lost
```

> **Exam note:** `int → float` and `long → double` can silently lose precision because floating-point types have fewer mantissa bits than their integer counterparts. The compiler allows it with no warning.

---

### Narrowing in Simple Assignment — Requires Explicit Cast

Assigning a larger type to a smaller type requires an **explicit cast**:

```java
double d = 3.99;
int i = (int) d;    // explicit cast required — truncates to 3

long l = 100L;
int i2 = (int) l;   // explicit cast required

int big = 200;
byte b = (byte) big;  // explicit cast required — may lose data
```

Without the cast, it's a compile error:

```java
double d = 3.99;
int i = d;          // COMPILE ERROR — possible lossy conversion
```

**Exception — compile-time constants:**
The compiler makes an exception for small **compile-time constant** integer values that fit in the target type:

```java
byte b = 42;     // OK — 42 is a compile-time constant that fits in byte
byte b2 = 200;   // COMPILE ERROR — 200 doesn't fit in byte (-128 to 127)
final int x = 10;
byte b3 = x;     // OK — x is a final (constant) with value 10, fits in byte
int y = 10;
byte b4 = y;     // COMPILE ERROR — y is not final, not a compile-time constant
```

---

### Compile-Time Constant Assignment — Deep Rules

This is one of the most nuanced areas of the assignment topic:

```java
// Case 1: literal fits → OK
byte b1 = 100;       // OK — 100 fits in byte

// Case 2: literal doesn't fit → ERROR
byte b2 = 200;       // COMPILE ERROR

// Case 3: final variable used as constant → OK
final int HUNDRED = 100;
byte b3 = HUNDRED;   // OK — HUNDRED is a compile-time constant = 100

// Case 4: non-final variable → ERROR
int hundred = 100;
byte b4 = hundred;   // COMPILE ERROR — even though value is 100, not constant

// Case 5: computed constant → depends
final int A = 50, B = 50;
byte b5 = A + B;     // OK — A+B = 100, compile-time constant expression

final int C = 100;
final int D = 100;
byte b6 = C + D;     // COMPILE ERROR — C+D = 200, doesn't fit in byte

// Case 6: non-final in expression → ERROR
int e = 50;
final int F = 50;
byte b7 = e + F;     // COMPILE ERROR — e is not final → not compile-time constant
```

---

### Compound Assignment Operators

Compound operators combine an arithmetic/bitwise operation with assignment:

| Operator | Equivalent (approximately) |
|---|---|
| `x += y` | `x = (type_of_x)(x + y)` |
| `x -= y` | `x = (type_of_x)(x - y)` |
| `x *= y` | `x = (type_of_x)(x * y)` |
| `x /= y` | `x = (type_of_x)(x / y)` |
| `x %= y` | `x = (type_of_x)(x % y)` |
| `x &= y` | `x = (type_of_x)(x & y)` |
| `x \|= y` | `x = (type_of_x)(x \| y)` |
| `x ^= y` | `x = (type_of_x)(x ^ y)` |
| `x <<= y` | `x = (type_of_x)(x << y)` |
| `x >>= y` | `x = (type_of_x)(x >> y)` |
| `x >>>= y` | `x = (type_of_x)(x >>> y)` |

The key phrase is `(type_of_x)` — the result is **implicitly cast back to the type of the left-hand variable**. This is the defining feature of compound assignment.

---

### The Implicit Narrowing Cast — The Core Exam Topic

This is the most tested aspect of compound assignment:

```java
// Simple assignment — requires explicit cast
byte b = 10;
b = b + 5;     // COMPILE ERROR — b+5 is int, cannot assign to byte

// Compound assignment — implicit cast included
byte b = 10;
b += 5;        // OK — equivalent to b = (byte)(b + 5)
```

More examples:

```java
short s = 1000;
s *= 2;    // OK — s = (short)(1000 * 2) = (short)2000 = 2000 (fits)
s *= 20;   // OK — s = (short)(2000 * 20) = (short)40000 = 7232 (overflow, wraps)

char c = 'A';  // 65
c += 1;        // OK — c = (char)(65 + 1) = (char)66 = 'B'
c += 100000;   // OK — wraps around in char range

float f = 1.0f;
f += 0.5;      // OK — f = (float)(1.0f + 0.5) = 1.5f
// Note: 0.5 is double, but result is cast to float
```

---

### Compound Assignment is NOT Simply `x = x op y`

This subtle distinction is tested:

```java
byte b = 10;
b = b + 1;    // COMPILE ERROR — not the same as b += 1
b += 1;       // OK — has implicit cast

// The JLS says compound assignment x op= y is equivalent to:
// x = (T)(x op y)   where T is the type of x
// NOT: x = x op y
```

Also, compound assignment evaluates the left-hand side **only once**:

```java
int[] arr = {10, 20, 30};
int i = 0;
arr[i++] += 5;   // arr[0] += 5 → arr[0] = 15; i = 1
// The index i++ is evaluated ONCE, not twice
System.out.println(arr[0]);  // 15
System.out.println(i);       // 1
```

---

### Compound Assignment with `String +=`

The `+=` operator also works for String concatenation:

```java
String s = "Hello";
s += " World";    // s = "Hello World"
s += 42;          // s = "Hello World42"
s += true;        // s = "Hello World42true"
System.out.println(s);  // Hello World42true
```

---

### Return Value of Assignment

Every assignment expression **returns the assigned value**. This enables:

```java
int a, b;
a = b = 5;    // both get 5

// In while loop — common pattern:
String line;
while ((line = readLine()) != null) {  // assigns and tests
    process(line);
}

// In if statement:
int result;
if ((result = compute()) > 100) {
    System.out.println("Big: " + result);
}
```

---

### Casting Between Numeric Types — Detailed Behavior

#### `double`/`float` → integer: Truncation

```java
System.out.println((int) 3.9);    //  3 — truncates toward zero
System.out.println((int) -3.9);   // -3 — truncates toward zero
System.out.println((int) 3.0);    //  3
System.out.println((long) 3.9);   //  3L
```

#### Integer → Smaller Integer: Bit Truncation

When casting a larger integer to a smaller one, Java **keeps only the lower bits**:

```java
// int to byte — keeps lowest 8 bits
int i = 130;           // binary: 00000000 00000000 00000000 10000010
byte b = (byte) i;     // keeps: 10000010 = -126 (two's complement)
System.out.println(b); // -126

int i2 = 256;          // binary: 00000000 00000000 00000001 00000000
byte b2 = (byte) i2;   // keeps lowest 8 bits: 00000000 = 0
System.out.println(b2); // 0

int i3 = 257;
byte b3 = (byte) i3;   // lowest 8 bits: 00000001 = 1
System.out.println(b3); // 1
```

**Formula for byte overflow:** `(byte) x = x - 256 * floor((x + 128) / 256)` — but for the exam, just remember the modulo pattern: the byte value is `x % 256` adjusted for the signed range.

Simpler approach:
- Take `x % 256` (get value in 0-255 range if originally positive)
- If result > 127, subtract 256 to get the signed byte

```java
// 300 % 256 = 44 — fits in byte as-is
System.out.println((byte) 300);   // 44

// 200 % 256 = 200 → 200 > 127 → 200 - 256 = -56
System.out.println((byte) 200);   // -56

// 256 % 256 = 0
System.out.println((byte) 256);   // 0

// 255 % 256 = 255 → 255 > 127 → 255 - 256 = -1
System.out.println((byte) 255);   // -1
```

---

#### `char` Casting

`char` is a 16-bit **unsigned** type (0–65535). Casting interactions with `int`:

```java
char c = (char) 65;          // 'A'
char c2 = (char) 66;         // 'B'
int i = 'A';                 // 65 — widening
char c3 = (char) ('A' + 1);  // 'B' — cast result of arithmetic

// Negative int to char — wraps to unsigned range
char c4 = (char) -1;         // 65535 — maximum char value
System.out.println((int) c4); // 65535

char c5 = (char) 65536;      // wraps: 65536 % 65536 = 0
System.out.println(c5);      // null character '\u0000'
```

---

### The Complete Assignment Type Compatibility Matrix

```
Widening (automatic, no cast):
byte → short → int → long → float → double
char → int → long → float → double

Narrowing (explicit cast required):
double → float → long → int → short/char → byte
```

**Special cases:**
- `byte` ↔ `char`: cast always required in both directions
- `short` ↔ `char`: cast always required in both directions
- `byte` → `char`: requires cast (byte could be negative, char is unsigned)
- `char` → `byte`: requires cast
- `char` → `short`: requires cast (char is unsigned 0-65535, short is signed -32768-32767)

---

### Assignment Operators — Full Behavior Summary

```java
int x = 10;

// All compound operators with x = 10:
x += 3;   // 13
x -= 3;   // 10 (back)
x *= 3;   // 30
x /= 3;   // 10 (back)
x %= 3;   // 1
x = 10;   // reset
x &= 6;   // 10 & 6 = 2 (bitwise AND: 1010 & 0110 = 0010)
x = 10;
x |= 5;   // 10 | 5 = 15 (bitwise OR: 1010 | 0101 = 1111)
x = 10;
x ^= 3;   // 10 ^ 3 = 9 (bitwise XOR: 1010 ^ 0011 = 1001)
x = 10;
x <<= 1;  // 10 << 1 = 20 (shift left, multiply by 2)
x = 10;
x >>= 1;  // 10 >> 1 = 5 (signed right shift, divide by 2)
x = -10;
x >>>= 1; // unsigned right shift (fills with 0 for sign bit)
```

---

## 2. Code Examples

### Example 1 — Simple vs Compound Assignment with byte

```java
public class ByteAssignment {
    public static void main(String[] args) {
        byte b = 100;

        // Simple assignment — compile error without cast
        // b = b + 10;      // COMPILE ERROR

        // Compound assignment — implicit cast
        b += 10;            // OK — (byte)(100+10) = 110
        System.out.println(b);  // 110

        b += 50;            // (byte)(110+50) = (byte)160 = -96 (overflow)
        System.out.println(b);  // -96

        // Explicit cast equivalent
        b = (byte)(b + 10); // OK — same as b += 10
        System.out.println(b);  // -86
    }
}
```

---

### Example 2 — Compile-Time Constants

```java
public class ConstantAssignment {
    public static void main(String[] args) {
        // Literals — compile-time constants
        byte b1 = 50;          // OK — 50 fits in byte
        // byte b2 = 200;      // ERROR — 200 doesn't fit

        // final variables — compile-time constants
        final int FIFTY = 50;
        byte b3 = FIFTY;       // OK — final, value known, fits

        // Non-final variables — NOT compile-time constants
        int fifty = 50;
        // byte b4 = fifty;    // ERROR — not a compile-time constant

        // Constant expressions
        final int A = 30, B = 20;
        byte b5 = A + B;       // OK — 30+20=50, compile-time constant

        final int C = 100, D = 100;
        // byte b6 = C + D;    // ERROR — 200 doesn't fit in byte

        System.out.println(b1 + " " + b3 + " " + b5);  // 50 50 50
    }
}
```

---

### Example 3 — Narrowing Cast Behavior

```java
public class NarrowingCast {
    public static void main(String[] args) {
        // double to int — truncation
        System.out.println((int) 9.9);    // 9
        System.out.println((int) -9.9);   // -9 — toward zero
        System.out.println((int) 9.0);    // 9

        // int to byte — bit truncation
        System.out.println((byte) 127);   // 127 — max byte
        System.out.println((byte) 128);   // -128 — overflow
        System.out.println((byte) 255);   // -1
        System.out.println((byte) 256);   // 0
        System.out.println((byte) 300);   // 44 — 300-256=44
        System.out.println((byte) -1);    // -1 — fits in byte range

        // int to char
        System.out.println((char) 65);    // A
        System.out.println((char) -1);    // '\uFFFF' = 65535
        System.out.println((int)(char)(-1)); // 65535
    }
}
```

---

### Example 4 — Chained Assignment

```java
public class ChainedAssignment {
    public static void main(String[] args) {
        int a, b, c;
        a = b = c = 42;
        System.out.println(a + " " + b + " " + c);  // 42 42 42

        // Return value of assignment
        int x = 0;
        int y = (x = 5) * 2;   // x is assigned 5, y = 5 * 2 = 10
        System.out.println(x + " " + y);  // 5 10

        // Chained with different types — widening
        int i;
        double d;
        d = i = 7;   // right-to-left: i=7, then d=7 (widened to 7.0)
        System.out.println(i + " " + d);  // 7 7.0
    }
}
```

---

### Example 5 — Compound Assignment with Overflow

```java
public class CompoundOverflow {
    public static void main(String[] args) {
        byte b = 120;
        b += 10;   // (byte)(120+10) = (byte)130 = -126
        System.out.println(b);  // -126

        short s = 30000;
        s += 5000;  // (short)(30000+5000) = (short)35000
        // 35000 > Short.MAX_VALUE (32767), so wraps
        System.out.println(s);  // -30536 (35000 - 65536)

        // float with double operand — cast to float
        float f = 1.0f;
        f += 0.1;    // 0.1 is double, result cast to float
        System.out.println(f);  // 1.1 (approximately)

        // String concatenation
        String str = "Hello";
        str += 42;
        str += true;
        System.out.println(str);  // Hello42true
    }
}
```

---

### Example 6 — Array Index with Compound Assignment

```java
public class ArrayCompound {
    public static void main(String[] args) {
        int[] arr = {1, 2, 3, 4, 5};
        int i = 0;

        // i++ evaluated ONCE for the index
        arr[i++] += 10;
        // arr[0] becomes 11, i becomes 1
        System.out.println(arr[0] + " " + i);  // 11 1

        // More complex
        arr[i] *= arr[i + 1];
        // arr[1] = arr[1] * arr[2] = 2 * 3 = 6
        System.out.println(arr[1]);  // 6
    }
}
```

---

### Example 7 — byte/char/short Narrowing Rules

```java
public class NarrowingRules {
    public static void main(String[] args) {
        // byte → char: requires cast (byte can be negative)
        byte b = 65;
        char c = (char) b;    // OK with cast
        System.out.println(c);  // A

        // char → byte: requires cast
        char ch = 'A';   // 65
        byte by = (byte) ch;   // OK with cast
        System.out.println(by);  // 65

        // char → short: requires cast (char unsigned 0-65535, short -32768-32767)
        char big = (char) 40000;   // 40000 is valid char
        short s = (short) big;     // 40000 > Short.MAX_VALUE, wraps
        System.out.println(s);     // -25536 (40000 - 65536)

        // int literal to char — OK if in range
        char x = 100;       // OK — 100 is compile-time constant in char range
        char y = 65535;     // OK — max char
        // char z = 65536;  // ERROR — exceeds char range
        // char w = -1;     // ERROR — negative literal to char
    }
}
```

---

### Example 8 — The `=` Expression Returns Value

```java
public class AssignmentExpression {
    static int counter = 0;

    static int next() {
        return ++counter;
    }

    public static void main(String[] args) {
        // Assignment in condition
        int value;
        while ((value = next()) < 5) {
            System.out.print(value + " ");  // 1 2 3 4
        }
        System.out.println();
        System.out.println("Last value: " + value);  // 5

        // Nested in expression
        int a = 0;
        int b = (a = 10) + 5;  // a assigned 10, b = 10+5 = 15
        System.out.println(a + " " + b);  // 10 15
    }
}
```

---

## 3. Exam-Style Questions

---

**Q1 — MCQ**
What is the output?
```java
byte b = 100;
b += 100;
System.out.println(b);
```
- A. 200
- B. -56
- C. Compile error
- D. Runtime exception

**Answer: B**
`b += 100` → `(byte)(100 + 100)` = `(byte)200`. 200 > 127, wraps: `200 - 256 = -56`.

---

**Q2 — MCQ**
Which line causes a compile error?
```java
byte b = 50;           // Line 1
b = b + 50;            // Line 2
b += 50;               // Line 3
final int x = 50;
b = x;                 // Line 4
```
- A. Line 1
- B. Line 2
- C. Line 3
- D. Line 4

**Answer: B**
Line 2: `b + 50` produces `int`, cannot assign to `byte` without cast — compile error.
Line 3: compound assignment has implicit cast — OK.
Line 4: `x` is a `final` compile-time constant with value 50, which fits in byte — OK.

---

**Q3 — Select All That Apply**
Which assignments compile without error? (Choose all that apply)
```java
A. byte b = 127;
B. byte b = 128;
C. final int X = 50; byte b = X;
D. int x = 50; byte b = x;
E. byte b = (byte) 200;
F. byte b = 10; b *= 20;
```

**Answer: A, C, E, F**
B: 128 out of byte range — ERROR.
D: `x` is not final, not a compile-time constant — ERROR.
A: 127 fits — OK. C: final constant fits — OK. E: explicit cast — OK. F: compound `*=` has implicit cast — OK.

---

**Q4 — MCQ**
What is the output?
```java
int a = 5, b = 10;
int c = (a = b) + a;
System.out.println(a + " " + b + " " + c);
```
- A. `5 10 15`
- B. `10 10 20`
- C. `10 10 15`
- D. `5 10 20`

**Answer: B**
`(a = b)` assigns `b` (10) to `a`, and returns 10. Then `+ a` uses the updated `a` = 10. `c = 10 + 10 = 20`. `a = 10`, `b = 10`, `c = 20`.

---

**Q5 — MCQ**
What is the output?
```java
System.out.println((int) -3.9);
System.out.println((int) 3.9);
System.out.println((byte) 200);
System.out.println((byte) 256);
```
- A. `-4` / `3` / `-56` / `0`
- B. `-3` / `3` / `-56` / `0`
- C. `-4` / `4` / `-56` / `1`
- D. `-3` / `4` / `56` / `0`

**Answer: B**
`(int)-3.9 = -3` (truncate toward zero). `(int)3.9 = 3`. `(byte)200 = 200-256 = -56`. `(byte)256 = 0`.

---

**Q6 — MCQ**
```java
int x = 10;
int y = x = 5;
System.out.println(x + " " + y);
```
- A. `10 5`
- B. `5 10`
- C. `5 5`
- D. Compile error

**Answer: C**
Right-to-left: `x = 5` executes first (x=5, returns 5). Then `y = 5` (the return value). Both become 5.

---

**Q7 — MCQ**
```java
byte b = 10;
b += 200;
System.out.println(b);
```
- A. 210
- B. -46
- C. Compile error
- D. -86

**Answer: B**
`b += 200` → `(byte)(10 + 200)` = `(byte)210`. 210 - 256 = -46.

---

**Q8 — Select All That Apply**
Which statements are true about compound assignment operators? (Choose all that apply)

- A. `x += y` is always exactly equivalent to `x = x + y`
- B. Compound assignment includes an implicit narrowing cast to the type of the left operand
- C. `byte b = 10; b += 5;` compiles without error
- D. `byte b = 10; b = b + 5;` compiles without error
- E. Compound assignment evaluates the left-hand side index expression only once
- F. `String s = "hi"; s += 42;` is a compile error

**Answer: B, C, E**
A: False — `x = x + y` lacks the implicit cast. D: False — `b + 5` is int, compile error. F: False — String `+=` is valid. B, C, E are all correct.

---

**Q9 — MCQ**
```java
final int A = 100;
final int B = 100;
byte b = A + B;
System.out.println(b);
```
- A. `-56`
- B. `200`
- C. Compile error — A+B = 200 exceeds byte range
- D. Runtime exception

**Answer: C**
`A + B = 200` which is a compile-time constant, but `200` does not fit in `byte` (-128 to 127). Compile error.

---

**Q10 — MCQ**
What is the output?
```java
char c = 'A';
c += 32;
System.out.println(c);
```
- A. `97`
- B. `a`
- C. Compile error
- D. `B`

**Answer: B**
`'A'` is 65. `c += 32` → `(char)(65 + 32)` = `(char)97` = `'a'`. Prints `a`.

---

**Q11 — MCQ**
```java
int[] arr = {10, 20, 30};
int i = 1;
arr[i++] += arr[i];
System.out.println(arr[1] + " " + i);
```
- A. `50 2`
- B. `40 2`
- C. `50 3`
- D. Compile error

**Answer: A**
`arr[i++]` evaluates index as 1 (i becomes 2). `arr[i]` is `arr[2]` = 30 (i is now 2). `arr[1] += 30` → `arr[1] = 20 + 30 = 50`. i = 2. Output: `50 2`.

---

## 4. Trick Analysis

### Trap 1: `b = b + 1` vs `b += 1` for byte/short/char
The most tested compound assignment trap. `b = b + 1` is a compile error for `byte b` because `b + 1` produces `int`. But `b += 1` is legal because compound assignment includes an implicit narrowing cast. This distinction appears in multiple OCP questions.

---

### Trap 2: `final int x = 50; byte b = x;` compiles
A `final` local variable with a known value IS a compile-time constant. The compiler treats it like a literal and checks if the value fits in the target type. So `final int x = 50; byte b = x;` is OK because `50` fits in `byte`. But `int x = 50; byte b = x;` is NOT OK because `x` is not `final`.

---

### Trap 3: `final int A = 100, B = 100; byte b = A + B;` is a compile error
Even though `A` and `B` are constants, their sum `200` does not fit in `byte`. The compiler evaluates the constant expression and checks the result. `200` > `127` → compile error. Candidates assume `final` variables always enable the assignment.

---

### Trap 4: `int y = x = 5` — both become 5
Right-to-left: `x = 5` executes first, returns 5. Then `y = 5`. Candidates expect `y` to get `x`'s original value. It gets the ASSIGNED value (5), which is what the expression `x = 5` returns.

---

### Trap 5: `(a = b) + a` — left side affects right side
In `int c = (a = b) + a`, the assignment `a = b` modifies `a` before the second `a` is read. So both uses of `a` in the expression see the new value. The exam uses this to test whether candidates understand that the assignment side effect happens during expression evaluation.

---

### Trap 6: `(byte) 200` is `-56`, not `200`
Narrowing cast keeps only the lower 8 bits. For `200`: binary `11001000` in 8 bits is `-56` in two's complement. Quick calculation: `200 - 256 = -56`. The exam presents various values and asks for the cast result.

---

### Trap 7: `(int) -3.9` is `-3`, not `-4`
Casting double to int truncates TOWARD ZERO. For negative values, this means the result is less negative than expected. `-3.9` truncated toward zero is `-3`. Candidates who think of "rounding down" expect `-4`.

---

### Trap 8: Compound assignment with array index — index evaluated once
`arr[i++] += 5` — the index `i++` is evaluated EXACTLY ONCE. The compound assignment does not re-evaluate the index for the read and write operations. The exam tests this by using side-effecting index expressions.

---

### Trap 9: `char c = -1` is a compile error but `(char) -1` is valid
Assigning a negative integer literal directly to `char` is a compile error because `-1` is out of `char`'s range (0-65535). But `(char) -1` with an explicit cast compiles and gives `65535` (all 16 bits set). The exam distinguishes between the literal assignment rule and the explicit cast behavior.

---

### Trap 10: `long → float` widening can lose precision
`float f = someVeryLongLong` compiles (widening) but may produce an imprecise result because `float` has only 23 mantissa bits while `long` has 63 value bits. This is the one widening conversion that silently loses data. The compiler allows it without any cast requirement.

---

## 5. Summary Sheet

### Assignment Operator Types

| Type | Operators | Implicit Cast? |
|---|---|---|
| Simple | `=` | No — requires explicit cast for narrowing |
| Compound arithmetic | `+=` `-=` `*=` `/=` `%=` | ✅ Yes — cast to left-hand type |
| Compound bitwise | `&=` `\|=` `^=` | ✅ Yes |
| Compound shift | `<<=` `>>=` `>>>=` | ✅ Yes |

---

### Narrowing Assignment Rules

| Scenario | Compiles? | Why |
|---|---|---|
| `byte b = 50` | ✅ | Literal 50 fits in byte |
| `byte b = 200` | ❌ | Literal 200 out of byte range |
| `final int X=50; byte b=X` | ✅ | Compile-time constant, value fits |
| `int x=50; byte b=x` | ❌ | Not a compile-time constant |
| `final int A=100,B=100; byte b=A+B` | ❌ | A+B=200 doesn't fit |
| `byte b = (byte)200` | ✅ | Explicit cast |
| `byte b = 10; b += 200` | ✅ | Implicit cast in compound |
| `byte b = 10; b = b + 200` | ❌ | No implicit cast in simple |

---

### Casting Results — Key Values

| Expression | Result | Why |
|---|---|---|
| `(int) 3.9` | `3` | Truncates toward zero |
| `(int) -3.9` | `-3` | Truncates toward zero |
| `(byte) 127` | `127` | Max byte value |
| `(byte) 128` | `-128` | 128 - 256 = -128 |
| `(byte) 200` | `-56` | 200 - 256 = -56 |
| `(byte) 255` | `-1` | 255 - 256 = -1 |
| `(byte) 256` | `0` | 256 % 256 = 0 |
| `(byte) 300` | `44` | 300 - 256 = 44 |
| `(char) -1` | `'\uFFFF'` (65535) | Low 16 bits of -1 |
| `(char) 65` | `'A'` | Unicode 65 |

---

### Widening Conversion Chain

```
byte → short → int → long → float → double
char →    int →  ↑
```

**Can lose precision:** `int → float`, `long → float`, `long → double`
All others are lossless.

---

### Key Rules to Memorize

| Rule | Detail |
|---|---|
| `b = b + 1` on byte | COMPILE ERROR — explicit cast needed |
| `b += 1` on byte | OK — implicit cast to byte |
| `final int x = n; byte b = x` | OK if n fits in byte |
| `int x = n; byte b = x` | COMPILE ERROR — not final |
| Assignment returns value | `int y = (x = 5)` gives y=5 |
| Chained assignment | Right-to-left: `a = b = 5` |
| `(int)` casting | Truncates TOWARD ZERO |
| `(byte)` casting | Keeps lower 8 bits — wraps for >127 |
| String `+=` | Legal — concatenates |
| Array index in compound | Index evaluated ONCE only |

---
