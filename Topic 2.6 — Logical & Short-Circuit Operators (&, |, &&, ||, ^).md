# Topic 2.6 — Logical & Short-Circuit Operators
## (`&`, `|`, `&&`, `||`, `^`)

---

## 1. Conceptual Explanation

### Overview

Java has two distinct categories of logical operators, and confusing them is one of the most common OCP exam mistakes:

| Category | Operators | Evaluates Both Sides? |
|---|---|---|
| **Short-circuit** | `&&`, `\|\|` | Only if necessary |
| **Non-short-circuit** | `&`, `\|`, `^` | Always |

Both categories produce the same **logical result** when applied to booleans. The difference is purely about **side effects** — whether the right-hand side expression is evaluated at all.

This distinction is tested relentlessly because it affects:
- Whether method calls on the right side execute
- Whether exceptions on the right side are thrown
- Whether variables on the right side are modified
- Program correctness in null-safety patterns

---

### Short-Circuit AND: `&&`

`&&` evaluates the left operand first. If the left operand is **`false`**, the entire expression is already `false` regardless of the right operand — so the right side is **skipped entirely**.

```
false && anything = false   (right side never evaluated)
true  && anything = anything (right side evaluated)
```

```java
int x = 5;
boolean result = (x > 10) && (++x > 0);
// x > 10 is false → right side SKIPPED → x is NOT incremented
System.out.println(result);  // false
System.out.println(x);       // 5 — unchanged!
```

---

### Short-Circuit OR: `||`

`||` evaluates the left operand first. If the left operand is **`true`**, the entire expression is already `true` — so the right side is **skipped entirely**.

```
true  || anything = true    (right side never evaluated)
false || anything = anything (right side evaluated)
```

```java
int x = 5;
boolean result = (x < 10) || (++x > 0);
// x < 10 is true → right side SKIPPED → x is NOT incremented
System.out.println(result);  // true
System.out.println(x);       // 5 — unchanged!
```

---

### Non-Short-Circuit AND: `&`

`&` on booleans **always evaluates both sides**, regardless of the left operand's value:

```java
int x = 5;
boolean result = (x > 10) & (++x > 0);
// x > 10 is false — but right side IS still evaluated
// ++x executes → x becomes 6
System.out.println(result);  // false
System.out.println(x);       // 6 — incremented!
```

---

### Non-Short-Circuit OR: `|`

`|` on booleans **always evaluates both sides**, regardless of the left operand's value:

```java
int x = 5;
boolean result = (x < 10) | (++x > 0);
// x < 10 is true — but right side IS still evaluated
// ++x executes → x becomes 6
System.out.println(result);  // true
System.out.println(x);       // 6 — incremented!
```

---

### XOR: `^`

`^` (exclusive OR) — always evaluates both sides. Returns `true` if the operands are **different**, `false` if they are the **same**:

```
true  ^ true  = false   (same → false)
false ^ false = false   (same → false)
true  ^ false = true    (different → true)
false ^ true  = true    (different → true)
```

```java
System.out.println(true  ^ true);   // false
System.out.println(false ^ false);  // false
System.out.println(true  ^ false);  // true
System.out.println(false ^ true);   // true
```

`^` has no short-circuit counterpart — both operands must always be known to determine if they're different.

---

### Truth Tables — All Five Operators

| A | B | `A && B` | `A \|\| B` | `A & B` | `A \| B` | `A ^ B` |
|---|---|---|---|---|---|---|
| true | true | true | true | true | true | false |
| true | false | false | true | false | true | true |
| false | true | false | true | false | true | true |
| false | false | false | false | false | false | false |

For booleans, `&&` and `&` produce identical logical results. Same for `||` and `|`. The ONLY difference is side effects via short-circuiting.

---

### Short-Circuit in Null Safety — Classic Pattern

The most practical use of short-circuit evaluation:

