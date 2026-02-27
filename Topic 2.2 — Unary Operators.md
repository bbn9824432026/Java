# Topic 2.2 — Unary Operators
## (`++`, `--`, `~`, `!`, `-`, `+`)

---

## 1. Conceptual Explanation

### What Are Unary Operators?

Unary operators operate on a **single operand**. They are some of the most frequently tested operators on the OCP exam because their behavior — especially with increment/decrement — produces counterintuitive results that Oracle deliberately exploits in tricky code snippets.

Java's unary operators:

| Operator | Name | Applied To |
|---|---|---|
| `+` | Unary plus | Numeric types |
| `-` | Unary minus (negation) | Numeric types |
| `++` | Increment | Numeric types, `char` |
| `--` | Decrement | Numeric types, `char` |
| `!` | Logical NOT | `boolean` only |
| `~` | Bitwise complement | Integral types only |

All unary operators have **right-to-left associativity** and sit at **level 2** in the precedence table (just below postfix operators).

---

### Unary `+` and `-`

#### Unary `+`
Rarely used. It does not change the value but it **promotes** the operand to at least `int`:

```java
byte b = 10;
byte result = +b;   // COMPILE ERROR — +b promotes to int
int result2 = +b;   // OK — int receives promoted value
```

> **Exam trap:** `+b` on a `byte` yields an `int`, not a `byte`. This surprises candidates who assume unary `+` is a no-op.

#### Unary `-`
Negates the value — arithmetically flips the sign:

```java
int x = 5;
int y = -x;    // y = -5
int z = -(-x); // z = 5

double d = 3.14;
double neg = -d;  // -3.14
```

Unary `-` also promotes to at least `int`:

```java
byte b = 10;
byte neg = -b;   // COMPILE ERROR — -b is int
int neg2 = -b;   // OK
```

**Negating `Integer.MIN_VALUE`:**

```java
int min = Integer.MIN_VALUE;    // -2147483648
int negMin = -min;              // still -2147483648 — overflow!
System.out.println(negMin == min);  // true — same value due to overflow
```

This is because `-Integer.MIN_VALUE` overflows back to `Integer.MIN_VALUE` in two's complement arithmetic. There is no positive representation for `2147483648` in a 32-bit signed integer.

---

### Increment `++` and Decrement `--`

These are the most heavily tested unary operators. Each has two forms.

#### Prefix Form (`++x`, `--x`)
1. **First** modifies the variable
2. **Then** returns the new (modified) value

```java
int x = 5;
int y = ++x;   // x becomes 6, y gets 6
// x = 6, y = 6
```

#### Postfix Form (`x++`, `x--`)
1. **First** returns the current value (for use in the expression)
2. **Then** modifies the variable

```java
int x = 5;
int y = x++;   // y gets 5 (current value), then x becomes 6
// x = 6, y = 5
```

> **Fundamental rule:** The difference is WHEN the side effect (the actual increment/decrement) is visible in the surrounding expression.

---

### The "Temp Value" Mental Model

When the JVM processes `x++`, it conceptually does this:
1. Save the current value of `x` into a temporary slot
2. Increment `x`
3. Return the value from the temporary slot (the OLD value)

When the JVM processes `++x`:
1. Increment `x`
2. Return the current value of `x` (the NEW value)

This mental model helps with complex expressions:

```java
int x = 5;
int result = x++ + x++;
// Step 1: first x++ → temp1 = 5, x becomes 6
// Step 2: second x++ → temp2 = 6, x becomes 7
// Step 3: 5 + 6 = 11
System.out.println(result);  // 11
System.out.println(x);       // 7
```

---

### Increment/Decrement on Non-int Types

`++` and `--` can be applied to all numeric primitives AND `char`:

```java
char c = 'A';
c++;
System.out.println(c);  // B

char d = 'Z';
d++;
System.out.println(d);  // [ (next Unicode character after Z)

byte b = 127;
b++;
System.out.println(b);  // -128 — wraps around (implicit cast)

short s = 32767;
s++;
System.out.println(s);  // -32768 — wraps around
```

