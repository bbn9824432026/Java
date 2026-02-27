# Topic 2.7 — Ternary Operator

---

## 1. Conceptual Explanation

### What Is the Ternary Operator?

The ternary operator is Java's only **three-operand operator**. It is a compact conditional expression that evaluates one of two expressions based on a boolean condition.

**Syntax:**
```
condition ? expressionIfTrue : expressionIfFalse
```

**How it works:**
1. `condition` is evaluated — must produce a `boolean` (or `Boolean`)
2. If `true` → `expressionIfTrue` is evaluated and returned
3. If `false` → `expressionIfFalse` is evaluated and returned

```java
int x = 10;
String result = (x > 5) ? "big" : "small";
System.out.println(result);  // "big"
```

---

### Ternary is an Expression, Not a Statement

This is the fundamental distinction between the ternary operator and an `if-else` statement. The ternary operator **returns a value** — it produces a result that can be:
- Assigned to a variable
- Passed as a method argument
- Used inside another expression
- Printed directly

```java
// Ternary as expression — must produce a value
int max = (a > b) ? a : b;

// Used directly in method call
System.out.println((x > 0) ? "positive" : "non-positive");

// Used in arithmetic
int abs = (n >= 0) ? n : -n;

// if-else is a statement — does NOT return a value
// int y = if (x > 0) x else -x;  // SYNTAX ERROR
```

---

### Both Branches Must Produce a Value

Unlike `if-else`, both the true and false branches of a ternary **must** be expressions that produce values:

```java
// Valid — both branches produce int values
int x = (flag) ? 10 : 20;

// Invalid — void method returns nothing
// (flag) ? System.out.println("yes") : System.out.println("no");
// COMPILE ERROR — void is not a value
```

> **Exam note:** Using a `void` method call in a ternary branch is a compile error. The ternary is not a replacement for `if-else` in all situations.

---

### Type of the Ternary Expression

The type of the ternary expression is determined by the types of the two branch expressions. This is where things get complex — and exam-worthy.

#### Case 1: Both branches same type → result is that type

```java
int a = 5, b = 10;
int result = (a > b) ? a : b;   // both int → result is int
```

#### Case 2: Both branches numeric → numeric promotion applies

```java
int i = 5;
double d = 2.5;
double result = (true) ? i : d;   // int and double → promoted to double
// i is promoted to double → result = 5.0

var r = (true) ? i : d;   // inferred as double
```

#### Case 3: One branch is `null` → type is the other branch's type

```java
String s = (true) ? null : "hello";   // type is String (null compatible with String)
String t = (false) ? "hello" : null;  // type is String
```

#### Case 4: Mixed reference types → least common supertype

```java
Object result = (flag) ? "hello" : Integer.valueOf(5);
// String and Integer → least common type is Object/Serializable/Comparable
```

#### Case 5: Autoboxing/unboxing in ternary — critical trap

```java
// One branch is int, other is Integer
Integer boxed = null;
int prim = 5;

int result = (true) ? prim : boxed;
// prim (int) and boxed (Integer) → result type is int (unboxing occurs)
// If false branch selected: boxed (null) unboxed → NullPointerException!

int result2 = (false) ? prim : boxed;  // NullPointerException — unboxing null
```

This is one of the most insidious exam traps — the ternary forces unboxing, and if the result type is primitive and the selected branch is a null wrapper, you get an NPE.

---

### Ternary Type Resolution — Full Rules

The JLS (Java Language Specification) determines the type of a ternary expression as follows:

**If both expressions have the same type:** result is that type.

**If both are numeric (byte/short/int/long/float/double/char):** apply numeric promotion.

**Special rule for small constants:**
If one operand is `byte`, `short`, or `char` and the other is a `int` compile-time constant that fits in the smaller type, the result is the smaller type:

```java
byte b = (true) ? (byte) 5 : 10;   // 10 is an int constant that fits in byte
// Hmm — this gets nuanced. Let's see:

byte result = (flag) ? b1 : b2;     // byte and byte → byte
short result2 = (flag) ? s1 : b1;   // short and byte → short
```

**If one operand is null:** type is the non-null operand's type.

**For reference types:** least specific type (common supertype).

---

### Short-Circuit Behavior in Ternary

Only ONE branch of the ternary is evaluated — the one selected by the condition. This is similar to short-circuit evaluation:

```java
int x = 5;
int result = (x > 3) ? x * 2 : x / 0;
// x > 3 is true → false branch (x / 0) is NEVER evaluated
// No ArithmeticException
System.out.println(result);  // 10
```

```java
String s = null;
int len = (s == null) ? 0 : s.length();
// s == null is true → false branch (s.length()) is NEVER evaluated
// No NPE
System.out.println(len);  // 0
```

> **Key rule:** Only the selected branch is evaluated. The non-selected branch has **no side effects**.

---

### Nested Ternary — Right-to-Left Associativity

The ternary operator is **right-to-left associative**. When ternary operators are chained without parentheses, they group from the right:

```java
// a ? b : c ? d : e
// Parses as:  a ? b : (c ? d : e)    — right-to-left

int x = 5;
String result = x > 10 ? "big" : x > 3 ? "medium" : "small";
// Parses as: x > 10 ? "big" : (x > 3 ? "medium" : "small")
// x > 10 = false → evaluate right side
// x > 3 = true → "medium"
System.out.println(result);  // "medium"
```

> **Exam trap:** Nested ternary groups RIGHT-to-LEFT. Reading left-to-right gives completely wrong results. Always parse from the rightmost ternary inward.

```java
// Confusing example:
int a = 1, b = 2, c = 3;
int result = a < b ? a < c ? a : c : b < c ? b : c;
// Parses: a<b ? (a<c ? a : c) : (b<c ? b : c)
// a<b = true → evaluate: a<c ? a : c
// a<c = true → result = a = 1
System.out.println(result);  // 1
```

---

### Ternary vs `if-else` — Key Differences

| Feature | Ternary | `if-else` |
|---|---|---|
| Returns a value | ✅ Yes | ❌ No |
| Used as expression | ✅ Yes | ❌ No |
| Both branches must be expressions | ✅ Yes | ❌ No |
| Void methods in branches | ❌ Not allowed | ✅ Allowed |
| Nesting | Limited (unreadable) | Cleaner |
| Type coercion | ✅ Automatic | ❌ Not applicable |

```java
// if-else — can call void methods
if (flag) {
    System.out.println("yes");
} else {
    System.out.println("no");
}

// Ternary — CANNOT use void methods as branches
// (flag) ? System.out.println("yes") : System.out.println("no");
// COMPILE ERROR

// Ternary — works as expression inside println
System.out.println(flag ? "yes" : "no");  // OK — ternary returns String
```

---

### Ternary in Variable Assignment

```java
// Assign minimum
int min = (a < b) ? a : b;

// Assign absolute value
int abs = (n >= 0) ? n : -n;

// Conditional message
String msg = (score >= 60) ? "Pass" : "Fail";

// Conditional with method call
String upper = (s != null) ? s.toUpperCase() : "N/A";

// Conditional array access
int val = (arr != null && arr.length > 0) ? arr[0] : -1;
```

---

### Ternary with Autoboxing — The NullPointerException Trap

This is the most dangerous ternary trap and appears frequently:

```java
// Scenario 1: condition determines which to evaluate
Integer boxed = null;
int primitive = 5;

// The type of this ternary is int (primitive wins over Integer when mixed)
// If false branch selected: unbox null → NPE!
int x = (true) ? primitive : boxed;   // OK — primitive selected, no unboxing of boxed
int y = (false) ? primitive : boxed;  // NPE — boxed selected, unboxed, null → NPE
```