```java
String s = null;

// Without short-circuit — NPE:
if (s != null & s.length() > 0) { }   // s.length() evaluated even if s is null → NPE

// With short-circuit — safe:
if (s != null && s.length() > 0) { }  // s.length() skipped if s is null → safe
```

```java
// Or pattern — check for value OR provide default:
String result = (s != null) ? s : "default";

// Short-circuit equivalent:
if (s == null || s.isEmpty()) {
    s = "default";
}
// If s is null: s==null is true, right side (s.isEmpty()) skipped — no NPE!
```

---

### `&` and `|` on Integer Types (Bitwise Operations)

When `&`, `|`, and `^` are applied to **integer types** (not booleans), they perform **bitwise** operations — operating on each individual bit:

```java
// Bitwise AND
System.out.println(5 & 3);   // 1
// 5 = 0101
// 3 = 0011
// &   0001 = 1

// Bitwise OR
System.out.println(5 | 3);   // 7
// 5 = 0101
// 3 = 0011
// |   0111 = 7

// Bitwise XOR
System.out.println(5 ^ 3);   // 6
// 5 = 0101
// 3 = 0011
// ^   0110 = 6
```

> **Critical distinction:** `&`, `|`, `^` are **context-sensitive**:
> - On `boolean` operands → logical operations (no short-circuit)
> - On integer operands → bitwise operations

`&&` and `||` are **only for booleans** — using them with integers is a compile error.

---

### Operator Precedence Among Logical Operators

From the full precedence table (Topic 2.1):

```
! (NOT)  — highest among logicals
&        — bitwise/logical AND
^        — bitwise/logical XOR
|        — bitwise/logical OR
&&       — short-circuit AND
||       — short-circuit OR   — lowest
```

This means `&` has **higher precedence** than `&&`, and `|` has **higher precedence** than `||`:

```java
boolean result = true | false && false;
// && has higher precedence than ||... but wait:
// The order here is | vs &&
// | (level 11) comes BEFORE && (level 12) in the table
// So | is evaluated first: true | false = true
// Then: true && false = false
// Hmm — need to clarify the full ordering
```

Let me be precise about the full bitwise/logical precedence:

```
Level 9:  &   (bitwise/logical AND) — highest of the three
Level 10: ^   (bitwise/logical XOR)
Level 11: |   (bitwise/logical OR)
Level 12: &&  (short-circuit AND)
Level 13: ||  (short-circuit OR)  — lowest
```

So the order from highest to lowest precedence is: `&` > `^` > `|` > `&&` > `||`

```java
// Example with mixed operators:
boolean r = false | true && false;
// && has higher precedence than |
// Step 1: true && false = false
// Step 2: false | false = false
System.out.println(r);  // false

boolean r2 = true | false & false;
// & has higher precedence than |
// Step 1: false & false = false
// Step 2: true | false = true
System.out.println(r2);  // true
```

---

### Short-Circuit with Method Calls — Side Effects

The exam frequently places method calls with side effects on the right side of `&&`/`||` to test whether candidates know when the method executes:

```java
static int counter = 0;

static boolean increment() {
    counter++;
    return true;
}

static boolean decrement() {
    counter--;
    return false;
}

public static void main(String[] args) {
    counter = 0;

    // && — right side skipped when left is false
    boolean r1 = false && increment();
    System.out.println(counter);  // 0 — increment() NOT called

    // && — right side evaluated when left is true
    boolean r2 = true && increment();
    System.out.println(counter);  // 1 — increment() called

    // || — right side skipped when left is true
    boolean r3 = true || increment();
    System.out.println(counter);  // 1 — increment() NOT called

    // || — right side evaluated when left is false
    boolean r4 = false || increment();
    System.out.println(counter);  // 2 — increment() called

    // Non-short-circuit — right side ALWAYS evaluated
    boolean r5 = false & increment();
    System.out.println(counter);  // 3 — increment() called despite false left
}
```

---

### Short-Circuit in Loops

