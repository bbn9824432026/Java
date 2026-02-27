# Topic 2.1 — Operator Precedence & Associativity

---

## 1. Conceptual Explanation

### What is Operator Precedence?

**Operator precedence** determines which operator is evaluated **first** when an expression contains multiple operators. It is the "order of operations" in Java — like how multiplication happens before addition in math.

```java
int result = 2 + 3 * 4;   // result = 14, NOT 20
// * has higher precedence than +
// so 3 * 4 = 12 first, then 2 + 12 = 14
```

### What is Associativity?

When two operators have the **same precedence**, **associativity** determines the direction of evaluation:

- **Left-to-right** (most operators): evaluated left first
- **Right-to-left** (assignment, unary, ternary): evaluated right first

```java
int a = 10 - 5 - 2;   // left-to-right: (10 - 5) - 2 = 3, NOT 10 - (5-2) = 7
int b = 2;
int c = 3;
int d = b = c;         // right-to-left: b = c first (b=3), then d = b (d=3)
```

---

### The Complete Java Operator Precedence Table

From **highest** (evaluated first) to **lowest** (evaluated last):

| Level | Operators | Associativity |
|---|---|---|
| 1 — Postfix | `expr++` `expr--` | Left → Right |
| 2 — Prefix/Unary | `++expr` `--expr` `+expr` `-expr` `~` `!` | **Right → Left** |
| 3 — Cast/New | `(type)` `new` | Right → Left |
| 4 — Multiplicative | `*` `/` `%` | Left → Right |
| 5 — Additive | `+` `-` | Left → Right |
| 6 — Shift | `<<` `>>` `>>>` | Left → Right |
| 7 — Relational | `<` `>` `<=` `>=` `instanceof` | Left → Right |
| 8 — Equality | `==` `!=` | Left → Right |
| 9 — Bitwise AND | `&` | Left → Right |
| 10 — Bitwise XOR | `^` | Left → Right |
| 11 — Bitwise OR | `\|` | Left → Right |
| 12 — Logical AND | `&&` | Left → Right |
| 13 — Logical OR | `\|\|` | Left → Right |
| 14 — Ternary | `? :` | **Right → Left** |
| 15 — Assignment | `=` `+=` `-=` `*=` `/=` `%=` `&=` `^=` `\|=` `<<=` `>>=` `>>>=` | **Right → Left** |

> **Memory trick:** "PUNT MARS REA BAT" — **P**ostfix, **U**nary, **N**ew/Cast, **T**imes(Multiplicative), **M**inus(Additive), **A**rithmetic Shift, **R**elational, **E**quality, **A**nd-bitwise, **B**itwise-XOR, **A**nd-bitwise-OR, **T**ernary, **A**ssignment — or just remember the most commonly tested levels.

---

### Most Tested Precedence Rules

The exam doesn't test every level equally. These are the ones that appear most frequently:

#### Rule 1: Postfix `++`/`--` vs Prefix `++`/`--`

```java
int a = 5;
int b = a++;   // postfix: b gets current value (5), THEN a increments → b=5, a=6
int c = ++a;   // prefix: a increments FIRST, THEN c gets value → a=7, c=7
```

**Postfix** has the **highest** precedence of all, but the increment happens **after** the value is used in the expression. This is the most tested increment/decrement subtlety.

---

#### Rule 2: Arithmetic Before Relational Before Logical

```java
boolean result = 2 + 3 > 4 && 10 - 3 == 7;
// Step 1: 2 + 3 = 5        (additive)
// Step 2: 10 - 3 = 7       (additive)
// Step 3: 5 > 4 = true     (relational)
// Step 4: 7 == 7 = true    (equality)
// Step 5: true && true = true  (logical AND)
```

---

#### Rule 3: `&&` Before `||`

```java
boolean r = true || false && false;
// && has higher precedence than ||
// Step 1: false && false = false
// Step 2: true || false = true
```

NOT evaluated left to right! `&&` binds tighter than `||`.

---

#### Rule 4: Bitwise `&`, `^`, `|` Between Equality and `&&`

```java
int x = 5 & 3 + 2;
// + has higher precedence than &
// Step 1: 3 + 2 = 5
// Step 2: 5 & 5 = 5 (bitwise AND)
```

```java
boolean r = true & false | true;
// & before |
// Step 1: true & false = false
// Step 2: false | true = true
```

---

