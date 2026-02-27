# Topic 2.3 — Binary Operators
## (Arithmetic, String Concatenation)

---

## 1. Conceptual Explanation

### What Are Binary Operators?

Binary operators operate on **two operands** — a left-hand side and a right-hand side. This topic covers:

1. **Arithmetic operators** — `+`, `-`, `*`, `/`, `%`
2. **String concatenation** — the `+` operator in string context

These appear in virtually every OCP exam question — either directly or embedded within larger expressions involving casting, promotion, overflow, and string conversion.

---

### The Five Arithmetic Operators

| Operator | Name | Example | Result |
|---|---|---|---|
| `+` | Addition | `5 + 3` | `8` |
| `-` | Subtraction | `5 - 3` | `2` |
| `*` | Multiplication | `5 * 3` | `15` |
| `/` | Division | `5 / 3` | `1` (integer division) |
| `%` | Modulo (remainder) | `5 % 3` | `2` |

---

### Integer Division — Truncates Toward Zero

Integer division discards the fractional part — it does **not** round:

```java
System.out.println(7 / 2);     // 3 — NOT 3.5
System.out.println(-7 / 2);    // -3 — truncates toward zero, NOT -4
System.out.println(7 / -2);    // -3
System.out.println(-7 / -2);   // 3
```

> **Critical rule:** Integer division always truncates **toward zero**. This means:
> - Positive result: removes decimal digits (floor)
> - Negative result: rounds TOWARD zero (ceiling for negatives), NOT toward negative infinity

```java
System.out.println(5 / 2);     // 2 — truncate
System.out.println(-5 / 2);    // -2 — toward zero (NOT -3)
System.out.println(5 / -2);    // -2 — toward zero
System.out.println(-5 / -2);   //  2 — two negatives → positive, truncate
```

---

### Integer Division by Zero — Exception

```java
int a = 5 / 0;        // ArithmeticException: / by zero at RUNTIME
int b = 0 / 0;        // ArithmeticException: / by zero at RUNTIME
```

This is **not** a compile error — the compiler does not evaluate literal `0` division. It's always a runtime exception.

> **Exam trap:** `5 / 0` compiles. `5.0 / 0` does NOT throw — it produces `Infinity`. Only integer division by zero throws `ArithmeticException`.

---

### The Modulo `%` Operator

Modulo returns the **remainder** after division.

```java
System.out.println(7 % 3);     // 1 — (7 = 2×3 + 1)
System.out.println(10 % 5);    // 0 — exactly divisible
System.out.println(3 % 7);     // 3 — (3 = 0×7 + 3)
```

#### Modulo with Negative Numbers

The sign of the result follows the **sign of the dividend** (left operand):

```java
System.out.println(-7 % 3);    // -1  — dividend is negative → result negative
System.out.println(7 % -3);    //  1  — dividend is positive → result positive
System.out.println(-7 % -3);   // -1  — dividend is negative → result negative
```

**Formula:** `a % b = a - (a/b) * b` where `/` is integer division (truncating toward zero)

Verify: `-7 % 3 = -7 - (-7/3)*3 = -7 - (-2)*3 = -7 + 6 = -1` ✅

> **Exam trap:** Many candidates assume `%` result sign follows the divisor. It follows the **dividend**. `-7 % 3 = -1`, NOT `2`.

#### Modulo with Floating-Point

`%` works on floating-point types too:

```java
System.out.println(5.5 % 2.1);  // approximately 1.3
System.out.println(5.0 % 2.0);  // 1.0
System.out.println(-5.0 % 2.0); // -1.0
```

Floating-point modulo does NOT throw for `% 0.0` — it produces `NaN`:

```java
System.out.println(5.0 % 0.0);  // NaN — no exception
System.out.println(5 % 0);      // ArithmeticException — integer!
```

---

### Numeric Type Promotion in Binary Operations

When two operands of different types are combined, **the smaller type is promoted** to match the larger:

**Promotion rules (applied in order):**
1. If either operand is `double` → both become `double`
2. Else if either is `float` → both become `float`
3. Else if either is `long` → both become `long`
4. Else both become `int` (even if both are `byte`, `short`, or `char`)