```java
// Safe iteration with null check
String[] arr = {"hello", null, "world"};
for (String s : arr) {
    if (s != null && s.length() > 3) {  // safe — if s is null, length() not called
        System.out.println(s);
    }
}
// Prints: hello, world (both length > 3)
```

---

### Compound Assignment with Logical Operators

```java
boolean a = true;
a &= false;    // a = a & false = false
System.out.println(a);  // false

boolean b = false;
b |= true;     // b = b | true = true
System.out.println(b);  // true

boolean c = true;
c ^= true;     // c = c ^ true = false
System.out.println(c);  // false

// Note: no &&= or ||= operators in Java!
// These do NOT exist:
// a &&= false;  // COMPILE ERROR
// b ||= true;   // COMPILE ERROR
```

> **Exam trap:** `&&=` and `||=` do NOT exist in Java. Only `&=`, `|=`, and `^=` are valid compound logical assignment operators. Candidates from other languages sometimes expect `&&=` to exist.

---

### De Morgan's Laws — Logical Equivalences

These transformations appear in exam questions about negating compound conditions:

```
!(A && B)  ≡  (!A || !B)
!(A || B)  ≡  (!A && !B)
```

```java
boolean a = true, b = false;

System.out.println(!(a && b));     // true
System.out.println(!a || !b);      // true — equivalent

System.out.println(!(a || b));     // false
System.out.println(!a && !b);      // false — equivalent
```

The exam presents code using one form and asks what it's equivalent to — knowing De Morgan's laws lets you quickly verify.

---

### `!` with Compound Expressions

```java
boolean x = true, y = false, z = true;

// ! has higher precedence than && and ||
System.out.println(!x && y);       // false — !x=false, false && false=false
System.out.println(!(x && y));     // true  — x&&y=false, !false=true
System.out.println(!x || y || z);  // true  — !x=false, false||false=false, false||true=true
```

---

### Short-Circuit and Exception Prevention

```java
String s = null;
int len = (s == null) ? 0 : s.length();  // ternary version — safe

// Short-circuit version:
int len2 = 0;
if (s != null && (len2 = s.length()) > 0) {
    System.out.println("Non-empty: " + len2);
} else {
    System.out.println("Null or empty, len: " + len2);  // len2 = 0
}

// Classic divide-by-zero prevention:
int divisor = 0;
int result = (divisor != 0) && (10 / divisor > 0) ? 1 : 0;
// divisor != 0 is false → right side skipped → no ArithmeticException
System.out.println(result);  // 0
```

---

## 2. Code Examples

### Example 1 — Short-Circuit vs Non-Short-Circuit Side Effects

```java
public class ShortCircuitDemo {
    static int x = 0;

    static boolean sideEffect(boolean val) {
        x++;
        return val;
    }

    public static void main(String[] args) {
        // && short-circuits on false
        x = 0;
        boolean r1 = sideEffect(false) && sideEffect(true);
        System.out.println(r1 + ", x=" + x);  // false, x=1 — second NOT called

        // & does NOT short-circuit
        x = 0;
        boolean r2 = sideEffect(false) & sideEffect(true);
        System.out.println(r2 + ", x=" + x);  // false, x=2 — both called

        // || short-circuits on true
        x = 0;
        boolean r3 = sideEffect(true) || sideEffect(false);
        System.out.println(r3 + ", x=" + x);  // true, x=1 — second NOT called

        // | does NOT short-circuit
        x = 0;
        boolean r4 = sideEffect(true) | sideEffect(false);
        System.out.println(r4 + ", x=" + x);  // true, x=2 — both called
    }
}
```

---

### Example 2 — Null Safety Pattern

```java
public class NullSafety {
    public static void main(String[] args) {
        String[] words = {"hello", null, "world", null, "java"};

        for (String w : words) {
            // Safe — short-circuit prevents NPE
            if (w != null && w.startsWith("h")) {
                System.out.println(w);  // hello
            }

            // DANGEROUS — & evaluates both sides
            // if (w != null & w.startsWith("h")) { }  // NPE when w is null
        }

        // Checking both conditions:
        String s = "Java";
        if (s != null && !s.isEmpty() && s.length() > 2) {
            System.out.println(s.toUpperCase());  // JAVA
        }
    }
}
```