```java
// Scenario 2: Map.get() returning null
Map<String, Integer> map = new HashMap<>();
// If key doesn't exist, get() returns null
int value = map.containsKey("key") ? map.get("key") : 0;
// If get() returns null: ternary type is int → unboxing null → NPE!

// Safe version:
Integer safeValue = map.get("key");
int value = (safeValue != null) ? safeValue : 0;
// Or use getOrDefault:
int value2 = map.getOrDefault("key", 0);
```

---

### Ternary as a Condition

The condition part must be `boolean` — not an integer, not `null`:

```java
int x = 5;
// String r = (x) ? "yes" : "no";    // COMPILE ERROR — int is not boolean
// String r = (null) ? "yes" : "no"; // COMPILE ERROR — null is not boolean

Boolean b = true;
String r = (b) ? "yes" : "no";       // OK — Boolean unboxed to boolean

Boolean nullBool = null;
// String r2 = (nullBool) ? "yes" : "no"; // NPE — unboxing null Boolean
```

---

### Ternary in String Concatenation

When ternary is used inside string concatenation, careful about type:

```java
int x = 5;
System.out.println("Value: " + (x > 3 ? "big" : "small")); // "Value: big"

// Without parens — precedence issue:
System.out.println("Value: " + x > 3 ? "big" : "small");
// Parses as: ("Value: " + x) > 3 ? "big" : "small"
// "Value: 5" > 3 — COMPILE ERROR — cannot compare String with int!
```

> **Exam trap:** Ternary without parentheses inside string concatenation. The `+` operator has higher precedence than `?:`, so the string concat happens before the ternary evaluates its condition unless you use parentheses.

---

### Ternary Return Type with Different Numeric Types

```java
// int and long
long r1 = (true) ? 5 : 10L;     // int 5 promoted to long → 5L
long r2 = (false) ? 5 : 10L;    // long 10L → 10L

// byte and int compile-time constant
byte by = 10;
int r3 = (true) ? by : 10;      // byte promoted to int → 10
byte r4 = (byte)((true) ? by : 10); // need cast

// Char and int
char c = 'A'; // 65
int r5 = (true) ? c : 100;     // char promoted to int → 65
```

---

## 2. Code Examples

### Example 1 — Ternary Fundamentals

```java
public class TernaryBasics {
    public static void main(String[] args) {
        int a = 10, b = 20;

        // Basic usage
        int max = (a > b) ? a : b;
        System.out.println("Max: " + max);   // 20

        // As method argument
        System.out.println(a > b ? "a is bigger" : "b is bigger");  // b is bigger

        // In arithmetic
        int abs = (a >= 0) ? a : -a;
        System.out.println("Abs: " + abs);   // 10

        // Nested
        int x = 5;
        String size = x > 10 ? "large" : x > 5 ? "medium" : "small";
        // Parses: x>10 ? "large" : (x>5 ? "medium" : "small")
        System.out.println(size);  // "small" — x=5, not > 5
    }
}
```

---

### Example 2 — Only Selected Branch Evaluated

```java
public class BranchEvaluation {
    static int counter = 0;

    static int sideEffect(int val) {
        counter++;
        return val;
    }

    public static void main(String[] args) {
        // True branch selected — false branch NOT evaluated
        counter = 0;
        int r1 = (true) ? sideEffect(10) : sideEffect(20);
        System.out.println(r1 + ", counter=" + counter);  // 10, counter=1

        // False branch selected — true branch NOT evaluated
        counter = 0;
        int r2 = (false) ? sideEffect(10) : sideEffect(20);
        System.out.println(r2 + ", counter=" + counter);  // 20, counter=1

        // Neither branch runs when condition causes short-circuit:
        int x = 5;
        counter = 0;
        int r3 = (x > 3) ? sideEffect(1) : (10 / 0);  // divide by zero never reached
        System.out.println(r3 + ", counter=" + counter);  // 1, counter=1
    }
}
```

---