```java
byte b = 10;
short s = 20;
int i = 30;
long l = 40L;
float f = 1.5f;
double d = 2.5;

// byte + short → int
var r1 = b + s;    // int

// int + long → long
var r2 = i + l;    // long

// long + float → float
var r3 = l + f;    // float

// float + double → double
var r4 = f + d;    // double

// byte + byte → int (Rule 4!)
var r5 = b + b;    // int — NOT byte
```

> **Exam trap:** `byte + byte = int`. Even when BOTH operands are `byte`, the result is `int`. This trips up many candidates.

---

### Overflow in Arithmetic

When arithmetic results exceed the type's range, **overflow wraps silently** — no exception:

```java
int max = Integer.MAX_VALUE;    // 2,147,483,647
System.out.println(max + 1);    // -2,147,483,648 — wraps to MIN_VALUE

long lmax = Long.MAX_VALUE;
System.out.println(lmax + 1);   // -9223372036854775808 — wraps

// To avoid overflow, use long arithmetic:
long safe = (long) max + 1;     // 2,147,483,648 — correct
```

---

### Mixed Integer and Floating-Point Arithmetic

When an integer and a floating-point value are combined, the integer is widened to the floating-point type:

```java
int i = 5;
double d = 2.0;
double result = i / d;   // i promoted to double: 5.0 / 2.0 = 2.5

// But watch this:
double trap = 5 / 2;     // 5/2 is INTEGER division = 2, then widened to 2.0
System.out.println(trap); // 2.0 — NOT 2.5
```

> **Critical trap:** `5 / 2` is evaluated as integer division first (= 2) and THEN the result is widened to `double` (= `2.0`). The widening happens AFTER the division, not before.

To force floating-point division, at least one operand must be floating-point at the time of division:

```java
double correct = 5.0 / 2;    // 2.5 — one operand is double
double correct2 = 5 / 2.0;   // 2.5
double correct3 = (double)5 / 2;  // 2.5 — cast before division
double wrong = (double)(5 / 2);   // 2.0 — cast AFTER division
```

---

### The `+` Operator for String Concatenation

When either operand of `+` is a `String`, the operation becomes **string concatenation**. The non-String operand is converted to its String representation.

#### Conversion Rules for Concatenation

| Type | Converted To |
|---|---|
| `int`, `long`, etc. | Decimal string: `42` → `"42"` |
| `double`, `float` | Decimal string: `3.14` → `"3.14"` |
| `boolean` | `"true"` or `"false"` |
| `char` | The character itself: `'A'` → `"A"` |
| `null` reference | `"null"` |
| Object reference | `obj.toString()` is called |

```java
System.out.println("Value: " + 42);       // "Value: 42"
System.out.println("Flag: " + true);      // "Flag: true"
System.out.println("Char: " + 'A');       // "Char: A"
System.out.println("Null: " + null);      // "Null: null"

Object obj = null;
System.out.println("Obj: " + obj);        // "Obj: null"
```

---

### String Concatenation Left-to-Right Evaluation

The `+` operator is **left-associative** — expressions are evaluated left to right. This means:

- If the **leftmost** operand is a String, ALL subsequent `+` become concatenation
- If the **leftmost** operand is numeric, `+` performs addition until a String is encountered

```java
// Left-to-right evaluation
System.out.println(1 + 2 + "3");    // "33" — 1+2=3(int), 3+"3"="33"
System.out.println("1" + 2 + 3);    // "123" — "1"+2="12", "12"+3="123"
System.out.println(1 + "2" + 3);    // "123" — 1+"2"="12", "12"+3="123"

// Parentheses change grouping
System.out.println(1 + (2 + "3"));  // "123" — 2+"3"="23", 1+"23"="123"
System.out.println((1 + 2) + "3");  // "33"  — 1+2=3, 3+"3"="33"
```

---

### `char` in String Concatenation vs Arithmetic

`char` behaves differently depending on context:

```java
char a = 'A';   // 65
char b = 'B';   // 66

// char + char = int (arithmetic promotion)
System.out.println(a + b);       // 131 — numeric addition

// char + String = String (concatenation)
System.out.println(a + "B");     // "AB" — concatenation
System.out.println("" + a + b);  // "AB" — both concatenated

// int + String vs char + String
System.out.println(65 + "A");    // "65A" — int converts to "65"
System.out.println('A' + "A");   // "AA" — char converts to "A"
```

