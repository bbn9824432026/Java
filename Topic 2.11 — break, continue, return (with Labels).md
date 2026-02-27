# Topic 2.11 — `break`, `continue`, `return` (with Labels)

---

## 1. Conceptual Explanation

### Overview

`break`, `continue`, and `return` are **jump statements** — they transfer control flow to a different point in the program. While these were introduced in the context of loops and switches, this topic covers them exhaustively, including:

- Precise semantics of each statement
- Labeled versions of `break` and `continue`
- Where each is legal and illegal
- Interactions with `finally` blocks
- Unreachable code implications
- Every subtle exam trap

---

### `break` — Complete Semantics

`break` terminates the execution of the **innermost enclosing** `switch`, `for`, `for-each`, `while`, or `do-while`.

**Three legal contexts for `break`:**
1. Inside a `switch` statement
2. Inside any loop (`for`, `for-each`, `while`, `do-while`)
3. Inside a labeled block (with label)

```java
// In switch:
switch (x) {
    case 1:
        System.out.println("one");
        break;   // exits switch
}

// In for loop:
for (int i = 0; i < 10; i++) {
    if (i == 5) break;   // exits for loop
}

// In while:
while (true) {
    break;   // exits while
}
```

**`break` is NOT valid outside a loop or switch:**

```java
// Standalone break — COMPILE ERROR:
public void method() {
    break;   // COMPILE ERROR — not in loop or switch
}
```

---

### `break` in Nested Loops — Critical Distinction

`break` exits only the **innermost** enclosing loop — NOT all loops:

```java
for (int i = 0; i < 3; i++) {
    for (int j = 0; j < 3; j++) {
        if (j == 1) break;   // exits inner for loop only
        System.out.print(j + " ");
    }
    System.out.print("| ");  // still executes — outer loop continues
}
// Output: 0 | 0 | 0 |
// Inner loop breaks at j==1 each time, outer continues
```

---

### `break` in `switch` Inside a Loop

One of the most tested traps — `break` inside a `switch` exits the `switch`, not the enclosing loop:

```java
for (int i = 0; i < 5; i++) {
    switch (i) {
        case 3:
            break;   // exits SWITCH only — NOT the for loop!
    }
    System.out.print(i + " ");  // still prints for ALL values including 3
}
// Output: 0 1 2 3 4
// break exits switch, but outer for continues
```

To break out of the loop from inside a switch, use a **labeled break**:

```java
loop:
for (int i = 0; i < 5; i++) {
    switch (i) {
        case 3:
            break loop;   // exits the LOOP
    }
    System.out.print(i + " ");
}
// Output: 0 1 2
```

---

### `continue` — Complete Semantics

`continue` skips the remainder of the **current iteration** and proceeds to the next iteration of the **innermost enclosing loop**.

`continue` is **only valid inside loops** — not inside a switch (unless the switch is inside a loop):

```java
// continue skips to next iteration:
for (int i = 0; i < 5; i++) {
    if (i == 2) continue;
    System.out.print(i + " ");
}
// Output: 0 1 3 4

// continue in switch inside loop — valid, continues the LOOP:
for (int i = 0; i < 5; i++) {
    switch (i) {
        case 2: continue;   // continues the FOR loop (skips rest of iteration)
    }
    System.out.print(i + " ");
}
// Output: 0 1 3 4  (i==2 continues the for loop)
```

> **Key distinction:** `break` in a switch exits the switch. `continue` in a switch exits the switch AND continues the enclosing loop.

---

### Where Execution Goes After `continue`

This varies by loop type:

| Loop | After `continue` goes to |
|---|---|
| `for (init; cond; update)` | The **update** expression, then condition |
| `for-each` | Next element (next iteration) |
| `while (cond)` | The **condition** check |
| `do-while { } while (cond)` | The **condition** check |

This difference is critical for `while` loops — forgetting to update the loop variable before `continue` causes infinite loops:

```java
// INFINITE LOOP:
int i = 0;
while (i < 5) {
    if (i == 3) continue;   // i never incremented when i==3
    i++;
}

// CORRECT:
int i = 0;
while (i < 5) {
    i++;                     // increment first
    if (i == 3) continue;   // now safe to continue
    System.out.print(i + " ");
}
```

---

### Labeled Statements — Syntax and Purpose

A **label** is an identifier followed by `:` placed immediately before a statement (usually a loop or block):

```java
labelName:
for (int i = 0; i < 5; i++) {
    // ...
}
```

Labels allow `break` and `continue` to target **outer loops** rather than the innermost one.

**Label naming rules:**
- Any valid Java identifier
- Conventionally written in lowercase or with descriptive names
- Can label any statement — but `break`/`continue` only target loops (and `break` also targets switch)
- Same label name cannot be reused in nested scope for the same purpose

---

### Labeled `break`

`break labelName` exits the labeled construct entirely:

```java
outer:
for (int i = 0; i < 3; i++) {
    middle:
    for (int j = 0; j < 3; j++) {
        for (int k = 0; k < 3; k++) {
            if (k == 1) break middle;   // exits middle loop
            System.out.println(i+","+j+","+k);
        }
        System.out.println("after inner");  // skipped by break middle
    }
    System.out.println("after middle");  // still runs
}
```

```java
// Break outer from deepest nested:
outer:
for (int i = 0; i < 5; i++) {
    for (int j = 0; j < 5; j++) {
        if (i + j == 6) break outer;
        System.out.print("("+i+","+j+") ");
    }
}
System.out.println("done");  // runs after break outer
```

---

### Labeled `continue`

`continue labelName` skips to the next iteration of the labeled loop:

```java
outer:
for (int i = 0; i < 3; i++) {
    for (int j = 0; j < 3; j++) {
        if (j == 1) continue outer;
        System.out.print(i+","+j+" ");
    }
    System.out.print("[end-i="+i+"] ");  // NEVER prints — continue outer skips this
}
// Output: 0,0  1,0  2,0
```

> **Critical:** `continue outer` skips ALL remaining code in the outer iteration — including code after the inner loop but still inside the outer loop body.

---

### What Can Be Labeled

Any **statement** can technically have a label in Java, but `break`/`continue` can only reference:

- **`break`:** labels on loops, `switch`, or any block (`{ }`)
- **`continue`:** labels on loops ONLY

```java
// Labeled block with break — valid:
block:
{
    System.out.println("before break");
    if (true) break block;
    System.out.println("after break");  // not reached
}
System.out.println("after block");  // reached

// Labeled if with break — less common but valid:
label:
if (true) {
    break label;
}
```

---

### `return` — Complete Semantics

`return` exits the current **method** entirely, optionally providing a value:

```java
// Void method:
void method() {
    System.out.println("before");
    return;                        // exits method
    // System.out.println("after");  // COMPILE ERROR — unreachable
}

// Value-returning method:
int getMax(int a, int b) {
    if (a > b) return a;
    return b;
}
```

---

### `return` Without Value in `void` Method

In a `void` method, `return;` is optional but can be used to exit early:

```java
void printPositive(int x) {
    if (x <= 0) return;   // early exit
    System.out.println(x);
}
```

---

### `return` Must Cover All Code Paths in Non-`void` Methods

The compiler checks that every possible execution path returns a value:

```java
// COMPILE ERROR — not all paths return:
int method(boolean flag) {
    if (flag) return 1;
    // what if flag is false? No return statement
}

// OK — all paths covered:
int method(boolean flag) {
    if (flag) return 1;
    return 0;
}

// OK — with else:
int method(boolean flag) {
    if (flag) {
        return 1;
    } else {
        return 0;
    }
}

// OK — with throw:
int method(boolean flag) {
    if (flag) return 1;
    throw new RuntimeException("not flag");  // throw is a valid exit
}
```

---

### `return` in Constructor

Constructors can have `return` (without a value) for early exit:

```java
class Demo {
    Demo(int x) {
        if (x < 0) return;   // early exit from constructor
        // further initialization
    }
}
```

---

### `return` vs `break` — When to Use Which

| Statement | Exits | From |
|---|---|---|
| `break` | Innermost loop or switch | Loop/switch |
| `break label` | Labeled construct | Loop/switch/block |
| `continue` | Current iteration | Loop |
| `continue label` | Current iteration of labeled loop | Loop |
| `return` | Entire method | Anywhere in method |
| `return value` | Entire method, providing result | Non-void method |

---

### Interaction of Jump Statements with `finally`

`finally` blocks **always execute** — even when `break`, `continue`, or `return` is used inside a `try` block:

```java
// return inside try — finally still runs:
int method() {
    try {
        return 1;          // initiates return
    } finally {
        System.out.println("finally");  // runs before method actually returns
    }
}
// Output: "finally"
// Returns: 1
```

```java
// break inside try — finally still runs:
for (int i = 0; i < 3; i++) {
    try {
        if (i == 1) break;
    } finally {
        System.out.println("finally i=" + i);
    }
    System.out.println("body i=" + i);
}
System.out.println("after loop");
// Output:
// body i=0
// finally i=0    ← finally runs even on normal completion
// finally i=1    ← finally runs when break executes
// after loop
```

---

### `return` Value Override by `finally`

If both `try` and `finally` have `return` statements, the `finally` return **overrides** the `try` return:

```java
int method() {
    try {
        return 1;    // would return 1...
    } finally {
        return 2;    // ...but finally overrides — returns 2
    }
}
System.out.println(method());  // 2
```

> **Exam trap:** `return` in `finally` overrides `return` in `try`. This is considered bad practice but is valid Java. The exam tests this behavior.

---

### Unreachable Code — Complete Rules

The Java compiler performs **reachability analysis**. Code that can never be reached is a compile error (not just a warning):

```java
void method() {
    return;
    System.out.println("unreachable");  // COMPILE ERROR
}

void method2() {
    while (true) { }
    System.out.println("unreachable");  // COMPILE ERROR
}

void method3() {
    throw new RuntimeException();
    System.out.println("unreachable");  // COMPILE ERROR
}
```

**BUT — the compiler only detects CONSTANT conditions:**

```java
// COMPILE ERROR — literal false:
while (false) {
    System.out.println("unreachable");  // ERROR
}

// COMPILES — variable (even if always false at runtime):
boolean flag = false;
while (flag) {
    System.out.println("may be unreachable at runtime but compiles");  // OK
}
```

---

### Unreachable Code After `break`/`continue` Inside Block

```java
for (int i = 0; i < 5; i++) {
    break;
    System.out.println(i);  // COMPILE ERROR — unreachable
}

for (int i = 0; i < 5; i++) {
    continue;
    System.out.println(i);  // COMPILE ERROR — unreachable
}
```

---

### `break` and `continue` Cannot Be Used Outside Loops

```java
void method() {
    continue;   // COMPILE ERROR — not inside a loop
    break;      // COMPILE ERROR — not inside a loop or switch
}
```

---

### Label Scope Rules

Labels follow these scoping rules:
- A label is only visible in the **labeled statement** and its children
- Two sibling statements can have the same label name (non-overlapping)
- A nested label cannot reuse the name of an enclosing label

```java
// Two sibling loops can share label names — not enclosing each other:
label:
for (int i = 0; i < 2; i++) { }  // label goes out of scope here

label:
for (int j = 0; j < 2; j++) { }  // OK — same name, different scope

// CANNOT reuse label name in enclosing context:
outer:
for (int i = 0; i < 3; i++) {
    outer:                         // COMPILE ERROR — duplicate label
    for (int j = 0; j < 3; j++) { }
}
```

---

### `break` Without Label in `switch` Inside Loop — Deep Trace

This is the most heavily tested combination:

```java
for (int i = 0; i < 5; i++) {
    switch (i % 3) {
        case 0:
            System.out.print("Z");
            break;      // exits switch, NOT for loop
        case 1:
            System.out.print("O");
            break;      // exits switch
        case 2:
            System.out.print("T");
            // no break — falls through in switch
        default:
            System.out.print("!");
    }
}
// i=0: 0%3=0 → "Z", break switch
// i=1: 1%3=1 → "O", break switch
// i=2: 2%3=2 → "T", fall-through → "!"
// i=3: 3%3=0 → "Z", break switch
// i=4: 4%3=1 → "O", break switch
// Output: ZOTE!ZO... wait:
// i=0→Z, i=1→O, i=2→T!(fallthrough), i=3→Z, i=4→O
// Output: ZOT!ZO
```

---

## 2. Code Examples

### Example 1 — `break` in Switch vs Loop

```java
public class BreakSwitchVsLoop {
    public static void main(String[] args) {
        // break exits switch, loop continues:
        System.out.println("break in switch (no label):");
        for (int i = 0; i < 5; i++) {
            switch (i) {
                case 2:
                    System.out.print("[break-switch] ");
                    break;           // exits switch only
                default:
                    System.out.print(i + " ");
            }
        }
        System.out.println();
        // Output: 0 1 [break-switch] 3 4

        // break with label exits loop:
        System.out.println("break loop (with label):");
        loop:
        for (int i = 0; i < 5; i++) {
            switch (i) {
                case 2:
                    System.out.print("[break-loop] ");
                    break loop;      // exits for loop
                default:
                    System.out.print(i + " ");
            }
        }
        System.out.println();
        // Output: 0 1 [break-loop]
    }
}
```

---

### Example 2 — `continue` in Switch Inside Loop

```java
public class ContinueInSwitch {
    public static void main(String[] args) {
        for (int i = 0; i < 6; i++) {
            switch (i % 3) {
                case 0:
                    continue;   // continues the FOR loop — skips print below
                default:
                    System.out.print(i + " ");
            }
            System.out.print(". ");  // only prints when case != 0
        }
        System.out.println();
        // i=0: 0%3=0 → continue for loop → skip print and "."
        // i=1: 1%3=1 → default → print "1" → print ". "
        // i=2: 2%3=2 → default → print "2" → print ". "
        // i=3: 3%3=0 → continue for loop → skip
        // i=4: 4%3=1 → default → print "4" → print ". "
        // i=5: 5%3=2 → default → print "5" → print ". "
        // Output: 1 . 2 . 4 . 5 .
    }
}
```

---

### Example 3 — Labeled `break` Three-Level Nesting

```java
public class ThreeLevelBreak {
    public static void main(String[] args) {
        outer:
        for (int i = 0; i < 3; i++) {
            middle:
            for (int j = 0; j < 3; j++) {
                for (int k = 0; k < 3; k++) {
                    System.out.print(i+","+j+","+k+" ");
                    if (k == 1) break middle;  // exits j loop
                }
                System.out.print("[never] ");   // unreachable due to break middle
            }
            System.out.print("[i="+i+"] ");    // runs — outer continues
        }
        // For each i:
        //   j=0, k=0: print, k=1: print then break middle
        //   [i=...] prints
        // Output:
        // 0,0,0 0,0,1 [i=0]
        // 1,0,0 1,0,1 [i=1]
        // 2,0,0 2,0,1 [i=2]
    }
}
```

---

### Example 4 — `finally` with `break` and `return`

```java
public class FinallyWithJumps {
    static int withReturn() {
        try {
            System.out.println("try");
            return 1;
        } finally {
            System.out.println("finally");
            // return 2;   // if uncommented, overrides return 1
        }
    }

    static void withBreak() {
        for (int i = 0; i < 3; i++) {
            try {
                if (i == 1) break;
                System.out.println("try i=" + i);
            } finally {
                System.out.println("finally i=" + i);
            }
        }
        System.out.println("after loop");
    }

    public static void main(String[] args) {
        System.out.println("return value: " + withReturn());
        System.out.println("---");
        withBreak();
    }
    // Output:
    // try
    // finally
    // return value: 1
    // ---
    // try i=0
    // finally i=0
    // finally i=1    (break triggers finally for i=1)
    // after loop
}
```