### Example 3 — The Autoboxing/NPE Trap

```java
public class TernaryNPE {
    public static void main(String[] args) {
        Integer nullInt = null;
        int primitive = 10;

        // Type of ternary is int (primitive) — unboxing will occur
        // True branch selected — safe (primitive already int)
        try {
            int r1 = (true) ? primitive : nullInt;
            System.out.println("r1=" + r1);  // 10 — no NPE
        } catch (NullPointerException e) {
            System.out.println("NPE on r1");
        }

        // False branch selected — nullInt unboxed → NPE!
        try {
            int r2 = (false) ? primitive : nullInt;
            System.out.println("r2=" + r2);
        } catch (NullPointerException e) {
            System.out.println("NPE on r2");  // prints this
        }

        // Safe — assign to Integer (no unboxing)
        Integer r3 = (false) ? primitive : nullInt;
        System.out.println("r3=" + r3);  // null — no NPE, Integer assigned
    }
}
```

---

### Example 4 — Ternary Type Resolution

```java
public class TernaryTypes {
    public static void main(String[] args) {
        // int + double → double
        double r1 = (true) ? 5 : 2.5;
        System.out.println(r1);  // 5.0

        // byte + int constant → byte
        byte b = 10;
        int r2 = (true) ? b : 20;
        System.out.println(r2);  // 10 (as int)

        // char + int → int
        char c = 'A';
        int r3 = (false) ? c : 90;
        System.out.println(r3);  // 90 (int)

        // String + null → String
        String r4 = (true) ? "hello" : null;
        System.out.println(r4);  // hello

        // String + Integer → Object (common supertype)
        Object r5 = (false) ? "hello" : Integer.valueOf(42);
        System.out.println(r5);  // 42
        System.out.println(r5.getClass().getSimpleName()); // Integer
    }
}
```

---

### Example 5 — Nested Ternary Tracing

```java
public class NestedTernary {
    public static void main(String[] args) {
        // Grade classification
        int score = 75;
        String grade = score >= 90 ? "A"
                     : score >= 80 ? "B"
                     : score >= 70 ? "C"
                     : score >= 60 ? "D"
                     : "F";
        // Right-to-left:
        // Innermost: score>=60 ? "D" : "F" → score=75 ≥ 60 → "D"
        // No wait — these all chain from top:
        // score>=90 ? "A" : (score>=80 ? "B" : (score>=70 ? "C" : (score>=60 ? "D" : "F")))
        // score>=90 false → score>=80 false → score>=70 true → "C"
        System.out.println(grade);  // C

        // Tricky example
        int x = 2;
        int r = x == 1 ? 10 : x == 2 ? 20 : 30;
        // x==1 false → (x==2 ? 20 : 30) → x==2 true → 20
        System.out.println(r);  // 20
    }
}
```

---

### Example 6 — Ternary in Concatenation — Precedence Trap

```java
public class TernaryConcat {
    public static void main(String[] args) {
        int x = 5;
        boolean flag = true;

        // CORRECT — parens around ternary
        System.out.println("Result: " + (flag ? "yes" : "no"));  // Result: yes

        // TRICKY — no parens:
        System.out.println("x=" + x > 0 ? "pos" : "neg");
        // Parses as: ("x=" + x) > 0 ? "pos" : "neg"
        // "x=5" > 0 → COMPILE ERROR (comparing String with int)

        // Another trap:
        System.out.println(flag ? "a" : "b" + "c");
        // Parses as: flag ? "a" : ("b" + "c")
        // flag=true → "a"
        // The "b"+"c" concatenation is in false branch, not the whole expression
        System.out.println(!flag ? "a" : "b" + "c");  // "bc" — false branch
    }
}
```

---

### Example 7 — Ternary vs if-else