> **Key point:** `++` and `--` include an implicit narrowing cast back to the original type. So `byte b = 127; b++;` is legal and wraps to `-128`.

---

### Increment/Decrement CANNOT Be Applied To:

```java
int x = 5;
++(x + 1);    // COMPILE ERROR — cannot increment expression result
++5;          // COMPILE ERROR — cannot increment literal
final int F = 5;
F++;          // COMPILE ERROR — cannot modify final variable
```

The operand of `++`/`--` must be a **variable** (an lvalue) — not an expression, literal, or final variable.

---

### The `!` Logical NOT Operator

Inverts a `boolean` value:

```java
boolean a = true;
boolean b = !a;       // false
boolean c = !false;   // true
boolean d = !!true;   // true — double negation
```

`!` works **only** on `boolean` — not on integers:

```java
int x = 0;
boolean b = !x;   // COMPILE ERROR — ! requires boolean
```

> **Exam trap:** Java is not C. `!0` is not `true`. `!x` on an int is always a compile error.

---

### Multiple `!` Applications

```java
boolean flag = true;
System.out.println(!flag);    // false
System.out.println(!!flag);   // true
System.out.println(!!!flag);  // false
```

Each `!` flips the value. An even number of `!` operators returns the original; odd number flips it.

---

### The `~` Bitwise Complement Operator

`~` flips every bit in an integer. It works on `byte`, `short`, `int`, `long`, and `char` — **NOT on `boolean` or floating-point types**.

The mathematical result of `~x` is always `-(x + 1)`:

```java
System.out.println(~5);    // -6  → -(5+1)
System.out.println(~0);    // -1  → -(0+1)
System.out.println(~-1);   //  0  → -(-1+1)
System.out.println(~127);  // -128 → -(127+1)
```

**Why `-(x+1)`?** In two's complement representation:
- `~5` flips all bits of `5` (binary: `00000101`) to get `11111010`
- In two's complement, `11111010` = `-6`
- Formula: `~x = -(x + 1)` always holds for any integer `x`

```java
int x = 5;
System.out.println(~x);     // -6
System.out.println(~x + 1); // -5 — this is how two's complement negation works
// In fact: -x == ~x + 1 for all integers
```

`~` on smaller types promotes to `int` first:

```java
byte b = 5;
byte result = ~b;   // COMPILE ERROR — ~b is int
int result2 = ~b;   // OK — -6
```

---

### `~` vs `!` — Common Confusion

| Operator | Works On | Effect |
|---|---|---|
| `!` | `boolean` only | Logical NOT — flips true/false |
| `~` | Integral types only | Bitwise NOT — flips all bits |

```java
boolean flag = true;
System.out.println(!flag);   // false ✅
// System.out.println(~flag); // COMPILE ERROR — ~ not for boolean

int x = 5;
System.out.println(~x);      // -6 ✅
// System.out.println(!x);   // COMPILE ERROR — ! not for int
```

---

### Chaining Unary Operators

Unary operators have right-to-left associativity, meaning you can chain them:

```java
int x = 5;
int y = - -x;     // -(-5) = 5 — note the space to avoid confusion with --
int z = --x;      // prefix decrement: x becomes 4, z = 4

// Don't confuse:
int a = 5;
int b = - -a;     // unary minus applied twice: -(-5) = 5
int c = --a;      // pre-decrement: a-1 = 4
```

> **Exam trap:** `- -x` (with a space) is NOT the same as `--x`. `- -x` applies unary minus twice; `--x` is the prefix decrement operator.

---

### Increment in Complex Expressions — Full Trace Method

When the exam presents an expression like:
```java
int a = 3, b = 5;
int c = ++a + b-- - --b + a++;
```

Use this systematic approach:

**Step 1:** Identify all operators left to right, noting prefix vs postfix.
**Step 2:** Process the expression left to right, applying prefix operators immediately and deferring postfix side effects.
**Step 3:** Track variable values after each operation.

```java
int a = 3, b = 5;
int c = ++a + b-- - --b + a++;
```

