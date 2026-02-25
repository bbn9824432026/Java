# Topic 1.6 — Primitive Data Types
## (Ranges, Defaults, Literals, Underscores, Hex/Binary)

---

## 1. Conceptual Explanation

### Why This Topic Matters on the Exam

Oracle tests primitive data types in ways that go far beyond just knowing the 8 types. The exam probes:
- Exact numeric ranges and overflow behavior
- Literal suffixes and what happens without them
- Numeric literal formats (decimal, hex, octal, binary)
- Underscore placement rules in literals
- Implicit vs explicit casting between types
- Compile-time vs runtime range violations

Every subtlety here has appeared on OCP exams in tricky code snippets.

---

### The 8 Primitive Types — Deep Dive

#### Integer Types

**`byte`**
- 8 bits, signed (two's complement)
- Range: -128 to 127
- Default (field): 0

```java
byte b = 127;    // max value — OK
byte b2 = 128;   // COMPILE ERROR — 128 is out of byte range
byte b3 = -128;  // min value — OK
byte b4 = -129;  // COMPILE ERROR
```

**`short`**
- 16 bits, signed
- Range: -32,768 to 32,767
- Default: 0
- Rarely used in practice — mostly legacy/binary protocol code

**`int`**
- 32 bits, signed
- Range: -2,147,483,648 to 2,147,483,647 (approximately ±2.1 billion)
- Default: 0
- **Default integer literal type** — any whole number literal without a suffix is an `int`

```java
int i = 2147483647;    // Integer.MAX_VALUE — OK
int i2 = 2147483648;   // COMPILE ERROR — too large for int literal
long l = 2147483648;   // COMPILE ERROR — the literal itself is still int-sized!
long l2 = 2147483648L; // OK — L suffix makes it a long literal
```

> **Critical trap:** `long l = 2147483648;` is a compile error even though `l` is a `long`. The problem is the **literal** `2147483648` — the compiler evaluates it as an `int` first, and it overflows `int` before assignment happens. You must add `L` suffix.

**`long`**
- 64 bits, signed
- Range: -2^63 to 2^63 - 1 (approximately ±9.2 × 10^18)
- Default: 0L
- Requires `L` or `l` suffix for literals exceeding int range
- Use uppercase `L` — lowercase `l` looks like the digit `1`

```java
long l1 = 100;       // OK — int literal 100 is widened to long
long l2 = 100L;      // OK — explicit long literal
long l3 = 100l;      // OK but bad practice — l looks like 1
```

---

#### Floating-Point Types

**`float`**
- 32 bits, IEEE 754 single-precision
- Approximately 7 significant decimal digits
- Default: 0.0f
- **Requires `f` or `F` suffix** — without it, decimal literals are `double`

```java
float f1 = 3.14;    // COMPILE ERROR — 3.14 is a double literal, loses precision
float f2 = 3.14f;   // OK
float f3 = 3.14F;   // OK
float f4 = (float) 3.14;  // OK — explicit cast
```

**`double`**
- 64 bits, IEEE 754 double-precision
- Approximately 15 significant decimal digits
- Default: 0.0
- **Default decimal literal type** — any decimal number without suffix is a `double`
- Optional `d` or `D` suffix (rarely used but valid)

```java
double d1 = 3.14;    // OK — double literal assigned to double
double d2 = 3.14d;   // OK — explicit double suffix
double d3 = 3.14D;   // OK
```

---

#### `char`

- 16 bits, **unsigned** (0 to 65,535)
- Represents a Unicode character
- Default: `'\u0000'` (null character)
- Can hold a single character, a Unicode escape, or an integer value 0-65535

```java
char c1 = 'A';           // character literal
char c2 = 65;            // integer literal — same as 'A'
char c3 = '\u0041';      // Unicode escape — same as 'A'
char c4 = '\n';          // escape sequence — newline
char c5 = -1;            // COMPILE ERROR — negative (char is unsigned)
char c6 = 65536;         // COMPILE ERROR — exceeds 65535
```

> **Exam trap:** `char` is the **only primitive that is unsigned**. It can hold values 0–65535. You can assign positive integer literals to `char` if they're in range. You CANNOT assign negative values.

```java
char c = 'A' + 1;  // 'B' — arithmetic on chars produces int, but fits in char
char c2 = 'A';
c2++;              // c2 is now 'B' — compound assignment handles the narrowing
```

---

#### `boolean`

- Size: JVM-defined (typically treated as 1 bit logically, but stored differently)
- Valid values: `true` and `false` ONLY
- Default: `false`

```java
boolean b1 = true;
boolean b2 = false;
boolean b3 = 1;       // COMPILE ERROR — Java is NOT C
boolean b4 = 0;       // COMPILE ERROR
boolean b5 = "true";  // COMPILE ERROR
boolean b6 = Boolean.parseBoolean("true");  // OK — runtime parsing
```

> **Critical trap:** Unlike C/C++, Java booleans are NOT numeric. `if (1)` is a compile error in Java. You cannot assign integers to booleans.

---

### Numeric Literal Formats

Java supports four numeric literal formats:

#### 1. Decimal (Base 10) — Default
```java
int i = 255;       // standard decimal
```

#### 2. Hexadecimal (Base 16) — Prefix `0x` or `0X`
Uses digits 0-9 and letters A-F (case-insensitive):
```java
int i = 0xFF;      // 255 in decimal
int i2 = 0xff;     // same — case insensitive
int i3 = 0XFF;     // same — X can be upper or lower case
System.out.println(0xFF);   // prints 255
```

#### 3. Octal (Base 8) — Prefix `0` (zero)
Uses digits 0-7:
```java
int i = 017;       // 15 in decimal (1×8 + 7 = 15)
int i2 = 010;      // 8 in decimal
int i3 = 08;       // COMPILE ERROR — 8 is not a valid octal digit
```

> **Exam trap:** Octal literals start with `0`. A number like `010` is NOT ten — it's eight. This is a classic trap.

#### 4. Binary (Base 2) — Prefix `0b` or `0B` (Java 7+)
```java
int i = 0b1010;    // 10 in decimal
int i2 = 0B1111;   // 15 in decimal
int i3 = 0b1010_1010;  // 170 — with underscores
```

**Quick conversion reference:**

| Decimal | Binary | Hex | Octal |
|---|---|---|---|
| 0 | 0b0000 | 0x0 | 00 |
| 8 | 0b1000 | 0x8 | 010 |
| 10 | 0b1010 | 0xA | 012 |
| 15 | 0b1111 | 0xF | 017 |
| 16 | 0b10000 | 0x10 | 020 |
| 255 | 0b11111111 | 0xFF | 0377 |

---

### Numeric Literal Suffixes

| Suffix | Type | Example |
|---|---|---|
| `L` or `l` | `long` | `100L` |
| `f` or `F` | `float` | `3.14f` |
| `d` or `D` | `double` | `3.14d` (optional) |
| none (whole number) | `int` | `42` |
| none (decimal) | `double` | `3.14` |

There is **no suffix for `byte`, `short`, or `char`**. You assign integer literals to them directly (if in range), and the compiler performs an implicit narrowing conversion for compile-time constants.

---

### Underscore in Numeric Literals (Java 7+)

Java 7 introduced the ability to place underscores inside numeric literals to improve readability. This is tested with specific placement rules.

```java
int million = 1_000_000;         // OK — readable
long creditCard = 1234_5678_9012_3456L;  // OK
int binary = 0b1010_1010;        // OK
int hex = 0xFF_EC_D1_8E;         // OK
float pi = 3.14_15_92f;          // OK
```

#### Underscore Placement Rules — Where Underscores CANNOT Go:

**1. At the beginning or end of a literal:**
```java
int i = _1000;    // COMPILE ERROR — starts with underscore (looks like identifier)
int i2 = 1000_;   // COMPILE ERROR — ends with underscore
```

**2. Adjacent to the decimal point:**
```java
double d = 3._14;   // COMPILE ERROR — next to decimal point
double d2 = 3_.14;  // COMPILE ERROR — next to decimal point
```

**3. Adjacent to the `L`, `f`, `d` suffix:**
```java
long l = 100_L;     // COMPILE ERROR — next to suffix
float f = 1.0_f;    // COMPILE ERROR — next to suffix
```

**4. Adjacent to the `0x`, `0b` prefix:**
```java
int i = 0_xFF;   // COMPILE ERROR — next to 0x prefix (between 0 and x)
int i2 = 0x_FF;  // COMPILE ERROR — right after 0x
int i3 = 0_b101; // COMPILE ERROR — between 0 and b
```

**What IS allowed:**
```java
int i = 0xFF_FF;     // OK — underscore inside hex digits
int i2 = 0b1010_0101; // OK — underscore inside binary digits
```

> **Memory trick:** Underscores can only appear **between digits**. Anything that is not a pure digit (decimal points, prefixes, suffixes) cannot have an underscore adjacent to it.

---

### Integer Overflow — Runtime Behavior

When integer arithmetic exceeds the range, Java wraps around silently. No exception is thrown:

```java
int max = Integer.MAX_VALUE;    // 2,147,483,647
System.out.println(max + 1);    // -2,147,483,648 — wraps around!

int min = Integer.MIN_VALUE;    // -2,147,483,648
System.out.println(min - 1);    // 2,147,483,647 — wraps around the other way
```

This is called **integer overflow** and is a runtime behavior, not a compile error. The JVM does not throw an exception for it.

> **Exam note:** The exam can show code that overflows and ask for the output. You need to recognize that overflow produces the wrapped value, not an exception.

---

### Numeric Type Promotion in Expressions

When you perform arithmetic, Java automatically promotes smaller types:

**Rules:**
1. `byte`, `short`, `char` are promoted to `int` before any arithmetic
2. If one operand is `long`, the other is promoted to `long`
3. If one operand is `float`, the other is promoted to `float`
4. If one operand is `double`, the other is promoted to `double`

```java
byte b1 = 10;
byte b2 = 20;
byte b3 = b1 + b2;   // COMPILE ERROR — b1 + b2 produces int, not byte

int result = b1 + b2;  // OK — int receives the promoted result
```

```java
int i = 5;
long l = 10L;
long result = i + l;   // i is promoted to long — result is long
```

```java
int i = 5;
double d = 2.5;
double result = i + d;  // i is promoted to double — result is double
```

---

### Casting Between Primitives

#### Widening Conversion (Implicit — No Cast Needed)
Goes from smaller to larger type. No data loss:
```
byte → short → int → long → float → double
                     char ↗
```

```java
int i = 100;
long l = i;      // implicit widening — OK
double d = i;    // implicit widening — OK
float f = i;     // implicit widening — OK
```

> **Exam trap:** `int` to `float` is widening but CAN lose precision because `float` has only 7 significant digits while `int` can have up to 10 digits. This is legal without a cast but may lose precision silently.

#### Narrowing Conversion (Explicit Cast Required)
Goes from larger to smaller type. May lose data:

```java
double d = 3.99;
int i = (int) d;    // explicit cast required — result is 3 (truncated, NOT rounded)
System.out.println(i);  // 3

long l = 300L;
byte b = (byte) l;  // explicit cast — may lose data
System.out.println(b);  // 44 (300 % 256 = 44 after wrapping)
```

> **Critical:** Casting a `double` to `int` **truncates** (removes the decimal part). It does NOT round. `(int) 3.99` is `3`, not `4`. `(int) -3.99` is `-3`, not `-4`.

#### Casting `char` ↔ `int`

```java
char c = 'A';
int i = c;           // widening — OK, i = 65
char c2 = (char) 66; // narrowing cast — c2 = 'B'
char c3 = 65;        // OK — compile-time constant, compiler checks range
```

---

### Special Float/Double Values

These exist but are rarely tested deeply on OCP:

```java
double posInf = Double.POSITIVE_INFINITY;   // 1.0/0.0
double negInf = Double.NEGATIVE_INFINITY;   // -1.0/0.0
double nan = Double.NaN;                    // 0.0/0.0

System.out.println(1.0 / 0.0);   // Infinity — no exception!
System.out.println(-1.0 / 0.0);  // -Infinity
System.out.println(0.0 / 0.0);   // NaN

// Integer division by zero IS an exception:
System.out.println(1 / 0);       // ArithmeticException: / by zero
```

> **Exam trap:** Floating-point division by zero does **not** throw an exception — it produces `Infinity` or `NaN`. Integer division by zero **does** throw `ArithmeticException`.

---

### The `var` and Literals Interaction

When using `var`, the type is inferred from the literal:

```java
var x = 42;      // inferred as int
var y = 42L;     // inferred as long
var z = 3.14;    // inferred as double
var w = 3.14f;   // inferred as float
var c = 'A';     // inferred as char
var b = true;    // inferred as boolean
```

---

## 2. Code Examples

### Example 1 — Range Violations at Compile Time

```java
public class RangeDemo {
    public static void main(String[] args) {
        byte b = 127;       // OK — max byte value
        byte b2 = 128;      // COMPILE ERROR — exceeds byte range

        short s = 32767;    // OK
        short s2 = 32768;   // COMPILE ERROR

        int i = 2147483647; // OK — Integer.MAX_VALUE
        // int i2 = 2147483648; // COMPILE ERROR — exceeds int range

        long l = 2147483648L; // OK — L suffix makes it long literal
    }
}
```

---

### Example 2 — Literal Type Confusion

```java
public class LiteralTypes {
    public static void main(String[] args) {
        // Default types
        int i = 42;           // int literal
        double d = 3.14;      // double literal (default for decimals)

        // Errors
        // float f = 3.14;    // COMPILE ERROR — 3.14 is double, not float
        float f = 3.14f;      // OK

        // long literal issue
        // long l = 9999999999; // COMPILE ERROR — too big for int literal
        long l = 9999999999L;  // OK

        // Widening is fine
        long l2 = 42;          // int literal 42 widened to long — OK
        double d2 = 42;        // int literal 42 widened to double — OK
    }
}
```

---

### Example 3 — Numeric Literal Formats

```java
public class LiteralFormats {
    public static void main(String[] args) {
        int decimal = 255;
        int hex     = 0xFF;
        int octal   = 0377;
        int binary  = 0b11111111;

        // All four are the same value
        System.out.println(decimal);  // 255
        System.out.println(hex);      // 255
        System.out.println(octal);    // 255
        System.out.println(binary);   // 255

        System.out.println(decimal == hex);    // true
        System.out.println(hex == octal);      // true
        System.out.println(octal == binary);   // true
    }
}
```

---

### Example 4 — Octal Trap

```java
public class OctalTrap {
    public static void main(String[] args) {
        int a = 010;    // OCTAL! Not 10. Equals 8.
        int b = 10;     // decimal 10
        System.out.println(a);      // 8
        System.out.println(a == b); // false

        // int c = 09;  // COMPILE ERROR — 9 is not a valid octal digit
    }
}
```

---

### Example 5 — Underscore Rules

```java
public class UnderscoreDemo {
    public static void main(String[] args) {
        // Valid
        int a = 1_000_000;
        long b = 1234_5678_9012L;
        float c = 3.14_15f;
        int hex = 0xFF_AA_BB;
        int bin = 0b1010_0101;

        // Invalid — won't compile
        // int d = _1000;       // starts with underscore
        // int e = 1000_;       // ends with underscore
        // double f = 3._14;    // next to decimal point
        // long g = 100_L;      // next to suffix
        // int h = 0_xFF;       // between 0 and x in 0x prefix
        // int i = 0x_FF;       // right after 0x
    }
}
```

---

### Example 6 — Arithmetic Promotion

```java
public class Promotion {
    public static void main(String[] args) {
        byte b1 = 10, b2 = 20;
        // byte sum = b1 + b2;  // COMPILE ERROR — result is int
        int sum = b1 + b2;      // OK

        char c = 'A';
        // char c2 = c + 1;     // COMPILE ERROR — char + int = int
        char c2 = (char)(c + 1); // OK — explicit cast
        System.out.println(c2);   // B

        int i = 10;
        long l = 20L;
        // int result = i + l;  // COMPILE ERROR — result is long
        long result = i + l;    // OK
    }
}
```

---

### Example 7 — Narrowing Cast Behavior

```java
public class NarrowingDemo {
    public static void main(String[] args) {
        // Truncation, not rounding
        double d = 9.99;
        int i = (int) d;
        System.out.println(i);   // 9 — NOT 10

        double neg = -9.99;
        int i2 = (int) neg;
        System.out.println(i2);  // -9 — truncates toward zero

        // Overflow during narrowing
        long big = 300L;
        byte b = (byte) big;
        System.out.println(b);   // 44 — (300 - 256 = 44)

        long tooBig = 256L;
        byte b2 = (byte) tooBig;
        System.out.println(b2);  // 0 — 256 wraps to 0
    }
}
```

---

### Example 8 — Overflow Wrapping

```java
public class Overflow {
    public static void main(String[] args) {
        int max = Integer.MAX_VALUE;
        System.out.println(max);      // 2147483647
        System.out.println(max + 1);  // -2147483648 — wraps to MIN_VALUE

        int min = Integer.MIN_VALUE;
        System.out.println(min - 1);  // 2147483647 — wraps to MAX_VALUE
    }
}
```

---

### Example 9 — Division by Zero

```java
public class DivisionByZero {
    public static void main(String[] args) {
        // Integer division by zero — EXCEPTION
        // System.out.println(5 / 0);      // ArithmeticException at runtime

        // Floating-point division by zero — NOT an exception
        System.out.println(5.0 / 0);      // Infinity
        System.out.println(-5.0 / 0);     // -Infinity
        System.out.println(0.0 / 0.0);    // NaN
        System.out.println(Double.isNaN(0.0 / 0.0));  // true
    }
}
```

---

## 3. Exam-Style Questions

---

**Q1 — MCQ**
What is the output?
```java
public class Test {
    public static void main(String[] args) {
        int x = 010;
        System.out.println(x);
    }
}
```
- A. 10
- B. 8
- C. 2
- D. Compile error

**Answer: B**
`010` is an octal literal. 1×8 + 0 = 8.

---

**Q2 — MCQ**
Which of the following lines causes a compile error?
```java
A. long l = 2147483648L;
B. long l = 2147483648;
C. int i = 0xFF;
D. float f = 3.14f;
```

**Answer: B**
`2147483648` without the `L` suffix is interpreted as an `int` literal, and it exceeds `Integer.MAX_VALUE`. This is a compile error. Line A adds `L` suffix making it valid. C and D are valid.

---

**Q3 — Select All That Apply**
Which of the following are valid `long` literals? (Choose all that apply)

- A. `123456789012345L`
- B. `123456789012345`
- C. `100l`
- D. `0xFFFFL`
- E. `100_000_L`

**Answer: A, C, D**
B: the number exceeds `int` range so without `L` it's a compile error. C: lowercase `l` is valid (bad practice but legal). E: underscore adjacent to suffix `L` is illegal.

---

**Q4 — MCQ**
What is the output?
```java
public class Test {
    public static void main(String[] args) {
        byte b = 127;
        b++;
        System.out.println(b);
    }
}
```
- A. 127
- B. 128
- C. -128
- D. Compile error

**Answer: C**
`b++` is equivalent to `b = (byte)(b + 1)`. 127 + 1 = 128, but cast to byte wraps to -128. Compound operators include an implicit cast back to the original type.

---

**Q5 — MCQ**
What is the output?
```java
public class Test {
    public static void main(String[] args) {
        System.out.println(1_000_000);
        System.out.println(0b1010);
        System.out.println(0_10);
    }
}
```
- A. 1000000, 10, 10
- B. 1000000, 10, 8
- C. 1_000_000, 0b1010, 0_10
- D. Compile error

**Answer: B**
`1_000_000` = one million. `0b1010` = binary 10. `0_10` = octal (starts with 0), and `010` octal = 8. The underscore in `0_10` is between the prefix `0` and the digits `10` — wait, is this valid? `0_10` — the `0` is the octal prefix and `10` are the octal digits. The underscore is between `0` and `1` which are both digits — this is valid. So `0_10` = octal `010` = 8. Answer is B.

---

**Q6 — MCQ**
Which is the result?
```java
double d = 7.0 / 0;
System.out.println(d);
```
- A. Compile error
- B. `ArithmeticException`
- C. `Infinity`
- D. `NaN`

**Answer: C**
`0` is an `int` literal, but `7.0` is `double`, so the division is floating-point. Floating-point division by zero yields `Infinity`. No exception.

---

**Q7 — Select All That Apply**
Which underscore placements are valid? (Choose all that apply)

- A. `int a = 1_000;`
- B. `int b = _1000;`
- C. `int c = 1000_;`
- D. `double d = 3.1_4;`
- E. `double e = 3._14;`
- F. `long f = 100_000L;`
- G. `int g = 0x_FF;`
- H. `int h = 0xFF_00;`

**Answer: A, D, F, H**
B starts with underscore (looks like identifier). C ends with underscore. E has underscore adjacent to decimal point. G has underscore right after `0x` prefix. All others are valid.

---

**Q8 — MCQ**
What is the output?
```java
public class Test {
    public static void main(String[] args) {
        int i = Integer.MAX_VALUE;
        long l = i + 1;
        System.out.println(l);
    }
}
```
- A. 2147483648
- B. -2147483648
- C. Compile error
- D. ArithmeticException

**Answer: B**
`i + 1` is computed as `int + int` = `int`. The `int` overflows to `-2147483648`. Then the overflowed `int` value is widened to `long`. The widening happens **after** the overflow. To get the correct value, you'd need `(long)i + 1`.

---

**Q9 — MCQ**
```java
byte b1 = 10;
byte b2 = 20;
byte b3 = b1 + b2;
```
What happens?

- A. `b3 = 30` — compiles and runs fine
- B. Compile error — result of `b1 + b2` is `int`
- C. Runtime exception — overflow
- D. `b3 = -26` — byte overflow

**Answer: B**
`byte + byte` promotes both to `int`. The result is `int`, which cannot be assigned to `byte` without an explicit cast. Compile error.

---

**Q10 — MCQ**
What is the result of `(int) -3.9`?

- A. -4
- B. -3
- C. 3
- D. Compile error

**Answer: B**
Casting a double to int **truncates toward zero**. `-3.9` truncated toward zero is `-3`.

---

## 4. Trick Analysis

### Trap 1: `010` is octal, not ten
Any integer literal starting with `0` followed by more digits is **octal**. `010` = 8, `07` = 7, `011` = 9. This is a classic exam trick embedded in larger code snippets where the octal literal is easy to miss.

---

### Trap 2: `long l = 2147483648;` doesn't compile
Even though the target type is `long`, the literal `2147483648` is evaluated as an `int` first. Since it exceeds `Integer.MAX_VALUE`, it's a compile error. You need `2147483648L`. Many candidates think the assignment to `long` "saves" the literal — it doesn't.

---

### Trap 3: `float f = 3.14;` doesn't compile
Decimal literals default to `double`. Assigning a `double` literal to `float` loses precision — the compiler rejects it. You need `3.14f` or `(float)3.14`.

---

### Trap 4: `byte b = b1 + b2;` doesn't compile
`byte + byte` yields `int`. This surprises almost everyone. The rule is: `byte`, `short`, and `char` are always promoted to `int` before arithmetic. The result must go back to an `int` variable or be explicitly cast.

---

### Trap 5: `b++` wraps silently
Compound assignment operators (`++`, `+=`, `-=`, etc.) include an implicit narrowing cast. So `byte b = 127; b++;` compiles and produces `-128`. This is different from `b = b + 1;` which would require an explicit cast.

---

### Trap 6: Integer overflow is silent
No exception for `Integer.MAX_VALUE + 1`. It wraps to `Integer.MIN_VALUE`. The exam shows this and asks for the output — candidates who expect an exception get it wrong.

---

### Trap 7: Floating-point division by zero is not an exception
`5.0 / 0` = `Infinity`. `0.0 / 0.0` = `NaN`. Only integer division by zero throws `ArithmeticException`. The exam pairs these to test if you know the distinction.

---

### Trap 8: Narrowing truncates, doesn't round
`(int) 9.99` = `9`, not `10`. `(int) -9.99` = `-9`, not `-10`. Truncation always goes toward zero.

---

### Trap 9: Underscore adjacent to non-digits
The rule is simple — underscores can only be between digits. But what counts as "digits" in context? In `0xFF_00`, both `FF` and `00` are hex digits — the underscore is between them. In `0x_FF`, the `_` is right after `x` which is part of the prefix — not a digit. In `3._14`, `_` is right after the decimal point — not a digit.

---

### Trap 10: `i + 1L` vs `(long)i + 1`
```java
int i = Integer.MAX_VALUE;
long a = i + 1;          // overflows as int THEN widens — wrong result
long b = (long)i + 1;    // widens FIRST then adds — correct result
long c = i + 1L;         // 1L is long, so i is promoted to long — correct
```
Knowing when the promotion/widening happens is critical.

---

## 5. Summary Sheet

### The 8 Primitives at a Glance

| Type | Bits | Range | Default | Literal Suffix |
|---|---|---|---|---|
| `byte` | 8 | -128 to 127 | 0 | none |
| `short` | 16 | -32,768 to 32,767 | 0 | none |
| `int` | 32 | ~±2.1 billion | 0 | none (default integer) |
| `long` | 64 | ~±9.2 × 10^18 | 0L | `L` or `l` |
| `float` | 32 | ~7 decimal digits | 0.0f | `f` or `F` (**required**) |
| `double` | 64 | ~15 decimal digits | 0.0 | `d` or `D` (optional) |
| `char` | 16 | 0 to 65,535 | '\u0000' | none (uses `'x'`) |
| `boolean` | JVM | true/false | false | none |

---

### Literal Formats

| Format | Prefix | Valid Digits | Example |
|---|---|---|---|
| Decimal | none | 0-9 | `255` |
| Hexadecimal | `0x` or `0X` | 0-9, A-F | `0xFF` |
| Octal | `0` | 0-7 | `0377` |
| Binary | `0b` or `0B` | 0-1 | `0b11111111` |

---

### Underscore Rules (One Rule to Memorize)
**Underscores may only appear BETWEEN digits.**
Cannot be at start, end, adjacent to `.`, adjacent to suffix (`L`, `f`), or adjacent to prefix (`0x`, `0b`).

---

### Widening Hierarchy (No Cast Needed)
```
byte → short → int → long → float → double
char → int
```

### Narrowing (Cast Required — May Lose Data)
Any direction opposite to above.

---

### Key Behaviors

| Scenario | Result |
|---|---|
| Integer overflow | Silent wrap-around (no exception) |
| `float` division by zero | `Infinity` (no exception) |
| Integer division by zero | `ArithmeticException` |
| `(int) 3.9` | `3` — truncates toward zero |
| `(int) -3.9` | `-3` — truncates toward zero |
| `byte + byte` | `int` result |
| `b++` on byte 127 | `-128` — implicit cast wraps |
| `boolean = 1` | **Compile error** — not C |

---