```java
public class TernaryVsIfElse {
    public static void main(String[] args) {
        int n = -5;

        // Ternary — expression-oriented
        int abs = n >= 0 ? n : -n;
        System.out.println(abs);  // 5

        // if-else equivalent
        int abs2;
        if (n >= 0) {
            abs2 = n;
        } else {
            abs2 = -n;
        }
        System.out.println(abs2); // 5

        // Where ternary wins: method arguments
        System.out.println(Math.abs(n) + " " + (n > 0 ? "positive" : "non-positive"));

        // Where if-else wins: void method calls, complex logic
        if (n > 0) {
            System.out.println("positive: " + n);
        } else {
            System.out.println("non-positive: " + n);
        }
        // Cannot do this with ternary — println is void
    }
}
```

---

### Example 8 — Boolean Condition Must Be boolean

```java
public class TernaryCondition {
    public static void main(String[] args) {
        // Boolean object — unboxed
        Boolean flag = Boolean.TRUE;
        String r = flag ? "yes" : "no";
        System.out.println(r);  // yes

        // Null Boolean — NPE
        Boolean nullFlag = null;
        try {
            String r2 = nullFlag ? "yes" : "no";  // NPE — unboxing null
        } catch (NullPointerException e) {
            System.out.println("NPE from null Boolean condition");
        }

        // int is NOT boolean — compile error:
        int x = 1;
        // String r3 = x ? "yes" : "no";  // COMPILE ERROR

        // Non-zero check requires explicit comparison:
        String r4 = x != 0 ? "yes" : "no";
        System.out.println(r4);  // yes
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
System.out.println(x > 3 ? "yes" : "no");
System.out.println(x > 10 ? x * 2 : x + 1);
```
- A. `yes` / `6`
- B. `yes` / `10`
- C. `no` / `6`
- D. `no` / `11`

**Answer: A**
`x > 3` is `true` → `"yes"`. `x > 10` is `false` → `x + 1 = 6`.

---

**Q2 — MCQ**
What is the output?
```java
int a = 1, b = 2, c = 3;
int result = a < b ? a < c ? a : c : b < c ? b : c;
System.out.println(result);
```
- A. `1`
- B. `2`
- C. `3`
- D. Compile error

**Answer: A**
Right-to-left: `a<b ? (a<c ? a : c) : (b<c ? b : c)`. `a<b (1<2)` = true → `a<c ? a : c`. `a<c (1<3)` = true → `a = 1`.

---

**Q3 — MCQ**
What is the output?
```java
Integer x = null;
int y = 5;
int result = (false) ? y : x;
System.out.println(result);
```
- A. `0`
- B. `null`
- C. `NullPointerException`
- D. Compile error

**Answer: C**
False branch selected → `x` (Integer null) is chosen. Result type is `int` (mixed int/Integer → int). Unboxing null → NPE.

---

**Q4 — Select All That Apply**
Which statements are true about the ternary operator? (Choose all that apply)

- A. Both branches must produce a value
- B. Only the selected branch is evaluated at runtime
- C. It can replace any `if-else` statement
- D. The condition must be `boolean` or `Boolean`
- E. Ternary associates right-to-left
- F. Void method calls can be used as branches

**Answer: A, B, D, E**
C is false — cannot replace `if-else` with void branches. F is false — void methods cannot be ternary branches. A, B, D, E are all correct.

---

**Q5 — MCQ**
What is the result type of this expression?
```java
(true) ? 5 : 2.5
```
- A. `int`
- B. `float`
- C. `double`
- D. Compile error — incompatible types

**Answer: C**
`int` and `double` → promoted to `double`. Result is `double` (5.0).

---

**Q6 — MCQ**
What is the output?
```java
int x = 0;
int y = (x != 0) ? (10 / x) : 99;
System.out.println(y);
```
- A. `99`
- B. `0`
- C. `ArithmeticException`
- D. Compile error

**Answer: A**
`x != 0` is `false` → false branch `99` selected. True branch `10 / x` is never evaluated. No division by zero.

---