Trace:
- `++a` → a becomes **4**, use **4**
- `b--` → use **5**, b becomes **4** (deferred)
- `--b` → b (now 4) becomes **3**, use **3**
- `a++` → use **4** (current a), a becomes **5** (deferred)
- Expression: `4 + 5 - 3 + 4 = 10`
- Final: c=10, a=5, b=3

---

### The `i = i++` Trap

This is one of the most notorious Java traps:

```java
int i = 5;
i = i++;
System.out.println(i);   // 5, NOT 6!
```

Why? Because:
1. `i++` — saves the current value (5) as a temp, increments `i` to 6
2. `i = temp` — assigns the saved temp value (5) back to `i`
3. The increment to 6 is immediately overwritten by the assignment

This is a well-known Java gotcha. The right-hand side is fully evaluated before the assignment on the left — and `i++` returns 5 (the old value) while setting `i` to 6, but then the assignment `i = 5` overwrites `i` back to 5.

Similarly:
```java
int i = 5;
i = ++i;
System.out.println(i);   // 6 — prefix version works as expected
// ++i: i becomes 6, returns 6. i = 6 (no surprise)
```

---

## 2. Code Examples

### Example 1 — Prefix vs Postfix Fundamentals

```java
public class PrefixPostfix {
    public static void main(String[] args) {
        int a = 10;

        // Postfix — use then modify
        System.out.println(a++);  // 10
        System.out.println(a);    // 11

        // Prefix — modify then use
        System.out.println(++a);  // 12
        System.out.println(a);    // 12

        // In assignment
        int b = 5;
        int c = b++;   // c=5, b=6
        int d = ++b;   // b=7, d=7
        System.out.println(c + " " + d + " " + b);  // 5 7 7
    }
}
```

---

### Example 2 — `i = i++` Classic Trap

```java
public class IEqualIPlusPlus {
    public static void main(String[] args) {
        int i = 5;
        i = i++;         // i stays 5!
        System.out.println(i);  // 5

        int j = 5;
        j = ++j;         // j becomes 6 (prefix works as expected)
        System.out.println(j);  // 6

        int k = 5;
        k++;             // no assignment — k correctly becomes 6
        System.out.println(k);  // 6
    }
}
```

---

### Example 3 — Increment in Loop

```java
public class LoopIncrement {
    public static void main(String[] args) {
        // Postfix in for loop — standard usage
        for (int i = 0; i < 5; i++) {
            System.out.print(i + " ");  // 0 1 2 3 4
        }
        System.out.println();

        // Prefix in for loop — identical result here
        for (int i = 0; i < 5; ++i) {
            System.out.print(i + " ");  // 0 1 2 3 4 — same output
        }
        System.out.println();
        // Difference only matters when the expression's VALUE is used
    }
}
```

---

### Example 4 — `~` Bitwise Complement

```java
public class BitwiseComplement {
    public static void main(String[] args) {
        System.out.println(~0);     // -1
        System.out.println(~1);     // -2
        System.out.println(~5);     // -6
        System.out.println(~-1);    //  0
        System.out.println(~-6);    //  5

        // Verify formula: ~x = -(x+1)
        int x = 42;
        System.out.println(~x == -(x + 1));   // true
        System.out.println(~x == -x - 1);     // true (equivalent)

        // On long
        long l = 100L;
        System.out.println(~l);  // -101
    }
}
```

---

### Example 5 — `!` on Boolean Only

```java
public class LogicalNot {
    public static void main(String[] args) {
        boolean flag = true;
        System.out.println(!flag);     // false
        System.out.println(!!flag);    // true
        System.out.println(!!!flag);   // false

        // In conditions
        int x = 5;
        boolean isPositive = x > 0;
        if (!isPositive) {
            System.out.println("not positive");
        } else {
            System.out.println("positive");  // this prints
        }

        // ! on int — compile error:
        // System.out.println(!x);  // COMPILE ERROR
    }
}
```

---

### Example 6 — Unary Minus Promotion