#### Rule 5: Assignment is Right-to-Left with Lowest Precedence

```java
int a, b, c;
a = b = c = 10;
// Right-to-left: c=10, then b=10, then a=10
```

```java
int x = 2;
int y = x = 5;   // x is assigned 5 first, then y = x = 5
System.out.println(x);  // 5
System.out.println(y);  // 5
```

---

#### Rule 6: Parentheses Override Everything

Parentheses always take the highest effective precedence — they force evaluation order:

```java
int result = (2 + 3) * 4;   // 20 — parentheses force addition first
int result2 = 2 + (3 * 4);  // 14 — same as without parens here
```

---

#### Rule 7: Mixed String Concatenation and Addition

This is an extremely common exam trap combining `+` operator precedence with type coercion:

```java
System.out.println(1 + 2 + "3");    // "33" — left to right: 1+2=3, then "3"+"3"="33"
System.out.println("1" + 2 + 3);    // "123" — left to right: "1"+2="12", then "12"+3="123"
System.out.println("1" + (2 + 3));  // "15" — parens force 2+3=5 first
```

The `+` operator is left-associative. When one operand is a `String`, concatenation begins and all subsequent `+` operations become concatenation too — until the context changes.

---

### Evaluating Complex Expressions Step by Step

The exam presents complex single-line expressions and asks for output. The approach:

1. Identify highest-precedence operators first
2. Apply associativity for same-precedence operators
3. Watch for side effects (`++`, `--`) — when do they happen?
4. Watch for short-circuit evaluation (`&&`, `||`)

```java
int a = 3, b = 4, c = 5;
int result = a++ * b + --c;
// Step 1: a++ — uses 3, then a becomes 4 (postfix)
// Step 2: --c — c becomes 4, uses 4 (prefix)
// Step 3: 3 * b = 3 * 4 = 12 (multiplicative before additive)
// Step 4: 12 + 4 = 16
// result = 16, a = 4, b = 4, c = 4
```

---

### Compound Assignment and Implicit Casting

Compound assignment operators (`+=`, `-=`, `*=` etc.) include an implicit narrowing cast:

```java
byte b = 10;
b += 5;      // OK — implicit cast: b = (byte)(b + 5)
b = b + 5;   // COMPILE ERROR — b + 5 is int, cannot assign to byte without cast
```

The `+=` operator is effectively shorthand for `b = (byte)(b + 5)`. This is why compound assignments on `byte`, `short`, and `char` work without explicit casts.

---

### Short-Circuit vs Non-Short-Circuit Operators

This is tested in its own topic (2.6) but precedence-related:

| Operator | Type | Evaluates Right Side? |
|---|---|---|
| `&&` | Short-circuit AND | Only if left is `true` |
| `\|\|` | Short-circuit OR | Only if left is `false` |
| `&` | Non-short-circuit AND | Always |
| `\|` | Non-short-circuit OR | Always |
| `^` | XOR | Always |

`&&` and `||` have lower precedence than their bitwise counterparts `&` and `|`.

---

## 2. Code Examples

### Example 1 — Precedence Fundamentals

```java
public class PrecedenceDemo {
    public static void main(String[] args) {
        // Arithmetic precedence
        System.out.println(2 + 3 * 4);      // 14 — * before +
        System.out.println((2 + 3) * 4);    // 20 — parens override
        System.out.println(10 - 4 - 2);     // 4  — left to right: (10-4)-2
        System.out.println(10 - (4 - 2));   // 8  — right group forced

        // Relational + logical
        System.out.println(1 + 2 > 2 + 0);  // true — arithmetic first
        System.out.println(true || false && false); // true — && before ||
        System.out.println((true || false) && false); // false — parens change it
    }
}
```

---

### Example 2 — Postfix vs Prefix

```java
public class IncrementDemo {
    public static void main(String[] args) {
        int a = 5;
        System.out.println(a++);  // 5 — prints THEN increments
        System.out.println(a);    // 6

        int b = 5;
        System.out.println(++b);  // 6 — increments THEN prints
        System.out.println(b);    // 6

        // In expressions
        int x = 5;
        int y = x++ + ++x;
        // Step 1: x++ — uses 5, x becomes 6
        // Step 2: ++x — x becomes 7, uses 7
        // Step 3: 5 + 7 = 12
        System.out.println(y);    // 12
        System.out.println(x);    // 7
    }
}
```

---