---

### Example 5 — `return` Override by `finally`

```java
public class FinallyReturnOverride {
    static int test() {
        try {
            System.out.println("try: returning 1");
            return 1;
        } finally {
            System.out.println("finally: returning 2");
            return 2;    // overrides try's return!
        }
    }

    public static void main(String[] args) {
        System.out.println("Result: " + test());
    }
    // Output:
    // try: returning 1
    // finally: returning 2
    // Result: 2
}
```

---

### Example 6 — Unreachable Code Scenarios

```java
public class UnreachableCode {
    // These cause compile errors — commented out to show what's illegal:

    void method1() {
        return;
        // System.out.println("unreachable");  // ERROR
    }

    void method2() {
        while (true) { }
        // System.out.println("unreachable");  // ERROR
    }

    void method3(boolean b) {
        while (b) {
            // System.out.println("reachable");  // OK — b could be true
        }
        System.out.println("after");  // OK — b might be false
    }

    void method4() {
        // while (false) {
        //     System.out.println("unreachable");  // ERROR — literal false
        // }
    }

    int method5(boolean flag) {
        if (flag) return 1;
        return 0;   // required — otherwise compile error
    }
}
```

---

### Example 7 — Labeled `continue` Deep Trace

```java
public class LabeledContinueTrace {
    public static void main(String[] args) {
        int outerCount = 0, innerCount = 0;

        outer:
        for (int i = 0; i < 3; i++) {
            outerCount++;
            for (int j = 0; j < 3; j++) {
                innerCount++;
                if (i == 1 && j == 1) continue outer;
                System.out.print("("+i+","+j+") ");
            }
            System.out.print("[end-outer-"+i+"] ");
        }
        System.out.println();
        System.out.println("outerCount="+outerCount+", innerCount="+innerCount);

        // When i=1,j=1: continue outer
        // → skips rest of inner loop for i=1 (j=2 never runs)
        // → skips [end-outer-1]
        // → jumps to i=2

        // Output:
        // (0,0) (0,1) (0,2) [end-outer-0] (1,0) (2,0) (2,1) (2,2) [end-outer-2]
        // outerCount=3, innerCount=8
        // (inner: i=0→3, i=1→2(stops at j=1), i=2→3 = 8)
    }
}
```

---

### Example 8 — `break`/`continue` in Various Loop Types

```java
public class BreakContinueAllLoops {
    public static void main(String[] args) {
        // break in while
        int i = 0;
        while (true) {
            if (i++ == 3) break;
        }
        System.out.println("while break, i=" + i);  // 4

        // continue in do-while
        i = 0;
        do {
            i++;
            if (i == 3) continue;
            System.out.print(i + " ");
        } while (i < 5);
        System.out.println();  // 1 2 4 5

        // break in for-each
        int[] arr = {10, 20, 30, 40, 50};
        int sum = 0;
        for (int n : arr) {
            if (n > 30) break;
            sum += n;
        }
        System.out.println("sum=" + sum);  // 60 (10+20+30)
    }
}
```

---

## 3. Exam-Style Questions

---

**Q1 — MCQ**
What is the output?
```java
for (int i = 0; i < 5; i++) {
    switch (i) {
        case 2: break;
        default: System.out.print(i + " ");
    }
}
```
- A. `0 1 3 4`
- B. `0 1`
- C. `0 1 2 3 4`
- D. `0 1 3 4`

**Answer: A**
`break` exits the `switch` only. For `i==2`: switch executes `case 2: break` — exits switch, does NOT print, but outer loop continues. All other values hit `default` and print. Output: `0 1 3 4`.

---