---

### Example 3 — XOR Applications

```java
public class XorDemo {
    public static void main(String[] args) {
        // Boolean XOR
        boolean a = true, b = false;
        System.out.println(a ^ b);   // true — different
        System.out.println(a ^ a);   // false — same
        System.out.println(b ^ b);   // false — same

        // XOR swap (classic trick)
        int x = 5, y = 10;
        x ^= y;   // x = 5^10 = 15
        y ^= x;   // y = 10^15 = 5
        x ^= y;   // x = 15^5 = 10
        System.out.println(x + " " + y);  // 10 5 — swapped!

        // XOR for toggle
        boolean flag = false;
        flag ^= true;  // toggle: false ^ true = true
        System.out.println(flag);  // true
        flag ^= true;  // toggle: true ^ true = false
        System.out.println(flag);  // false
    }
}
```

---

### Example 4 — Precedence with Mixed Logical Operators

```java
public class LogicalPrecedence {
    public static void main(String[] args) {
        // & higher precedence than |
        System.out.println(true | false & false);
        // & first: false & false = false
        // then |: true | false = true
        // Output: true

        // && higher precedence than ||
        System.out.println(true || false && false);
        // && first: false && false = false
        // then ||: true || false = true
        // Output: true

        // & higher precedence than &&
        System.out.println(false & true && true);
        // & first: false & true = false
        // then &&: false && true = false (short-circuits — but left is already false)
        // Output: false

        // Full mixed example
        boolean r = true | false & false ^ true;
        // & first: false & false = false
        // ^ next: false ^ true = true (wait — need to check precedence: & > ^ > |)
        // Hmm: false & false = false, then false ^ true = true, then true | true = true
        System.out.println(r);  // true
    }
}
```

---

### Example 5 — No `&&=` or `||=` in Java

```java
public class NoShortCircuitAssign {
    public static void main(String[] args) {
        boolean a = true;

        // These work:
        a &= false;    // a = a & false = false
        System.out.println(a);  // false

        a |= true;     // a = a | true = true
        System.out.println(a);  // true

        a ^= false;    // a = a ^ false = true (unchanged)
        System.out.println(a);  // true

        // These do NOT exist:
        // a &&= false;  // COMPILE ERROR
        // a ||= true;   // COMPILE ERROR
    }
}
```

---

### Example 6 — De Morgan's Laws

```java
public class DeMorgan {
    public static void main(String[] args) {
        boolean p = true, q = false;

        // !(p && q) == !p || !q
        System.out.println(!(p && q));    // true
        System.out.println(!p || !q);     // true — equivalent

        // !(p || q) == !p && !q
        System.out.println(!(p || q));    // false
        System.out.println(!p && !q);     // false — equivalent

        // Practical usage — inverting a filter:
        // Original: process if (isActive && isValid)
        // Skip if: !(isActive && isValid) == !isActive || !isValid
        boolean isActive = true, isValid = false;
        if (!isActive || !isValid) {
            System.out.println("Skip processing");  // prints
        }
    }
}
```

---

### Example 7 — Bitwise vs Logical on Different Types

```java
public class BitwiseVsLogical {
    public static void main(String[] args) {
        // On boolean — logical (no short-circuit)
        boolean a = true, b = false;
        System.out.println(a & b);    // false — logical AND
        System.out.println(a | b);    // true  — logical OR
        System.out.println(a ^ b);    // true  — logical XOR

        // On integers — bitwise
        int x = 12, y = 10;  // 1100 & 1010
        System.out.println(x & y);   // 8  — 1000
        System.out.println(x | y);   // 14 — 1110
        System.out.println(x ^ y);   // 6  — 0110

        // && and || on integers — COMPILE ERROR:
        // System.out.println(x && y);  // COMPILE ERROR
        // System.out.println(x || y);  // COMPILE ERROR
    }
}
```