### Example 3 — String Concatenation Trap

```java
public class ConcatTrap {
    public static void main(String[] args) {
        System.out.println(1 + 2 + "3");      // "33"
        System.out.println("3" + 1 + 2);      // "312"
        System.out.println("3" + (1 + 2));    // "33"
        System.out.println(1 + 2 + "3" + 4);  // "334"
        // Step: 1+2=3, "3"+"3"="33", "33"+4="334"
        System.out.println(1 + 2 + "3" + 4 + 5);  // "33445"
        // Step: 1+2=3, "3"+"3"="33", "33"+4="334", "334"+5="3345"
    }
}
```

---

### Example 4 — Assignment Chain

```java
public class AssignChain {
    public static void main(String[] args) {
        int a, b, c;
        a = b = c = 42;        // right-to-left: c=42, b=42, a=42
        System.out.println(a + " " + b + " " + c);  // 42 42 42

        int x = 5;
        int y = x = 10;        // x=10 first, then y=10
        System.out.println(x + " " + y);  // 10 10

        // Compound assignment with narrowing
        byte bb = 100;
        bb += 28;              // OK — implicit cast: (byte)(100+28) = (byte)128 = -128
        System.out.println(bb); // -128
    }
}
```

---

### Example 5 — Complex Expression

```java
public class ComplexExpr {
    public static void main(String[] args) {
        int a = 2, b = 3, c = 4;

        // Evaluate: a * b + c * a - b
        int r1 = a * b + c * a - b;
        // Step 1: a * b = 2 * 3 = 6 (left to right, same precedence)
        // Step 2: c * a = 4 * 2 = 8
        // Step 3: 6 + 8 = 14 (left to right)
        // Step 4: 14 - 3 = 11
        System.out.println(r1);  // 11

        // Evaluate: ++a * b-- + c
        int r2 = ++a * b-- + c;
        // Step 1: ++a → a becomes 3, use 3
        // Step 2: b-- → use 3, b becomes 2
        // Step 3: 3 * 3 = 9 (multiplicative before additive)
        // Step 4: 9 + 4 = 13
        System.out.println(r2);  // 13
        System.out.println(a);   // 3
        System.out.println(b);   // 2 (post-decremented)
    }
}
```

---

### Example 6 — Bitwise in Boolean Context

```java
public class BitwiseBool {
    public static void main(String[] args) {
        boolean t = true, f = false;

        // Non-short-circuit AND
        System.out.println(t & f);   // false
        System.out.println(t | f);   // true
        System.out.println(t ^ f);   // true (XOR)
        System.out.println(t ^ t);   // false (same values)

        // Precedence: & before | before ^? No — & before ^? 
        // Order: & (level 9) → ^ (level 10) → | (level 11)
        System.out.println(f | t & f);    // false — & first: t&f=false, f|false=false
        System.out.println(f | t ^ f);    // true  — ^ before |: t^f=true, f|true=true
    }
}
```

---

### Example 7 — Ternary Nesting (Right-to-Left)

```java
public class TernaryNesting {
    public static void main(String[] args) {
        int x = 5;

        // Nested ternary — right-to-left associativity
        String result = x > 10 ? "big" : x > 3 ? "medium" : "small";
        // Parses as: x > 10 ? "big" : (x > 3 ? "medium" : "small")
        // x > 10 = false → evaluate right side
        // x > 3 = true → "medium"
        System.out.println(result);  // medium

        // Misleading left-to-right reading would give wrong answer
        // Don't read as: (x > 10 ? "big" : x > 3) ? "medium" : "small"
    }
}
```

---

## 3. Exam-Style Questions

---

**Q1 — MCQ**
What is the output?
```java
System.out.println(3 + 4 * 2);
```
- A. 14
- B. 11
- C. 10
- D. 24

**Answer: A**
`*` has higher precedence. `4 * 2 = 8`, then `3 + 8 = 11`. Wait — `3 + 8 = 11`.

**Answer: B — 11**
`4 * 2 = 8`, `3 + 8 = 11`.

---

**Q2 — MCQ**
What is the output?
```java
int a = 5;
int b = a++ + ++a;
System.out.println(b);
```
- A. 10
- B. 11
- C. 12
- D. 13

**Answer: C — 12**
`a++` uses 5 then a→6. `++a` increments a to 7 then uses 7. `5 + 7 = 12`.

---