**Q2 — MCQ**
What is the output?
```java
outer:
for (int i = 0; i < 3; i++) {
    for (int j = 0; j < 3; j++) {
        if (i == 1 && j == 1) break outer;
        System.out.print(i + "" + j + " ");
    }
}
System.out.print("done");
```
- A. `00 01 02 10 done`
- B. `00 01 02 10 11 done`
- C. `00 01 02 10 done`
- D. `00 01 02 done`

**Answer: A**
Prints up to `i=1,j=0` then at `i=1,j=1` breaks outer. `00 01 02 10 done`.

---

**Q3 — MCQ**
What is the output?
```java
int i = 0;
while (i < 5) {
    if (i++ == 2) continue;
    System.out.print(i + " ");
}
```
- A. `1 2 4 5`
- B. `1 2 3 4 5`
- C. Infinite loop
- D. `1 2 4 5 6`

**Answer: A**
`i++` is postfix — uses THEN increments. When condition `i++ == 2` is evaluated with `i=2`: uses 2 (true), increments i to 3, then `continue` skips print. Other iterations: `i=0`→false, i=1, print 1; `i=1`→false, i=2, print 2; `i=2`→true, i=3, continue; `i=3`→false, i=4, print 4; `i=4`→false, i=5, print 5. Output: `1 2 4 5`.

---

**Q4 — MCQ**
What is the output?
```java
static int test() {
    try {
        return 10;
    } finally {
        return 20;
    }
}
public static void main(String[] args) {
    System.out.println(test());
}
```
- A. `10`
- B. `20`
- C. Compile error
- D. Both 10 and 20

**Answer: B**
`finally` `return` overrides `try` `return`. Returns `20`.

---

**Q5 — Select All That Apply**
Which are valid uses of `break` in Java?

- A. Inside a `for` loop
- B. Inside a `while` loop
- C. Inside a `switch` statement
- D. Inside an `if` statement (not within a loop or switch)
- E. With a label targeting an enclosing loop
- F. Standalone in a method body with no enclosing loop or switch

**Answer: A, B, C, E**
D and F are invalid — `break` without a context (not inside loop/switch) is a compile error. A, B, C, E are all valid uses.

---

**Q6 — MCQ**
What is the output?
```java
outer:
for (int i = 0; i < 3; i++) {
    for (int j = 0; j < 3; j++) {
        if (j == 1) continue outer;
        System.out.print(i + "" + j + " ");
    }
    System.out.print("[" + i + "] ");  // does this print?
}
```
- A. `00 [0] 10 [1] 20 [2]`
- B. `00 10 20`
- C. `00 01 02 10 11 12 20 21 22`
- D. `00 10 20 [2]`

**Answer: B**
`continue outer` skips to the next `i` — including `[i]` print after inner loop. Only `j=0` is printed for each `i`. Output: `00 10 20`.

---

**Q7 — MCQ**
```java
void method() {
    for (int i = 0; i < 3; i++) {
        try {
            if (i == 1) break;
            System.out.println("try " + i);
        } finally {
            System.out.println("finally " + i);
        }
    }
    System.out.println("after");
}
```
What is the output?

- A. `try 0` / `finally 0` / `try 2` / `finally 2` / `after`
- B. `try 0` / `finally 0` / `finally 1` / `after`
- C. `try 0` / `finally 0` / `after`
- D. `try 0` / `finally 0` / `try 1` / `finally 1` / `after`

**Answer: B**
`i=0`: prints `try 0`, then `finally 0`. `i=1`: `break` triggered, but `finally 1` still runs before break takes effect. Loop exits. `after` prints. Output: `try 0 / finally 0 / finally 1 / after`.

---

**Q8 — MCQ**
Which causes a compile error?
```java
A: outer: for(int i=0;i<3;i++) { break outer; }
B: outer: for(int i=0;i<3;i++) { continue outer; }
C: outer: { break outer; }
D: outer: if(true) { continue outer; }
```

- A. A only
- B. D only
- C. A and B
- D. All compile

**Answer: B**
`continue` can only target loops — not blocks or `if` statements. `continue outer` where `outer` labels an `if` → compile error. A targets a loop with `break` ✅. B targets a loop with `continue` ✅. C labels a block with `break` ✅. D tries to `continue` an `if` ❌.