> **Critical distinction:** `char + char` is integer addition. `char + String` is string concatenation. This is a very common exam trap.

---

### String Concatenation with `null`

```java
String s = null;
System.out.println("Hello " + s);   // "Hello null"
System.out.println(s + " World");   // "null World"
System.out.println(s + s);          // "nullnull"

// But calling a method on null throws NPE:
System.out.println(s.toString());   // NullPointerException
```

The `+` operator handles `null` gracefully — it converts it to the string `"null"`. Calling methods on null still throws `NullPointerException`.

---

### Compound Assignment Operators

Binary operators have corresponding compound assignment operators. As covered in Topic 2.1, these include an **implicit narrowing cast**:

```java
int x = 10;
x += 5;    // x = x + 5 = 15
x -= 3;    // x = x - 3 = 12
x *= 2;    // x = x * 2 = 24
x /= 4;    // x = x / 4 = 6
x %= 4;    // x = x % 4 = 2

// With narrowing cast for smaller types:
byte b = 100;
b += 50;   // OK — (byte)(100 + 50) = (byte)150 = -106 (overflow, but compiles)
```

---

### Division and Modulo Relationship

The relationship between `/` and `%` is always:

```
(a / b) * b + (a % b) == a
```

This holds for all non-zero `b`, including negatives:

```java
int a = -7, b = 3;
int div = a / b;   // -2 (truncates toward zero)
int mod = a % b;   // -1
// Verify: (-2) * 3 + (-1) = -6 + (-1) = -7 ✅
System.out.println(div * b + mod == a);  // true
```

---

### `+` on `char` — The Tricky Cases

```java
System.out.println('a' + 'b');      // 195 — int arithmetic (97 + 98)
System.out.println("" + 'a' + 'b'); // "ab" — concatenation
System.out.println('a' + "" + 'b'); // "ab" — 'a'+"" = "a", "a"+'b' = "ab"
System.out.println('a' + 'b' + ""); // "195" — 97+98=195(int), 195+""="195"
```

The position of the String relative to the `char` operands completely changes the behavior.

---

### Numeric Promotion — Complete Rules for Binary Operators

This table covers ALL cases for binary arithmetic:

| Left Operand | Right Operand | Result Type |
|---|---|---|
| `byte` | `byte` | `int` |
| `byte` | `short` | `int` |
| `byte` | `int` | `int` |
| `byte` | `long` | `long` |
| `byte` | `float` | `float` |
| `byte` | `double` | `double` |
| `short` | `short` | `int` |
| `int` | `int` | `int` |
| `int` | `long` | `long` |
| `int` | `float` | `float` |
| `int` | `double` | `double` |
| `long` | `long` | `long` |
| `long` | `float` | `float` |
| `long` | `double` | `double` |
| `float` | `float` | `float` |
| `float` | `double` | `double` |
| `double` | `double` | `double` |

---

## 2. Code Examples

### Example 1 — Integer Division Truncation

```java
public class IntDivision {
    public static void main(String[] args) {
        // Positive division
        System.out.println(10 / 3);    // 3
        System.out.println(10 / 4);    // 2
        System.out.println(1 / 2);     // 0 — truncates, NOT rounds

        // Negative division — toward zero
        System.out.println(-10 / 3);   // -3 (NOT -4)
        System.out.println(10 / -3);   // -3
        System.out.println(-10 / -3);  //  3

        // Common mistake: expecting floating point
        double d = 5 / 2;     // integer division FIRST → 2, widened to 2.0
        System.out.println(d); // 2.0 — NOT 2.5

        double correct = 5.0 / 2;  // 2.5
        System.out.println(correct);
    }
}
```

---

### Example 2 — Modulo with Negatives

```java
public class ModuloDemo {
    public static void main(String[] args) {
        // Sign follows dividend (left operand)
        System.out.println(7 % 3);     //  1
        System.out.println(-7 % 3);    // -1
        System.out.println(7 % -3);    //  1
        System.out.println(-7 % -3);   // -1

        // Useful modulo patterns
        System.out.println(10 % 2);    // 0 — even number check
        System.out.println(11 % 2);    // 1 — odd number check
        System.out.println(7 % 10);    // 7 — when divisor > dividend

        // Floating-point modulo
        System.out.println(5.5 % 2.0); // 1.5
        System.out.println(5 % 0);     // ArithmeticException — comment out to run others
    }
}
```