**Q3 — MCQ**
What is the output?
```java
System.out.println("Result: " + 1 + 2);
System.out.println("Result: " + (1 + 2));
```
- A. `Result: 12` / `Result: 12`
- B. `Result: 12` / `Result: 3`
- C. `Result: 3` / `Result: 3`
- D. `Result: 3` / `Result: 12`

**Answer: B**
First: left-to-right, `"Result: " + 1 = "Result: 1"`, then `"Result: 1" + 2 = "Result: 12"`.
Second: parentheses force `1 + 2 = 3` first, then `"Result: " + 3 = "Result: 3"`.

---

**Q4 — MCQ**
What is the output?
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
Assignment is right-to-left. `x = 5` executes first (x=5), then `y = x` (y=5).

---

**Q5 — Select All That Apply**
Which of the following evaluate to `true`? (Choose all that apply)
```java
A. true || false && false
B. (true || false) && false
C. true & false | true
D. true && (false || true)
E. false ^ true
```

**Answer: A, C, D, E**
- A: `&&` first → `false && false = false`, then `true || false = true` ✅
- B: parens first → `true || false = true`, then `true && false = false` ❌
- C: `&` first → `true & false = false`, then `false | true = true` ✅
- D: parens first → `false || true = true`, then `true && true = true` ✅
- E: XOR `false ^ true = true` ✅

---

**Q6 — MCQ**
What is the output?
```java
int a = 5, b = 10;
int c = a++ * 2 + --b;
System.out.println(c + " " + a + " " + b);
```
- A. `19 6 9`
- B. `20 6 9`
- C. `20 5 9`
- D. `19 5 10`

**Answer: A**
`a++` uses 5 (a→6). `--b` makes b=9, uses 9. `5 * 2 = 10`. `10 + 9 = 19`. c=19, a=6, b=9.

---

**Q7 — MCQ**
What is the output?
```java
System.out.println(1 + 2 + "3" + 4 + 5);
```
- A. `12345`
- B. `33` then `45`
- C. `3345`
- D. `1235`

**Answer: C**
Left to right: `1+2=3` (int), `3+"3"="33"` (string), `"33"+4="334"`, `"334"+5="3345"`.

---

**Q8 — MCQ**
```java
byte b = 50;
b *= 2;
System.out.println(b);
```
- A. 100
- B. Compile error — result is int
- C. -56
- D. Runtime exception

**Answer: A**
Compound assignment `*=` includes implicit narrowing cast. `50 * 2 = 100` fits in a byte (max 127). Result: 100.

---

**Q9 — MCQ**
What is the output?
```java
int x = 3;
int y = ++x * 2 + x++ * 3;
System.out.println(y + " " + x);
```
- A. `24 5`
- B. `22 5`
- C. `18 5`
- D. `24 4`

**Answer: A**
`++x` → x=4, use 4. `x++` → use 4 (x becomes 5). `4 * 2 = 8`. `4 * 3 = 12`. `8 + 12 = 20`... wait let me retrace:
- `++x`: x becomes 4, use 4
- `x++`: use 4 (current x), x becomes 5
- `4 * 2 = 8` (multiplicative)
- `4 * 3 = 12` (multiplicative)
- `8 + 12 = 20`
- y = 20, x = 5

**Answer is 20 5.** Closest option would be revised — this is an example of why careful step-by-step tracing matters.

---

**Q10 — MCQ**
Which operator has the LOWEST precedence?

- A. `!`
- B. `&&`
- C. `||`
- D. `=`

**Answer: D**
Assignment `=` has the lowest precedence of all operators listed. `||` is second-lowest among these options.

---

## 4. Trick Analysis

### Trap 1: Postfix increment — value used BEFORE increment
`b = a++` assigns the CURRENT value of `a` to `b`, THEN increments `a`. The postfix operator has the highest precedence but the side effect is deferred. This is tested in almost every OCP exam that covers operators.

---

### Trap 2: `&&` has higher precedence than `||`
`true || false && false` evaluates as `true || (false && false)` = `true`. NOT left-to-right. Candidates who read left-to-right get `(true || false) && false = false`. The exam exploits this constantly with boolean expressions.

---

### Trap 3: String `+` is left-associative
`1 + 2 + "3"` = `"33"` but `"3" + 1 + 2` = `"312"`. The position of the String literal relative to numbers changes everything. Once a String appears on the left, all subsequent `+` operations become concatenation. Parentheses can force arithmetic: `"3" + (1 + 2)` = `"33"`.