---

### Example 8 — Complex Short-Circuit Chain

```java
public class ComplexShortCircuit {
    static int count = 0;

    static boolean check(boolean val, String label) {
        count++;
        System.out.println("Evaluated: " + label + " = " + val);
        return val;
    }

    public static void main(String[] args) {
        count = 0;
        boolean result = check(true, "A")
                      || check(false, "B")
                      && check(true, "C");

        // Precedence: && before ||
        // So parses as: check(A) || (check(B) && check(C))
        // check(A) = true → || short-circuits → B and C NOT evaluated!
        System.out.println("Result: " + result);  // true
        System.out.println("Count: " + count);    // 1 — only A evaluated!
    }
}
```

---

## 3. Exam-Style Questions

---

**Q1 — MCQ**
What is the output?
```java
int x = 5;
boolean r = (x > 10) && (++x > 0);
System.out.println(r + " " + x);
```
- A. `false 6`
- B. `false 5`
- C. `true 6`
- D. `true 5`

**Answer: B**
`x > 10` is `false` → `&&` short-circuits → `++x` never executes. `r = false`, `x = 5`.

---

**Q2 — MCQ**
What is the output?
```java
int x = 5;
boolean r = (x > 10) & (++x > 0);
System.out.println(r + " " + x);
```
- A. `false 6`
- B. `false 5`
- C. `true 6`
- D. `true 5`

**Answer: A**
`&` does NOT short-circuit. Both sides evaluated. `x > 10 = false`, `++x > 0 = 6 > 0 = true`. `false & true = false`. x = 6.

---

**Q3 — MCQ**
What is the output?
```java
int x = 5;
boolean r = (x < 10) || (++x > 0);
System.out.println(r + " " + x);
```
- A. `true 6`
- B. `true 5`
- C. `false 5`
- D. `false 6`

**Answer: B**
`x < 10` is `true` → `||` short-circuits → `++x` never executes. `r = true`, `x = 5`.

---

**Q4 — Select All That Apply**
Which statements are true about `&&` and `&` when used with booleans?

- A. `&&` and `&` always produce the same boolean result
- B. `&&` may skip evaluating the right operand
- C. `&` always evaluates both operands
- D. `&&` is more efficient than `&` in all cases
- E. `&` can be applied to integers but `&&` cannot
- F. `&&=` is a valid Java operator

**Answer: A, B, C, E**
D is false — `&&` is more efficient when short-circuiting, but identical otherwise. F is false — `&&=` does not exist. A, B, C, E are all true.

---

**Q5 — MCQ**
```java
String s = null;
boolean r = (s != null) && (s.length() > 0);
System.out.println(r);
```
- A. Compile error
- B. NullPointerException
- C. `false`
- D. `true`

**Answer: C**
`s != null` is `false` → `&&` short-circuits → `s.length()` never called → no NPE. Result: `false`.

---

**Q6 — MCQ**
```java
String s = null;
boolean r = (s != null) & (s.length() > 0);
System.out.println(r);
```
- A. `false`
- B. `true`
- C. NullPointerException
- D. Compile error

**Answer: C**
`&` does NOT short-circuit. Both sides evaluated. `s.length()` called on `null` → `NullPointerException`.

---

**Q7 — MCQ**
What is the output?
```java
System.out.println(true ^ true);
System.out.println(true ^ false);
System.out.println(false ^ false);
```
- A. `true` / `true` / `false`
- B. `false` / `true` / `false`
- C. `true` / `false` / `true`
- D. `false` / `false` / `true`

**Answer: B**
XOR: same→false, different→true. `T^T=false`, `T^F=true`, `F^F=false`.

---

**Q8 — MCQ**
```java
boolean a = true;
a &&= false;
System.out.println(a);
```
- A. `true`
- B. `false`
- C. Compile error
- D. Runtime error