**Q7 — MCQ**
What is the output?
```java
String s = null;
System.out.println("val=" + s != null ? s.toUpperCase() : "NULL");
```
- A. `NULL`
- B. `val=null`
- C. `NullPointerException`
- D. Compile error

**Answer: D** (or B depending on exact context)
Carefully parse: `("val=" + s) != null ? s.toUpperCase() : "NULL"`. `"val=" + null = "val=null"`. `"val=null" != null` is `true` → `s.toUpperCase()` is evaluated. `s` is `null` → NPE.

Actually **Answer: C** — NPE because the condition `"val=null" != null` is `true`, so `s.toUpperCase()` is called on null.

> **Lesson:** The `+` concatenation has higher precedence than `!=` which is higher than `?:`. Always use parentheses around the ternary condition.

---

**Q8 — MCQ**
```java
Boolean flag = null;
String result = flag ? "yes" : "no";
System.out.println(result);
```
- A. `null`
- B. `no`
- C. `NullPointerException`
- D. Compile error

**Answer: C**
`flag` is `Boolean` (wrapper). The ternary condition requires `boolean` (primitive) → unboxing occurs. Unboxing `null` → NPE.

---

**Q9 — MCQ**
What is the output?
```java
int x = 5;
String r = x > 3 ? "a" : "b" + "c";
System.out.println(r);
```
- A. `a`
- B. `bc`
- C. `abc`
- D. Compile error

**Answer: A**
`x > 3` is `true` → `"a"` selected. The `"b" + "c"` is in the false branch — not evaluated. The false branch's concatenation happens at compile time anyway but doesn't matter since it's not selected. Result: `"a"`.

---

**Q10 — MCQ**
```java
int a = 10;
int b = (a = 20) > 15 ? a : 0;
System.out.println(a + " " + b);
```
- A. `10 0`
- B. `20 20`
- C. `20 0`
- D. `10 20`

**Answer: B**
Condition: `(a = 20)` assigns 20 to `a`, returns 20. `20 > 15` is `true` → true branch: `a` = 20. `b = 20`. Output: `20 20`.

---

**Q11 — MCQ**
Which is the correct type of the ternary expression?
```java
boolean flag = true;
var result = flag ? 42 : "hello";
```
- A. `int`
- B. `String`
- C. `Object`
- D. Compile error — incompatible types in ternary

**Answer: C**
`int` and `String` — common supertype is `Object` (also `Serializable` and `Comparable`, but `Object` is what `var` infers as the most general type). Result type is `Object`.

Actually more precisely, the compile-time type is `Serializable & Comparable<?>` but `var` would show a specific intersection type. The exam typically expects **D (compile error)** as the answer for clearly incompatible types, or **C (Object)** when it compiles. This DOES compile — `var result` would be inferred as `Object`.

---

## 4. Trick Analysis

### Trap 1: Unboxing null in ternary → NPE
```java
int result = (false) ? 5 : nullInteger;
```
When the result type of a ternary is a primitive (because one branch is primitive), the other branch is unboxed. If that branch is a null wrapper, you get NPE. The fix is to assign to the wrapper type or check for null first.

---

### Trap 2: Ternary condition precedence in string concatenation
```java
System.out.println("x=" + x > 0 ? "pos" : "neg");
// NOT: "x=" + (x > 0 ? "pos" : "neg")
// BUT: ("x=" + x) > 0 ? "pos" : "neg"  → COMPILE ERROR
```
Always parenthesize the ternary inside string concatenation. The `+` operator has higher precedence than `?:`.

---

### Trap 3: Nested ternary — right-to-left, not left-to-right
```java
a ? b : c ? d : e
// = a ? b : (c ? d : e)     RIGHT-to-LEFT
// ≠ (a ? b : c) ? d : e     wrong reading
```
The exam writes multi-level ternaries and tests whether candidates parse them correctly. Always start from the rightmost `?:` and work outward.

---