---

### Example 3 — Type Promotion

```java
public class TypePromotion {
    public static void main(String[] args) {
        byte b1 = 10, b2 = 20;

        // byte + byte = int
        // byte sum = b1 + b2;   // COMPILE ERROR
        int sum = b1 + b2;        // OK
        var auto = b1 + b2;       // inferred as int

        // short + int = int
        short s = 100;
        int r2 = s + 50;

        // int + long = long
        int i = Integer.MAX_VALUE;
        long safe = (long) i + 1;  // correct — cast before addition
        long overflow = i + 1;     // wraps as int first, then widens to long
        System.out.println(safe);     // 2147483648
        System.out.println(overflow); // -2147483648 — overflow then widen

        // int + double = double
        double d = 5 / 2;     // 2.0 — int division, then widen
        double d2 = 5 / 2.0;  // 2.5 — double operand forces float division
        System.out.println(d);   // 2.0
        System.out.println(d2);  // 2.5
    }
}
```

---

### Example 4 — String Concatenation Deep Dive

```java
public class StringConcat {
    public static void main(String[] args) {
        // Left-to-right evaluation
        System.out.println(1 + 2 + "3");      // "33"
        System.out.println("3" + 1 + 2);      // "312"
        System.out.println("3" + (1 + 2));    // "33"
        System.out.println(1 + "2" + 3);      // "123"

        // char behavior
        System.out.println('A' + 'B');         // 131 — int addition
        System.out.println("" + 'A' + 'B');   // "AB" — concatenation
        System.out.println('A' + 'B' + "");   // "131" — addition first
        System.out.println('A' + "" + 'B');   // "AB" — each char concat'd

        // null in concatenation
        String s = null;
        System.out.println("val: " + s);       // "val: null"
        System.out.println(s + s);             // "nullnull"

        // boolean concatenation
        System.out.println("Flag: " + true);   // "Flag: true"
        System.out.println("Sum: " + (1 == 1)); // "Sum: true"
    }
}
```

---

### Example 5 — The `double d = 5/2` Classic Trap

```java
public class DivisionTrap {
    public static void main(String[] args) {
        double a = 5 / 2;           // 2.0 — integer division, THEN widen
        double b = 5 / 2.0;         // 2.5 — one double operand
        double c = 5.0 / 2;         // 2.5 — one double operand
        double d = (double)(5 / 2); // 2.0 — cast AFTER division
        double e = (double)5 / 2;   // 2.5 — cast BEFORE division

        System.out.println(a);  // 2.0
        System.out.println(b);  // 2.5
        System.out.println(c);  // 2.5
        System.out.println(d);  // 2.0
        System.out.println(e);  // 2.5
    }
}
```

---

### Example 6 — Overflow Scenarios

```java
public class OverflowDemo {
    public static void main(String[] args) {
        // int overflow
        int max = Integer.MAX_VALUE;
        System.out.println(max + 1);        // -2147483648

        // Multiplication overflow
        int bigProduct = 1_000_000 * 1_000_000;
        System.out.println(bigProduct);     // -727379968 — overflow!

        // Use long to avoid
        long safeProduct = 1_000_000L * 1_000_000;
        System.out.println(safeProduct);    // 1000000000000

        // int division never overflows EXCEPT:
        System.out.println(Integer.MIN_VALUE / -1);  // -2147483648 — overflow!
        // MIN_VALUE = -2147483648, dividing by -1 should give 2147483648
        // but that overflows int, so result is -2147483648 again
    }
}
```

---

### Example 7 — Compound Assignment with Narrowing

```java
public class CompoundNarrowing {
    public static void main(String[] args) {
        byte b = 100;

        b += 27;    // (byte)(100+27) = (byte)127 — OK, max byte
        System.out.println(b);  // 127

        b += 1;     // (byte)(127+1) = (byte)128 = -128 — wraps
        System.out.println(b);  // -128

        // Without compound: compile error
        // b = b + 1;  // COMPILE ERROR — b+1 is int, needs cast

        // Correct explicit version
        b = (byte)(b + 1);  // OK
        System.out.println(b);  // -127
    }
}
```