---

**Q9 — MCQ**
```java
int result = 0;
for (int i = 0; i < 5; i++) {
    if (i == 3) continue;
    result += i;
}
System.out.println(result);
```
- A. `10`
- B. `7`
- C. `6`
- D. `3`

**Answer: B**
Sum `0+1+2+4 = 7` (3 is skipped by `continue`).

---

**Q10 — Select All That Apply**
Which statements are true about labeled `continue`?

- A. It skips the rest of the current inner loop iteration
- B. It skips to the next iteration of the labeled (outer) loop
- C. Code after the inner loop but still inside the outer iteration is skipped
- D. The update expression of the labeled `for` loop still executes
- E. It can target a labeled `switch` statement
- F. It can target a labeled `if` statement

**Answer: A, B, C, D**
E and F are false — `continue` can only target loops. A: correct — skips inner iteration ✅. B: correct — moves to next outer iteration ✅. C: correct — skips code after inner loop in outer body ✅. D: correct — the `for` update still runs when continuing to it ✅.

---

**Q11 — MCQ**
```java
int n = 0;
label:
{
    n = 1;
    if (n > 0) break label;
    n = 2;
}
System.out.println(n);
```
- A. `0`
- B. `1`
- C. `2`
- D. Compile error — cannot use `break` with a block label

**Answer: B**
`break label` on a labeled block is valid. `n=1`, condition true, `break label` exits the block. `n=2` is skipped. Output: `1`.

---

## 4. Trick Analysis

### Trap 1: `break` in `switch` inside loop — exits switch, not loop
The single most tested `break` trap. When a `switch` is inside a `for` loop, `break` in the switch exits only the switch. The outer loop continues. To exit the loop from inside the switch, use `break loopLabel`.

---

### Trap 2: `continue` in `switch` inside loop — continues the loop
Opposite behavior from `break`. `continue` inside a switch that is inside a loop does NOT continue the switch (which has no "continue"). It continues the **enclosing loop**. This is counterintuitive but correct.

---

### Trap 3: `continue outer` skips code after inner loop
```java
outer: for (int i = 0; ...) {
    for (int j = 0; ...) { continue outer; }
    System.out.println("skipped!");  // NEVER runs
}
```
`continue outer` skips ALL remaining code in the outer iteration — including this `println`. Candidates think the `println` runs because it's "after the inner loop, still in outer".

---

### Trap 4: `finally` always runs — even with `break`/`continue`/`return`
`finally` runs regardless of how the `try` block exits. Break, continue, return, throw — `finally` always runs. The exam presents `try-finally` inside loops and tests exactly when `finally` executes.

---

### Trap 5: `return` in `finally` overrides `return` in `try`
If both `try` and `finally` have `return`, the `finally` return wins. The return value from `try` is discarded. This is tested with surprising output — the `finally` value is what gets returned.

---

### Trap 6: `continue` in `while` — infinite loop if update missing
```java
while (i < 5) {
    if (i == 3) continue;  // update never runs when i==3
    i++;
}
```
Classic infinite loop. The fix is always to put the update BEFORE the `continue` check (or use a `for` loop which handles updates automatically).

---

### Trap 7: Labeled `break` targets can be any statement — labeled `continue` only loops
`break label` can target a labeled loop, switch, or even a plain block `{ }`. `continue label` can ONLY target a labeled loop. `continue` on a labeled block or labeled `if` is a compile error.

---

### Trap 8: Unreachable code after `while(false)` is a compile error but `while(variable)` is not
The compiler detects `while(false)` as unreachable because it's a literal `false`. But `boolean b = false; while(b)` compiles — the compiler doesn't track variable values. The exam uses literal vs. variable conditions to test this.

---