### Trap 4: Void method as ternary branch — compile error
```java
flag ? System.out.println("yes") : System.out.println("no");  // COMPILE ERROR
```
`println` returns `void`. Ternary branches must produce values. Use `if-else` for void method calls.

---

### Trap 5: null Boolean condition → NPE
```java
Boolean b = null;
String r = b ? "yes" : "no";  // NPE — unboxing null Boolean
```
The ternary condition must be `boolean`. If a `Boolean` wrapper is provided, it's unboxed. Null `Boolean` unboxing throws NPE. Candidates expect `"no"` (treating null as false) — that's not how Java works.

---

### Trap 6: `int` is not `boolean` — no C-style truthiness
```java
int x = 1;
String r = x ? "yes" : "no";  // COMPILE ERROR — x is int, not boolean
```
Java does not treat non-zero integers as `true`. The condition must be explicitly boolean. The exam presents C-style `x ? "yes" : "no"` as a trap for candidates with C/C++ background.

---

### Trap 7: Type of mixed ternary
```java
double d = (true) ? 5 : 2.5;
// 5 is int, 2.5 is double → result is double → 5.0 stored
// Even though true branch (5) is selected, the type is double
```
The result TYPE is determined at compile time by both branches — not just the one that runs. So even if the `int` branch is selected, if the other branch is `double`, the `int` is promoted.

---

### Trap 8: Assignment in ternary condition
```java
int b = (a = 20) > 15 ? a : 0;
```
The condition `(a = 20)` assigns to `a` AND evaluates to 20. So `a` is already 20 by the time the true branch `a` is read. This creates a non-obvious dependency between the condition and the branches.

---

## 5. Summary Sheet

### Ternary Operator Quick Reference

| Aspect | Detail |
|---|---|
| Syntax | `condition ? trueExpr : falseExpr` |
| Condition type | Must be `boolean` or `Boolean` (unboxed) |
| Returns | A value — is an expression, not a statement |
| Branch evaluation | Only the selected branch is evaluated |
| Associativity | **Right-to-left** |
| Void methods as branches | ❌ Not allowed — compile error |
| null Boolean condition | NPE — unboxing null |
| int as condition | ❌ Compile error — not C |

---

### Type Resolution Rules

| Branch Types | Result Type |
|---|---|
| Same type | That type |
| numeric + numeric | Promoted (widening) |
| int + double | `double` |
| int + Integer | `int` (unboxing — NPE risk if null) |
| String + null | `String` |
| String + Integer | `Object` (common supertype) |
| int + String | Compile error / `Object` depending on context |

---

### Short-Circuit Behavior

| Situation | Branch Evaluated |
|---|---|
| Condition is `true` | True branch only |
| Condition is `false` | False branch only |
| Division by zero in false branch, condition true | False branch never evaluated — no exception |
| Method call in false branch, condition true | Method never called |

---

### Key Traps to Memorize

| Trap | Detail |
|---|---|
| `"str" + x > 0 ? ... : ...` | Parses as `("str"+x) > 0` — compile error |
| `nullInteger` with primitive result type | Unboxing → NPE |
| `null` Boolean as condition | NPE |
| Nested ternary direction | Right-to-left, not left-to-right |
| Type determined by both branches | Not just selected branch — affects promotion |
| `void` branch | Compile error |

---

### Ternary vs if-else Decision

| Use Ternary When | Use if-else When |
|---|---|
| Assigning one of two values | Calling void methods |
| Passing conditional argument | Complex multi-statement branches |
| Returning conditional value | More than two outcomes without nesting |
| Inline in expressions | Readability matters with complex logic |

---

### Operator Precedence Context

```
Higher  →  + (concatenation)
           > < >= <= (relational)
           == != (equality)
           ?: (ternary)    ← lower than + and comparisons
Lower   →  = (assignment)
```

Always parenthesize: `System.out.println("x: " + (flag ? "yes" : "no"));`

---