---

### Example 8 — Concatenation with Object's `toString()`

```java
public class ToStringConcat {
    static class Point {
        int x, y;
        Point(int x, int y) { this.x = x; this.y = y; }

        @Override
        public String toString() {
            return "(" + x + "," + y + ")";
        }
    }

    public static void main(String[] args) {
        Point p = new Point(3, 4);
        System.out.println("Point: " + p);   // "Point: (3,4)" — toString() called

        Point nullPoint = null;
        System.out.println("Point: " + nullPoint); // "Point: null" — no NPE
        // System.out.println(nullPoint.toString()); // NPE
    }
}
```

---

## 3. Exam-Style Questions

---

**Q1 — MCQ**
What is the output?
```java
System.out.println(7 / 2);
System.out.println(7.0 / 2);
System.out.println(7 / 2.0);
System.out.println((double)(7 / 2));
```
- A. `3` / `3.5` / `3.5` / `3.5`
- B. `3` / `3.5` / `3.5` / `3.0`
- C. `3.5` / `3.5` / `3.5` / `3.5`
- D. `3` / `3.0` / `3.5` / `3.0`

**Answer: B**
`7/2 = 3` (int). `7.0/2 = 3.5`. `7/2.0 = 3.5`. `(double)(7/2) = (double)3 = 3.0`.

---

**Q2 — MCQ**
What is the output?
```java
System.out.println(-7 % 3);
System.out.println(7 % -3);
System.out.println(-7 % -3);
```
- A. `-1` / `1` / `-1`
- B. `2` / `-2` / `2`
- C. `-1` / `-1` / `-1`
- D. `1` / `1` / `1`

**Answer: A**
Sign follows the dividend. `-7%3 = -1`, `7%-3 = 1`, `-7%-3 = -1`.

---

**Q3 — MCQ**
What is the output?
```java
System.out.println(1 + 2 + "3" + 4);
```
- A. `1234`
- B. `334`
- C. `3304`
- D. `3034`

**Answer: B**
`1+2=3` (int), `3+"3"="33"`, `"33"+4="334"`.

---

**Q4 — Select All That Apply**
Which lines compile without error? (Choose all that apply)
```java
A. byte b = 10; b = b + 1;
B. byte b = 10; b += 1;
C. byte b = 10; b++;
D. short s = 10; s = s * 2;
E. short s = 10; s *= 2;
F. int i = Integer.MAX_VALUE; i++;
```

**Answer: B, C, E, F**
A: `b+1` is int, needs explicit cast — ERROR.
B: compound `+=` has implicit cast — OK.
C: `++` has implicit cast — OK.
D: `s*2` is int, needs cast — ERROR.
E: compound `*=` has implicit cast — OK.
F: compiles fine, wraps at runtime — OK.

---

**Q5 — MCQ**
What is the output?
```java
System.out.println('A' + 'B');
System.out.println("" + 'A' + 'B');
System.out.println('A' + 'B' + "");
```
- A. `AB` / `AB` / `AB`
- B. `131` / `AB` / `131`
- C. `131` / `AB` / `AB`
- D. `AB` / `AB` / `131`

**Answer: B**
`'A'+'B'` = `65+66` = `131`. `""+'A'+'B'` = `"AB"`. `'A'+'B'+""` = `131+""` = `"131"`.

---

**Q6 — MCQ**
```java
int a = -5;
int b = 3;
System.out.println(a / b);
System.out.println(a % b);
System.out.println((a/b)*b + (a%b));
```
- A. `-2` / `-1` / `-5`
- B. `-1` / `-2` / `-5`
- C. `-2` / `1` / `-5`
- D. `-2` / `-1` / `-4`

**Answer: A**
`-5/3 = -1`... wait. `-5/3` truncates toward zero: `-5/3 = -1` (not -2). Let me recalculate:
- `-5 / 3`: truncate toward zero → `-1` (since `-1.67` truncates to `-1`)
- `-5 % 3`: `-5 - (-1)*3 = -5+3 = -2`
- `(-1)*3 + (-2) = -3 + (-2) = -5` ✅