### Trap 9: `return` must cover all paths — `throw` counts as a valid exit
```java
int method(boolean flag) {
    if (flag) return 1;
    throw new RuntimeException();  // valid exit — no second return needed
}
```
A `throw` statement is a valid code path terminator. The compiler accepts it as "this path doesn't need a return." The exam sometimes uses `throw` as the non-`return` exit path.

---

### Trap 10: Duplicate label name in enclosing scope
```java
outer:
for (int i = 0; i < 3; i++) {
    outer:                    // COMPILE ERROR — same name in enclosing scope
    for (int j = 0; j < 3; j++) { }
}
```
Two sibling loops CAN share a label name (sequential scopes). But an inner loop CANNOT reuse the name of an enclosing loop's label — that's a compile error.

---

## 5. Summary Sheet

### Jump Statement Contexts

| Statement | Valid In | Exits |
|---|---|---|
| `break` | Loop, `switch` | Innermost loop or switch |
| `break label` | Loop, `switch`, any block | The labeled construct |
| `continue` | Loop only | Current iteration |
| `continue label` | Loop only (label must be loop) | Current iteration of labeled loop |
| `return` | Any method/constructor | The entire method |
| `return value` | Non-void method | Method with value |

---

### `break` vs `continue` in `switch` Inside Loop

| Statement | Effect |
|---|---|
| `break` in switch inside loop | Exits switch only; loop continues |
| `continue` in switch inside loop | Exits switch AND continues the loop |
| `break loopLabel` in switch | Exits the labeled loop |

---

### Where Execution Resumes After Jump Statements

| Statement | Resumes At |
|---|---|
| `break` | First statement after the loop/switch |
| `break label` | First statement after the labeled construct |
| `continue` in `for` | Update expression (`i++`), then condition |
| `continue` in `while`/`do-while` | Condition check |
| `continue label` | Update (if `for`) or condition of labeled loop |
| `return` | Caller of the method |

---

### `finally` Interaction Rules

| Try block exits via | `finally` runs? | Value returned |
|---|---|---|
| Normal completion | ✅ Yes | N/A |
| `return value` | ✅ Yes | `try`'s value (unless `finally` also returns) |
| `return` + `finally return` | ✅ Yes | **`finally`'s value overrides** |
| `break` | ✅ Yes | N/A |
| `continue` | ✅ Yes | N/A |
| `throw` | ✅ Yes | Exception propagates (unless `finally` catches) |

---

### Unreachable Code Rules

| Code Situation | Compiles? |
|---|---|
| After `return` in same block | ❌ Compile error |
| After `break` in same block | ❌ Compile error |
| After `continue` in same block | ❌ Compile error |
| After `throw` in same block | ❌ Compile error |
| Body of `while(false)` | ❌ Compile error |
| Body of `while(variable)` | ✅ Compiles |
| After `while(true)` with no `break` | ❌ Compile error |
| After `while(true)` with `break` | ✅ Compiles |

---

### Labeled Constructs — What Can Be Targeted

| Target of Label | `break` Allowed? | `continue` Allowed? |
|---|---|---|
| `for` loop | ✅ | ✅ |
| `while` loop | ✅ | ✅ |
| `do-while` loop | ✅ | ✅ |
| `switch` statement | ✅ | ❌ |
| Plain block `{ }` | ✅ | ❌ |
| `if` statement | ✅ | ❌ |

---

### Key Rules to Memorize

| Rule | Detail |
|---|---|
| `break` in switch inside loop | Exits switch only — use `break label` to exit loop |
| `continue` in switch | Continues the enclosing LOOP |
| `continue outer` | Skips ALL remaining code in outer iteration |
| `finally` always runs | Even with `break`, `continue`, `return`, `throw` |
| `finally return` | Overrides `try return` |
| `while(false)` body | Compile error — unreachable |
| `while(variable)` body | Compiles even if always false at runtime |
| `continue` for loop | Update (`i++`) still runs |
| `continue` while loop | Jumps directly to condition |
| Duplicate label in enclosing scope | Compile error |
| `continue` on non-loop label | Compile error |
| `throw` ends a code path | No `return` needed after `throw` |

---