```java
public class UnaryMinusPromotion {
    public static void main(String[] args) {
        byte b = 10;
        // byte neg = -b;   // COMPILE ERROR — -b is int
        int neg = -b;        // OK

        // But compound assignment includes cast:
        b = (byte) -b;       // explicit cast needed
        System.out.println(b);  // -10

        // Negating MIN_VALUE overflows:
        int min = Integer.MIN_VALUE;
        System.out.println(-min);           // -2147483648 — same as min!
        System.out.println(-min == min);    // true — overflow
    }
}
```

---

### Example 7 — Char Increment

```java
public class CharIncrement {
    public static void main(String[] args) {
        char c = 'A';
        System.out.println(c++);  // A (then c becomes 'B')
        System.out.println(c);    // B
        System.out.println(++c);  // C

        // Numeric assignment from char arithmetic
        char x = 'a';
        // char y = x + 1;   // COMPILE ERROR — x+1 is int
        char y = (char)(x + 1);   // explicit cast needed
        char z = x;
        z++;                      // compound — OK, no cast needed
        System.out.println(y);    // b
        System.out.println(z);    // b
    }
}
```

---

### Example 8 — Complex Increment Expression

```java
public class ComplexIncrement {
    public static void main(String[] args) {
        int a = 5, b = 3;

        // Trace carefully:
        int c = a++ + ++b + a;
        // a++ → use 5, a becomes 6
        // ++b → b becomes 4, use 4
        // a   → use 6 (a is now 6 from a++)
        // 5 + 4 + 6 = 15
        System.out.println(c);   // 15
        System.out.println(a);   // 6
        System.out.println(b);   // 4

        // Another tricky one:
        int x = 10;
        int y = x-- - --x;
        // x-- → use 10, x becomes 9
        // --x → x (now 9) becomes 8, use 8
        // 10 - 8 = 2
        System.out.println(y);   // 2
        System.out.println(x);   // 8
    }
}
```

---

### Example 9 — `- -x` vs `--x`

```java
public class MinusMinusVsDecrement {
    public static void main(String[] args) {
        int x = 5;
        int a = - -x;   // unary minus twice: -(-5) = 5. x unchanged.
        System.out.println(a);   // 5
        System.out.println(x);   // 5 — x NOT modified

        int y = 5;
        int b = --y;    // prefix decrement: y becomes 4, b = 4
        System.out.println(b);   // 4
        System.out.println(y);   // 4 — y IS modified
    }
}
```

---

## 3. Exam-Style Questions

---

**Q1 — MCQ**
What is the output?
```java
int a = 5;
int b = a++;
int c = ++a;
System.out.println(a + " " + b + " " + c);
```
- A. `7 5 7`
- B. `7 6 7`
- C. `6 5 6`
- D. `7 5 6`

**Answer: A**
`b = a++` → b=5, a=6. `c = ++a` → a=7, c=7. Output: `7 5 7`.

---

**Q2 — MCQ**
What is the output?
```java
int i = 5;
i = i++;
System.out.println(i);
```
- A. 4
- B. 5
- C. 6
- D. Compile error

**Answer: B**
`i++` returns 5 (old value) and increments `i` to 6. Then `i = 5` (the returned value) overwrites back to 5. Classic `i = i++` trap.

---

**Q3 — MCQ**
What is the output?
```java
System.out.println(~5);
System.out.println(~0);
System.out.println(~-1);
```
- A. `-5` / `-1` / `1`
- B. `-6` / `-1` / `0`
- C. `-6` / `1` / `0`
- D. `5` / `0` / `1`

**Answer: B**
`~5 = -(5+1) = -6`. `~0 = -(0+1) = -1`. `~-1 = -(-1+1) = 0`.

---

**Q4 — Select All That Apply**
Which of the following cause compile errors? (Choose all that apply)

```java
A. byte b = 5; b++;
B. byte b = 5; b = -b;
C. boolean x = true; x = !x;
D. int x = 5; boolean b = !x;
E. int x = 5; int y = ~x;
F. boolean b = true; int x = ~b;
```