**Corrected Answer: B** — `-1` / `-2` / `-5`.

---

**Q7 — MCQ**
What is the output?
```java
double d = 5 / 2;
System.out.println(d);
```
- A. `2`
- B. `2.0`
- C. `2.5`
- D. Compile error

**Answer: B**
`5/2` performs integer division = `2`. Then widened to `double` = `2.0`. NOT `2.5`.

---

**Q8 — Select All That Apply**
Which produce `NaN` or `Infinity`? (Choose all that apply)
```java
A. 1.0 / 0.0
B. -1.0 / 0.0
C. 0.0 / 0.0
D. 1 / 0
E. 1.0 % 0.0
F. 0.0 % 0.0
```

**Answer: A, B, C, E, F**
D throws `ArithmeticException` — only integer `/` or `%` by zero throws.
A → `Infinity`. B → `-Infinity`. C → `NaN`. E → `NaN`. F → `NaN`.

---

**Q9 — MCQ**
```java
int i = 1_000_000;
int j = i * i;
System.out.println(j);
```
- A. `1000000000000`
- B. `1000000000`
- C. A negative number due to overflow
- D. Compile error

**Answer: C**
`1_000_000 * 1_000_000 = 1_000_000_000_000` but `int` max is ~2.1 billion. The result overflows silently, producing a negative number (`-727379968`).

---

**Q10 — MCQ**
```java
String s = null;
System.out.println("Hello " + s);
System.out.println(s + 5 + 3);
```
- A. `Hello null` / `nullnull`
- B. `Hello null` / `null53`
- C. NullPointerException
- D. `Hello null` / `null8`

**Answer: B**
`"Hello " + null = "Hello null"`. `null + 5 = "null5"`, `"null5" + 3 = "null53"`. Left-to-right, null becomes "null" string first.

---

**Q11 — MCQ**
```java
byte b = 10;
byte c = 20;
var result = b + c;
System.out.println(((Object)result).getClass().getSimpleName());
```
- A. `Byte`
- B. `Short`
- C. `Integer`
- D. `Long`

**Answer: C**
`byte + byte` promotes to `int`. `var` infers `int`. Boxing to `Integer`. `getSimpleName()` returns `"Integer"`.

---

## 4. Trick Analysis

### Trap 1: Integer division truncates toward zero, not floor
`-7 / 2` is `-3`, NOT `-4`. Truncation toward zero means removing the decimal part. For negative results this is different from "floor" (which would round toward negative infinity). The exam uses negative division specifically to catch candidates who apply floor instead of truncation.

---

### Trap 2: `double d = 5 / 2` is `2.0`, not `2.5`
The division `5 / 2` is evaluated as integer division first, producing `2`. THEN the `2` is widened to `double` (`2.0`). The declared type of `d` does NOT retroactively change how the right-hand side is evaluated. To get `2.5`, at least one operand must be `double` or `float` before or during division.

---

### Trap 3: Sign of `%` result follows the dividend
`-7 % 3 = -1` (dividend `-7` is negative → result is negative). `7 % -3 = 1` (dividend `7` is positive → result is positive). Candidates wrongly assume the sign follows the divisor. Remember: **dividend determines sign**.

---

### Trap 4: `char + char` is arithmetic, not concatenation
`'A' + 'B'` = `131` (numeric addition of ASCII values). To concatenate chars: `"" + 'A' + 'B'` or use `String.valueOf()`. The exam places char addition where candidates expect string concatenation.

---

### Trap 5: `'A' + 'B' + ""` vs `"" + 'A' + 'B'`
Left-to-right evaluation:
- `'A' + 'B' + ""` → `65+66=131`, then `131+"" = "131"` — NUMERIC string
- `"" + 'A' + 'B'` → `""+65="A"`, then `"A"+66="AB"` — wait, `""+'A'` = `"A"` (char converts to character string), then `"A"+'B'` = `"AB"`

This is subtle: when a `char` is concatenated with a String, it contributes its CHARACTER (not its numeric value). When two `char`s are added (both numeric), they add numerically.

---

### Trap 6: `long safe = i + 1` still overflows if `i` is `int`
Even if you assign to `long`, if `i` is `int` and `1` is `int`, the addition happens as `int` arithmetic FIRST, overflows, THEN the overflowed `int` is widened to `long`. The fix is `(long)i + 1` or `i + 1L`.