**Answer: C**
`&&=` is NOT a valid Java operator. Compile error.

---

**Q9 — MCQ**
What is the output?
```java
int count = 0;
boolean r = (count++ > 0) || (count++ > 0) || (count++ > 0);
System.out.println(r + " " + count);
```
- A. `false 3`
- B. `true 2`
- C. `false 1`
- D. `true 1`

**Answer: B**
First `count++`: uses 0, count→1. `0 > 0 = false`. Second `count++`: uses 1, count→2. `1 > 0 = true`. `||` short-circuits → third NOT evaluated. `false || true = true`. count = 2. Output: `true 2`.

---

**Q10 — MCQ**
Which of the following correctly applies De Morgan's law to `!(a && b)`?

- A. `!a && !b`
- B. `!a || !b`
- C. `a || b`
- D. `a && b`

**Answer: B**
De Morgan: `!(A && B) = !A || !B`.

---

**Q11 — Select All That Apply**
Which of the following are valid compound logical assignment operators in Java?

- A. `&=`
- B. `|=`
- C. `^=`
- D. `&&=`
- E. `||=`
- F. `!=`

**Answer: A, B, C**
`&&=` and `||=` do not exist. `!=` is a comparison operator, not a compound assignment. `&=`, `|=`, `^=` are all valid.

---

**Q12 — MCQ**
```java
boolean a = true;
boolean b = false;
System.out.println(a | b & !a);
```
- A. `true`
- B. `false`
- C. Compile error
- D. `true` then `false`

**Answer: A**
Precedence: `!` first, then `&`, then `|`.
- `!a = false`
- `b & false = false & false = false`
- `a | false = true | false = true`

---

## 4. Trick Analysis

### Trap 1: `&&` vs `&` — same result but different side effects
The logical result is identical for booleans. The ONLY difference is whether the right side is evaluated. The exam places a method call, `++`, `--`, or division on the right side and asks about the value of a variable or whether an exception is thrown. Always ask: which operator is used? Does it short-circuit?

---

### Trap 2: `&` on null reference → NPE
`(s != null) & (s.length() > 0)` — `&` evaluates both sides, so `s.length()` runs even when `s` is `null`. The exam deliberately uses `&` where `&&` would be safe to test whether candidates know `&` doesn't short-circuit.

---

### Trap 3: `&&=` and `||=` don't exist
Java has `&=`, `|=`, and `^=` but NOT `&&=` or `||=`. Other languages (C, JavaScript) have similar constructs, making this a cross-language confusion trap. The exam presents `a &&= false` and asks what it does — it's a compile error.

---

### Trap 4: `&&` and `||` precedence relative to `&`, `|`, `^`
`&` has HIGHER precedence than `&&`. `|` has HIGHER precedence than `||`. So `a | b && c` parses as `a | (b && c)`, NOT `(a | b) && c`. The exam writes mixed expressions relying on candidates reading left-to-right.

---

### Trap 5: Short-circuit chain with precedence
```java
true || false && false
```
Because `&&` has higher precedence than `||`, this is `true || (false && false)` = `true || false` = `true`. A candidate reading left-to-right would compute `(true || false) && false` = `true && false` = `false`. Completely wrong.

---

### Trap 6: `++x` or `x++` in short-circuited expression
When the right side is skipped due to short-circuiting, ALL expressions on the right side are skipped — including increment/decrement operations. The variable stays at its pre-expression value.

---

### Trap 7: XOR — "different means true"
`^` returns `true` when operands are **different**, `false` when **same**. Candidates often confuse this with OR. `true ^ true = false` (same). `true ^ false = true` (different). The exam uses XOR in longer boolean expressions.

---

### Trap 8: `|` and `&` on integers vs booleans
`&`, `|`, `^` are overloaded operators — they work as logical operators on `boolean` and as bitwise operators on integers. `&&` and `||` only work on booleans. Writing `5 && 3` is a compile error; `5 & 3` is bitwise AND = 1.