**Answer: B, D, F**
- A: `b++` has implicit cast — OK
- B: `-b` promotes to int, cannot assign to byte without cast — **ERROR**
- C: `!x` on boolean — OK
- D: `!x` on int — **ERROR**
- E: `~x` on int — OK, y = -6
- F: `~b` on boolean — **ERROR**, `~` not for boolean

---

**Q5 — MCQ**
What is the output?
```java
int x = 3, y = 4;
int z = x++ * ++y;
System.out.println(x + " " + y + " " + z);
```
- A. `3 5 15`
- B. `4 5 15`
- C. `4 4 12`
- D. `3 4 12`

**Answer: B**
`x++` uses 3, x→4. `++y` y→5, uses 5. `z = 3 * 5 = 15`. Output: `4 5 15`.

---

**Q6 — MCQ**
What is the output?
```java
int a = 10;
int b = - -a;
System.out.println(a + " " + b);
```
- A. `9 9`
- B. `10 10`
- C. `9 10`
- D. `10 -10`

**Answer: B**
`- -a` is unary minus applied twice: `-(-10) = 10`. `a` is NOT modified (no `--` operator). a=10, b=10.

---

**Q7 — MCQ**
```java
byte b = 127;
b++;
System.out.println(b);
```
- A. 128
- B. -128
- C. Compile error
- D. Runtime exception

**Answer: B**
`b++` includes implicit cast: `(byte)(127 + 1) = (byte)(128) = -128`. Wraps around.

---

**Q8 — Select All That Apply**
Which are valid uses of `++` or `--`? (Choose all that apply)

- A. `int x = 5; x++;`
- B. `int x = 5; (x + 1)++;`
- C. `char c = 'A'; c++;`
- D. `final int x = 5; x++;`
- E. `double d = 3.14; d++;`
- F. `int x = 5; ++x;`

**Answer: A, C, E, F**
- B: Cannot increment an expression result — **ERROR**
- D: Cannot modify `final` — **ERROR**
- A, C, E, F: All valid. `++`/`--` work on all numeric types including `double` and `char`.

---

**Q9 — MCQ**
What is the output?
```java
int n = 5;
int result = n-- - --n;
System.out.println(result + " " + n);
```
- A. `2 3`
- B. `1 3`
- C. `2 4`
- D. `0 3`

**Answer: A**
`n--` uses 5, n→4. `--n` n(now 4)→3, uses 3. `5 - 3 = 2`. n=3. Output: `2 3`.

---

**Q10 — MCQ**
What is the output?
```java
boolean a = true, b = false;
System.out.println(!a || b);
System.out.println(!(a || b));
System.out.println(!a || !b);
```
- A. `false` / `false` / `true`
- B. `false` / `true` / `true`
- C. `true` / `false` / `false`
- D. `false` / `false` / `false`

**Answer: A**
- `!a || b` → `false || false = false`
- `!(a || b)` → `!(true) = false`
- `!a || !b` → `false || true = true`

---

## 4. Trick Analysis

### Trap 1: `i = i++` returns original value
The most famous Java operator trap. `i = i++` does NOT increment `i`. The postfix `i++` returns the OLD value, increments `i`, then the assignment overwrites `i` with the OLD value. Net result: `i` is unchanged. Use `i++` as a standalone statement when you just want to increment.

---

### Trap 2: Unary minus and plus promote to `int`
`byte b = 10; byte neg = -b;` is a **compile error** because `-b` produces an `int`. The same applies to `+b`. Candidates assume unary operators preserve the type. They don't for `byte`, `short`, and `char`.

---

### Trap 3: `~` formula is `-(x+1)`, not `-x`
`~5` is `-6`, NOT `-5`. Candidates confuse bitwise complement with arithmetic negation. `-5` would be `~5 + 1` (that's how two's complement negation works). `~` just flips bits, which mathematically equals `-(x+1)`.

---

### Trap 4: `~` does not work on `boolean`, `!` does not work on `int`
These are completely separate operators for completely separate types. `~true` is a compile error. `!5` is a compile error. The exam presents mixed usage to test this.

---