---

### Trap 7: `5 % 0` vs `5.0 % 0.0`
Integer modulo by zero throws `ArithmeticException`. Floating-point modulo by zero produces `NaN` — no exception. The exam pairs these to test the distinction.

---

### Trap 8: `Integer.MIN_VALUE / -1` overflows
`-2147483648 / -1` should be `2147483648`, but that's too large for `int`. The result wraps back to `-2147483648`. This is the only integer division that produces overflow — the exam may present this.

---

### Trap 9: `null` in concatenation vs method call
`"Hello " + null` → `"Hello null"` — safe, no NPE.
`null.toString()` → `NullPointerException` — unsafe.
`null + null` → `"nullnull"` — surprising but valid.
The `+` operator handles null gracefully; method calls do not.

---

### Trap 10: Multiplication overflow with `int`
`1_000_000 * 1_000_000 = 10^12` which overflows `int`. Result is a seemingly random negative number. The exam presents large multiplications and asks for output — the answer is always the overflowed value, not an exception. Use `long` literals (`1_000_000L`) to prevent this.

---

## 5. Summary Sheet

### Arithmetic Operator Behavior

| Operator | Integer Behavior | Float/Double Behavior |
|---|---|---|
| `+` | Addition, overflow wraps | Addition, may produce Infinity |
| `-` | Subtraction, overflow wraps | Subtraction |
| `*` | Multiplication, overflow wraps | Multiplication, may produce Infinity |
| `/` | Truncates toward zero; `÷0` throws | Returns decimal; `÷0.0` = Infinity |
| `%` | Remainder; `%0` throws; sign=dividend | Remainder; `%0.0` = NaN; sign=dividend |

---

### Type Promotion Summary

| Operation | Result Type |
|---|---|
| `byte` OP `byte` | `int` ⚠ |
| `short` OP `short` | `int` ⚠ |
| `char` OP `char` | `int` ⚠ |
| `int` OP `int` | `int` |
| `int` OP `long` | `long` |
| `long` OP `float` | `float` |
| any OP `double` | `double` |

---

### Division — Key Behaviors

| Expression | Result | Why |
|---|---|---|
| `5 / 2` | `2` | Integer division truncates |
| `-5 / 2` | `-2` | Truncates toward zero |
| `5 / 2.0` | `2.5` | Double operand → float division |
| `double d = 5/2` | `2.0` | Int division first, then widen |
| `(double)5 / 2` | `2.5` | Cast before division |
| `(double)(5/2)` | `2.0` | Cast after integer division |
| `5 / 0` | `ArithmeticException` | Integer ÷ zero |
| `5.0 / 0` | `Infinity` | Float ÷ zero |
| `0.0 / 0.0` | `NaN` | Float zero ÷ float zero |

---

### Modulo — Sign Rules

| Expression | Result | Rule |
|---|---|---|
| `7 % 3` | `1` | Positive dividend → positive |
| `-7 % 3` | `-1` | Negative dividend → negative |
| `7 % -3` | `1` | Positive dividend → positive |
| `-7 % -3` | `-1` | Negative dividend → negative |
| `5.0 % 0.0` | `NaN` | Float mod zero |
| `5 % 0` | `ArithmeticException` | Integer mod zero |

---

### String Concatenation Rules

| Rule | Detail |
|---|---|
| Left-to-right | Evaluate left first; first String triggers all subsequent `+` as concat |
| `char + char` | Numeric addition (int result) |
| `char + String` | Character concatenation |
| `null + String` | Produces `"null"` string — no NPE |
| `obj + String` | Calls `obj.toString()` |
| Parentheses | Force arithmetic before concatenation |

---

### Key Compile/Runtime Rules

| Statement | Error Type |
|---|---|
| `byte b = b1 + b2` | Compile error — int result |
| `b += 1` | OK — implicit cast |
| `5 / 0` | Runtime — ArithmeticException |
| `5.0 / 0` | OK — returns Infinity |
| `(long)i + 1` vs `i + 1L` | Both safe — widen before addition |
| `i + 1` (int overflow) | Silent wrap — no exception |

---