---

### Trap 9: Multiple short-circuits in one expression
```java
boolean r = check(true) || check(false) && check(true);
// && before ||: parses as check(true) || (check(false) && check(true))
// check(true) is true → || short-circuits → NEITHER check(false) NOR check(true) evaluated
```
The exam chains multiple short-circuit operators with precedence to make candidates think more evaluations happen than actually do.

---

### Trap 10: De Morgan trap — wrong transformation
`!(a && b)` is NOT `!a && !b` (that's wrong). It IS `!a || !b`. The exam presents the wrong transformation as an option. And `!(a || b)` is NOT `!a || !b` — it IS `!a && !b`. The AND/OR swap is the key.

---

## 5. Summary Sheet

### Operator Comparison Table

| Feature | `&&` | `\|\|` | `&` (bool) | `\|` (bool) | `^` (bool) |
|---|---|---|---|---|---|
| Short-circuits? | ✅ on `false` | ✅ on `true` | ❌ Always | ❌ Always | ❌ Always |
| Works on integers? | ❌ | ❌ | ✅ (bitwise) | ✅ (bitwise) | ✅ (bitwise) |
| Compound `=` form | ❌ None | ❌ None | ✅ `&=` | ✅ `\|=` | ✅ `^=` |
| Result type | boolean | boolean | boolean/int | boolean/int | boolean/int |

---

### Short-Circuit Rules

| Expression | Right Side Evaluated? | Why |
|---|---|---|
| `false && expr` | ❌ No | Result already `false` |
| `true && expr` | ✅ Yes | Need right side |
| `true \|\| expr` | ❌ No | Result already `true` |
| `false \|\| expr` | ✅ Yes | Need right side |
| `false & expr` | ✅ Yes | No short-circuit |
| `true & expr` | ✅ Yes | No short-circuit |
| `false \| expr` | ✅ Yes | No short-circuit |
| `true \| expr` | ✅ Yes | No short-circuit |

---

### XOR Truth Table

| A | B | `A ^ B` |
|---|---|---|
| true | true | false |
| true | false | true |
| false | true | true |
| false | false | false |

**Memory aid:** XOR = "different → true, same → false"

---

### Precedence Among Logical/Bitwise Operators

```
Highest  →  !  (unary NOT)
            &  (AND — bitwise/logical)
            ^  (XOR)
            |  (OR — bitwise/logical)
            && (short-circuit AND)
Lowest   →  || (short-circuit OR)
```

---

### De Morgan's Laws

| Original | Equivalent |
|---|---|
| `!(A && B)` | `!A \|\| !B` |
| `!(A \|\| B)` | `!A && !B` |

---

### Key Rules to Memorize

| Rule | Detail |
|---|---|
| `&&` short-circuits | Right side skipped when left is `false` |
| `\|\|` short-circuits | Right side skipped when left is `true` |
| `&`, `\|`, `^` never short-circuit | Both sides always evaluated |
| `&&=`, `\|\|=` | DO NOT EXIST — compile error |
| `&=`, `\|=`, `^=` | Valid compound assignment operators |
| `&&`, `\|\|` on integers | COMPILE ERROR — only for booleans |
| `&`, `\|`, `^` on integers | Bitwise operations — valid |
| `null & method()` | NPE — `&` evaluates method() even if left is false |
| `null && method()` | Safe — `&&` skips method() if null check fails |
| `^` result | Different operands → `true`; same → `false` |

---

### Null Safety Pattern

```java
// SAFE — short-circuit prevents NPE:
if (obj != null && obj.someMethod()) { ... }
if (obj == null || obj.someMethod()) { ... }

// DANGEROUS — non-short-circuit, both sides evaluated:
if (obj != null & obj.someMethod()) { ... }   // NPE if obj is null
if (obj == null | obj.someMethod()) { ... }   // NPE if obj is null
```

---