### Trap 5: `++` includes implicit cast for `byte`/`char`
`byte b = 127; b++;` compiles and wraps to `-128`. No compile error, no runtime exception. But `byte b = 127; b = b + 1;` IS a compile error. The compound nature of `++` includes the cast; explicit addition does not.

---

### Trap 6: `- -x` vs `--x`
With a space: `- -x` is unary minus applied twice — `x` is NOT modified. Without a space: `--x` is the prefix decrement — `x` IS modified. The exam uses this spacing trick in code snippets.

---

### Trap 7: `++` cannot be applied to expressions or finals
`++(a + b)` is always a compile error — you can only increment a variable, not an expression result. `final int x = 5; x++;` is a compile error — `final` variables cannot be modified.

---

### Trap 8: Postfix in loops — prefix/postfix same result when standalone
`i++` and `++i` as standalone statements (not used in a larger expression) produce identical results. The difference ONLY matters when the expression's VALUE is used somewhere. In `for (int i = 0; i < n; i++)` vs `for (int i = 0; i < n; ++i)` — identical behavior.

---

### Trap 9: Negating `Integer.MIN_VALUE` overflows
`-Integer.MIN_VALUE == Integer.MIN_VALUE`. There is no positive counterpart for the minimum int value in 32-bit two's complement. The exam may show this equality and ask what it prints — it prints `true`.

---

### Trap 10: Multiple `!` applications
`!!!true` = `false` (odd number of `!` flips). `!!!!true` = `true` (even number). The exam stacks multiple `!` operators to test whether candidates can mentally track the flips.

---

## 5. Summary Sheet

### Unary Operators Reference

| Operator | Name | Works On | Effect | Promotes to int? |
|---|---|---|---|---|
| `+x` | Unary plus | Numeric | No change | ✅ (byte/short/char) |
| `-x` | Unary minus | Numeric | Negates value | ✅ (byte/short/char) |
| `++x` | Prefix incr. | Numeric, char | Increment FIRST, return new | Implicit cast back |
| `x++` | Postfix incr. | Numeric, char | Return current, increment AFTER | Implicit cast back |
| `--x` | Prefix decr. | Numeric, char | Decrement FIRST, return new | Implicit cast back |
| `x--` | Postfix decr. | Numeric, char | Return current, decrement AFTER | Implicit cast back |
| `!x` | Logical NOT | `boolean` ONLY | Flips true/false | ❌ |
| `~x` | Bitwise NOT | Integral ONLY | Flips all bits | ✅ |

---

### Prefix vs Postfix Decision Table

| Expression | Value Returned | Side Effect |
|---|---|---|
| `++a` | New value (a+1) | a is incremented before use |
| `a++` | Old value (original a) | a is incremented after use |
| `--a` | New value (a-1) | a is decremented before use |
| `a--` | Old value (original a) | a is decremented after use |

---

### Key Rules to Memorize

| Rule | Detail |
|---|---|
| `i = i++` | i is UNCHANGED — old value overwrites the increment |
| `~x` formula | Always equals `-(x+1)` |
| `~` vs `!` | `~` for integers, `!` for booleans — NOT interchangeable |
| Unary `-`/`+` on byte | Produces int — cannot assign back to byte without cast |
| `++`/`--` on byte/char | Includes implicit cast — wraps without error |
| `++`/`--` on final | COMPILE ERROR |
| `++`/`--` on expression | COMPILE ERROR — needs a variable (lvalue) |
| `- -x` (with space) | Unary minus twice — x unchanged |
| `--x` (no space) | Prefix decrement — x modified |
| `-Integer.MIN_VALUE` | Still `Integer.MIN_VALUE` — overflow |

---

### `~` Quick Reference

| x | ~x | Formula check |
|---|---|---|
| 0 | -1 | -(0+1) = -1 ✅ |
| 1 | -2 | -(1+1) = -2 ✅ |
| 5 | -6 | -(5+1) = -6 ✅ |
| -1 | 0 | -(-1+1) = 0 ✅ |
| -5 | 4 | -(-5+1) = 4 ✅ |
| 127 | -128 | -(127+1) = -128 ✅ |

---