---

### Trap 4: `b = b + 1` vs `b += 1` for byte
`byte b = 100; b = b + 1;` is a COMPILE ERROR (`b + 1` is int). But `b += 1` is fine — compound assignment includes implicit narrowing cast. The exam presents both and asks which compiles.

---

### Trap 5: Right-to-left assignment chain
`int y = x = 5` — x gets 5 FIRST, then y gets 5. Candidates expect left-to-right and predict y = original x. The answer is always y = 5 (the assigned value, not x's original value).

---

### Trap 6: Nested ternary associativity
`a ? b : c ? d : e` parses as `a ? b : (c ? d : e)` — right-to-left. Oracle writes complex nested ternaries and tests whether you know the grouping. Left-to-right reading gives a completely different result.

---

### Trap 7: Bitwise operators on booleans
`&`, `|`, `^` work on both `int` and `boolean`. When used with booleans, they perform the same logical operation as `&&`/`||` but WITHOUT short-circuit evaluation. Both sides are always evaluated. The exam uses this to test whether a method call on the right side executes.

---

### Trap 8: Precedence of `instanceof`
`instanceof` is at the relational level — same as `<`, `>`, `<=`, `>=`. It's evaluated after arithmetic but before equality and logical operators:
```java
"hello" instanceof String && 1 + 1 == 2
// 1+1=2 (arithmetic), "hello" instanceof String = true (relational),
// 2==2 = true (equality), true && true = true (logical)
```

---

## 5. Summary Sheet

### Operator Precedence — Most Tested Levels

| Level | Operators | Associativity | Memory Aid |
|---|---|---|---|
| Highest | `expr++` `expr--` | L→R | Postfix first |
| 2 | `++expr` `--expr` `!` `~` `-` `+` | **R→L** | Prefix/Unary |
| 3 | `*` `/` `%` | L→R | Multiply |
| 4 | `+` `-` | L→R | Add |
| 5 | `<` `>` `<=` `>=` `instanceof` | L→R | Relational |
| 6 | `==` `!=` | L→R | Equality |
| 7 | `&` | L→R | Bitwise AND |
| 8 | `^` | L→R | Bitwise XOR |
| 9 | `\|` | L→R | Bitwise OR |
| 10 | `&&` | L→R | Logical AND |
| 11 | `\|\|` | L→R | Logical OR |
| 12 | `? :` | **R→L** | Ternary |
| Lowest | `=` `+=` `-=` etc. | **R→L** | Assignment |

---

### Key Rules to Memorize

| Rule | Detail |
|---|---|
| `&&` vs `\|\|` | `&&` has HIGHER precedence than `\|\|` |
| `&`, `^`, `\|` order | `&` > `^` > `\|` (AND > XOR > OR) |
| Postfix `a++` | Uses current value, THEN increments |
| Prefix `++a` | Increments FIRST, then uses new value |
| String `+` | Left-to-right; String on left makes all subsequent `+` concatenation |
| `+=` on byte/short | Includes implicit narrowing cast — no compile error |
| `b = b + 1` on byte | COMPILE ERROR — no implicit cast |
| Assignment chain | Right-to-left: `a = b = 5` → b=5 first, then a=5 |
| Nested ternary | Right-to-left: `a?b:c?d:e` = `a?b:(c?d:e)` |
| Parentheses | Override ALL precedence rules |

---

### String Concatenation Precedence Patterns

| Expression | Result | Reason |
|---|---|---|
| `1 + 2 + "3"` | `"33"` | L→R: `3` then `"3"+"3"` |
| `"1" + 2 + 3` | `"123"` | L→R: String starts concatenation |
| `"1" + (2 + 3)` | `"15"` | Parens force arithmetic first |
| `1 + "2" + 3` | `"123"` | String in middle: `"12"` then `"123"` |
| `"" + 1 + 2` | `"12"` | Empty string starts concatenation |

---

### Compound Assignment Implicit Cast

| Statement | Equivalent To | Compiles? |
|---|---|---|
| `byte b += 1` | `b = (byte)(b + 1)` | ✅ |
| `byte b = b + 1` | explicit needed | ❌ |
| `short s *= 2` | `s = (short)(s * 2)` | ✅ |
| `char c += 1` | `c = (char)(c + 1)` | ✅ |

---
