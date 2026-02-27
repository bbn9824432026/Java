# Topic 2.10 — `for`, `while`, `do-while` Loops

---

## 1. Conceptual Explanation

### Overview

Java has four loop constructs:
1. **`for` loop** — traditional, counter-based
2. **enhanced `for-each` loop** — iterates over arrays and `Iterable`
3. **`while` loop** — condition-checked before each iteration
4. **`do-while` loop** — condition-checked after each iteration

All four are tested heavily on OCP — not just syntax but subtle behavioral differences, compile-time rules, infinite loops, unreachable code, and interactions with `break`/`continue`.

---

### The Traditional `for` Loop

**Full syntax:**
```java
for (initialization; condition; update) {
    body
}
```

Three components — all optional:
1. **Initialization** — runs once before the loop starts
2. **Condition** — evaluated before EACH iteration; loop continues while `true`
3. **Update** — runs after EACH iteration body completes

**Execution order:**
```
init → [condition check → body → update] → [condition check → ...] → done
```

```java
for (int i = 0; i < 5; i++) {
    System.out.print(i + " ");
}
// Output: 0 1 2 3 4
```

---

### `for` Loop — Multiple Variables and Expressions

The initialization and update sections can contain **multiple expressions** separated by commas:

```java
// Multiple initialization variables (must be same type)
for (int i = 0, j = 10; i < j; i++, j--) {
    System.out.print(i + ":" + j + " ");
}
// Output: 0:10  1:9  2:8  3:7  4:6

// Cannot declare variables of different types:
// for (int i = 0, long l = 0; ...) — COMPILE ERROR

// Update can have multiple expressions:
for (int i = 0; i < 10; i++, System.out.print(i + " ")) {
    // print happens in update section
}
```

---

### `for` Loop — All Sections Optional

Every section of a `for` loop is optional:

```java
// Empty for — infinite loop
for (;;) {
    // runs forever
}

// No initialization
int i = 0;
for (; i < 5; i++) { ... }

// No update (manual update inside body)
for (int j = 0; j < 5;) {
    j++;
}

// No condition — infinite loop (equivalent to while(true))
for (int k = 0; ; k++) {
    if (k >= 5) break;
}
```

> **Exam note:** `for(;;)` is an infinite loop — it compiles and runs forever. The missing condition is treated as `true`.

---

### Variable Scope in `for` Loop

Variables declared in the initialization section are scoped to the **entire `for` construct** — including the condition, update, and body:

```java
for (int i = 0; i < 5; i++) {
    System.out.println(i);
}
// System.out.println(i);  // COMPILE ERROR — i out of scope

// Variables declared outside live beyond the loop:
int j = 0;
for (; j < 5; j++) { }
System.out.println(j);  // 5 — j still in scope
```

---

### The Enhanced `for-each` Loop

**Syntax:**
```java
for (type variable : iterable) {
    body
}
```

Works with:
- **Arrays** (any type)
- Any object implementing `java.lang.Iterable` (all Collections, etc.)

```java
int[] nums = {1, 2, 3, 4, 5};
for (int n : nums) {
    System.out.print(n + " ");
}
// Output: 1 2 3 4 5

List<String> names = List.of("Alice", "Bob", "Carol");
for (String name : names) {
    System.out.println(name);
}
```

---

### `for-each` — Key Limitations

1. **Cannot modify the array/collection** via the loop variable (for arrays):

```java
int[] arr = {1, 2, 3};
for (int n : arr) {
    n = n * 2;   // modifies LOCAL copy, not array element
}
System.out.println(Arrays.toString(arr));  // [1, 2, 3] — unchanged!
```

2. **Cannot access the index** directly:

```java
// No i available in for-each:
for (String s : list) {
    // no index access — use traditional for if you need the index
}
```

3. **Cannot iterate in reverse** directly.

4. **Cannot remove elements** from a collection during iteration (use `Iterator` instead).

5. **`var` works in for-each** (Java 10+):

```java
var list = List.of("a", "b", "c");
for (var s : list) {
    System.out.println(s);
}
```

---

### `for-each` with `var`

```java
int[] arr = {1, 2, 3};
for (var n : arr) {
    // n is inferred as int
    System.out.println(n);
}

List<String> list = new ArrayList<>();
for (var s : list) {
    // s is inferred as String
    System.out.println(s.toUpperCase());
}
```

---

### The `while` Loop

**Syntax:**
```java
while (condition) {
    body
}
```

- Condition is evaluated **before** each iteration
- If condition is `false` initially, body **never executes**

```java
int i = 0;
while (i < 5) {
    System.out.print(i + " ");
    i++;
}
// Output: 0 1 2 3 4

// Condition false initially — body never runs:
int x = 10;
while (x < 5) {
    System.out.println("never");
}
System.out.println("after");  // "after" — always prints
```

---

### `while (true)` — Infinite Loop

```java
// Classic infinite loop with break:
int count = 0;
while (true) {
    if (count >= 5) break;
    System.out.print(count + " ");
    count++;
}
// Output: 0 1 2 3 4
```

---

### The `do-while` Loop

**Syntax:**
```java
do {
    body
} while (condition);
```

- Body executes **at least once** — condition checked AFTER the first iteration
- The semicolon after `while(condition)` is **required** — this is a common mistake

```java
int i = 0;
do {
    System.out.print(i + " ");
    i++;
} while (i < 5);
// Output: 0 1 2 3 4

// Even when condition is false — body runs once:
int x = 10;
do {
    System.out.println("runs once");  // prints!
} while (x < 5);
```

---

### `while` vs `do-while` — Key Difference

| Aspect | `while` | `do-while` |
|---|---|---|
| Condition checked | Before first iteration | After first iteration |
| Minimum executions | 0 (may never run) | 1 (always runs once) |
| Semicolon | Not required | **Required** after `while(condition)` |

```java
// while — may not execute:
int a = 10;
while (a < 5) {
    System.out.println("while body");  // never prints
}

// do-while — always executes once:
int b = 10;
do {
    System.out.println("do-while body");  // prints once
} while (b < 5);
```

---

### `break` in Loops

`break` immediately exits the **innermost** enclosing loop (or switch):

```java
for (int i = 0; i < 10; i++) {
    if (i == 5) break;
    System.out.print(i + " ");
}
// Output: 0 1 2 3 4
```

---

### `continue` in Loops

`continue` skips the rest of the current iteration and moves to the next:

```java
for (int i = 0; i < 10; i++) {
    if (i % 2 == 0) continue;  // skip even numbers
    System.out.print(i + " ");
}
// Output: 1 3 5 7 9
```

**In `for` loops:** after `continue`, the **update expression** still runs:

```java
for (int i = 0; i < 5; i++) {
    if (i == 2) continue;
    System.out.print(i + " ");
}
// Output: 0 1 3 4
// When i==2: continue → skips body → update (i++) still runs → i becomes 3
```

**In `while`/`do-while` loops:** after `continue`, the **condition** is re-evaluated immediately:

```java
int i = 0;
while (i < 5) {
    i++;
    if (i == 3) continue;  // skip body remainder, jump to condition check
    System.out.print(i + " ");
}
// Output: 1 2 4 5
// Note: i++ runs BEFORE continue check — so i is updated
```

---

### Labeled `break` and `continue`

Java supports **labels** on loops, allowing `break` and `continue` to target an outer loop:

```java
// Labeled break — exits outer loop
outer:
for (int i = 0; i < 3; i++) {
    for (int j = 0; j < 3; j++) {
        if (j == 1) break outer;  // exits OUTER loop
        System.out.println(i + "," + j);
    }
}
// Output: 0,0
// (when j==1 in first outer iteration, both loops exit)
```

```java
// Labeled continue — skips to next iteration of outer loop
outer:
for (int i = 0; i < 3; i++) {
    for (int j = 0; j < 3; j++) {
        if (j == 1) continue outer;  // skip to next i
        System.out.println(i + "," + j);
    }
}
// Output: 0,0 / 1,0 / 2,0
// Each time j==1, we skip rest of inner AND rest of outer iteration, move to next i
```

> **Label syntax:** label is an identifier followed by `:` placed before the loop. Labels are rarely used in production but are tested on OCP.

---

### Unreachable Code After `break`/`continue`/`return`

Code immediately following `break`, `continue`, or `return` inside a loop is **unreachable** and causes a compile error:

```java
for (int i = 0; i < 5; i++) {
    break;
    System.out.println(i);  // COMPILE ERROR — unreachable
}

while (true) {
    return;
    System.out.println("never");  // COMPILE ERROR — unreachable
}
```

However, `while(false)` body is also unreachable:

```java
while (false) {
    System.out.println("never");  // COMPILE ERROR — unreachable
}

// But for(;;) and while(true) body IS reachable:
while (true) {
    System.out.println("reachable");  // OK — compiles
    break;
}
```

---

### Infinite Loop Detection

The compiler detects some unreachable code after infinite loops:

```java
while (true) { }
System.out.println("after");  // COMPILE ERROR — unreachable after infinite loop

for (;;) { }
System.out.println("after");  // COMPILE ERROR — unreachable

// BUT:
while (true) {
    break;  // loop can end
}
System.out.println("after");  // OK — reachable because break exits
```

---

### Loop Variables and Scope — Detailed Rules

```java
// for loop variable — scoped to for construct
for (int i = 0; i < 5; i++) { }
// i not accessible here

// Redeclaration in sequential loops — OK
for (int i = 0; i < 3; i++) { }
for (int i = 0; i < 3; i++) { }  // OK — first i is gone

// Cannot shadow outer variable in for init:
int i = 10;
for (int i = 0; i < 5; i++) { }  // COMPILE ERROR — i already declared
```

---

### Initialization in `for` Loop — Rules

Multiple variables can be declared in initialization but they must be the **same type**:

```java
for (int i = 0, j = 10; i < j; i++, j--) { }     // OK — both int
for (int i = 0, j = 10, k = 5; ...) { }            // OK — all int

// Different types — COMPILE ERROR:
// for (int i = 0, long l = 0L; ...) { }            // ERROR

// Already declared variable:
int x = 0;
// for (int x = 0; ...) { }   // ERROR — x already in scope

// But variable assignment (not declaration) in init:
int y;
for (y = 0; y < 5; y++) { }  // OK — not declaring, just assigning
System.out.println(y);         // 5 — y still in scope
```

---

### `for-each` Internals — Arrays vs Iterables

For **arrays**, the for-each loop is compiled to a traditional indexed loop:
```java
for (int n : arr)  →  for (int i = 0; i < arr.length; i++) { int n = arr[i]; ... }
```

For **`Iterable`**, it uses an `Iterator`:
```java
for (String s : list)  →  Iterator<String> it = list.iterator(); while (it.hasNext()) { String s = it.next(); ... }
```

This matters for:
- Modification during iteration → `ConcurrentModificationException` for Collections
- Null collection → NPE (the iterable itself cannot be null)

```java
List<String> list = null;
for (String s : list) { }  // NullPointerException — cannot call null.iterator()

int[] arr = null;
for (int n : arr) { }      // NullPointerException — cannot access null.length
```

---

### `do-while` Missing Semicolon

A very common syntax error — the semicolon after `while(condition)` in `do-while` is mandatory:

```java
do {
    System.out.println("hello");
} while (true)   // COMPILE ERROR — missing semicolon!

do {
    System.out.println("hello");
} while (true);  // correct
```

---

### Loop with `return` — Void vs Value

```java
// In a void method — return exits the method
void printUntil5() {
    for (int i = 0; i < 10; i++) {
        if (i == 5) return;
        System.out.print(i + " ");
    }
}
// Output: 0 1 2 3 4

// In a value-returning method — loop must ensure return on all paths
int findFirst(int[] arr, int target) {
    for (int i = 0; i < arr.length; i++) {
        if (arr[i] == target) return i;
    }
    return -1;  // required — compiler needs guaranteed return
}
```

---

## 2. Code Examples

### Example 1 — `for` Loop Variants

```java
public class ForLoopVariants {
    public static void main(String[] args) {
        // Standard
        for (int i = 0; i < 5; i++) {
            System.out.print(i + " ");
        }
        System.out.println();  // 0 1 2 3 4

        // Counting down
        for (int i = 4; i >= 0; i--) {
            System.out.print(i + " ");
        }
        System.out.println();  // 4 3 2 1 0

        // Two variables
        for (int i = 0, j = 10; i < j; i += 2, j--) {
            System.out.print("(" + i + "," + j + ") ");
        }
        System.out.println();  // (0,10) (2,9) (4,8) (6,7)

        // Empty for — all sections optional
        int k = 0;
        for (;;) {
            if (k++ >= 3) break;
            System.out.print(k + " ");
        }
        System.out.println();  // 1 2 3
    }
}
```

---

### Example 2 — `for-each` Limitations

```java
import java.util.*;

public class ForEachLimits {
    public static void main(String[] args) {
        // Cannot modify array via loop variable
        int[] arr = {1, 2, 3, 4, 5};
        for (int n : arr) {
            n *= 2;    // modifies LOCAL COPY only
        }
        System.out.println(Arrays.toString(arr));  // [1, 2, 3, 4, 5] unchanged

        // Can modify objects via reference
        List<StringBuilder> sbs = new ArrayList<>();
        sbs.add(new StringBuilder("hello"));
        sbs.add(new StringBuilder("world"));
        for (StringBuilder sb : sbs) {
            sb.append("!");   // modifies the OBJECT (not the reference)
        }
        System.out.println(sbs);  // [hello!, world!] — changed!

        // Null iterable → NPE
        List<String> nullList = null;
        try {
            for (String s : nullList) { }   // NullPointerException
        } catch (NullPointerException e) {
            System.out.println("NPE from null iterable");
        }
    }
}
```

---

### Example 3 — `while` vs `do-while` Difference

```java
public class WhileVsDoWhile {
    public static void main(String[] args) {
        int x = 10;

        // while — condition false initially — body never executes
        System.out.println("while:");
        while (x < 5) {
            System.out.println("while body");  // never prints
        }
        System.out.println("after while");  // prints

        // do-while — body executes once regardless
        System.out.println("do-while:");
        do {
            System.out.println("do-while body");  // prints once
        } while (x < 5);
        System.out.println("after do-while");  // prints
    }
}
```

---

### Example 4 — `break` and `continue` in Loops

```java
public class BreakContinue {
    public static void main(String[] args) {
        // break — exit loop
        System.out.print("break: ");
        for (int i = 0; i < 10; i++) {
            if (i == 5) break;
            System.out.print(i + " ");
        }
        System.out.println();  // 0 1 2 3 4

        // continue — skip iteration
        System.out.print("continue: ");
        for (int i = 0; i < 10; i++) {
            if (i % 3 == 0) continue;
            System.out.print(i + " ");
        }
        System.out.println();  // 1 2 4 5 7 8

        // continue in while — increment must be BEFORE continue!
        System.out.print("while-continue: ");
        int i = 0;
        while (i < 5) {
            i++;                    // increment BEFORE continue check
            if (i == 3) continue;   // skip print for i==3
            System.out.print(i + " ");
        }
        System.out.println();  // 1 2 4 5
    }
}
```

---

### Example 5 — Labeled `break` and `continue`

```java
public class LabeledLoops {
    public static void main(String[] args) {
        // Labeled break — exits outer loop
        System.out.println("Labeled break:");
        outer:
        for (int i = 0; i < 4; i++) {
            for (int j = 0; j < 4; j++) {
                if (i == 2 && j == 2) break outer;
                System.out.print("(" + i + "," + j + ") ");
            }
        }
        System.out.println();

        // Labeled continue — continue outer loop
        System.out.println("Labeled continue:");
        outer:
        for (int i = 0; i < 3; i++) {
            for (int j = 0; j < 3; j++) {
                if (j == 1) continue outer;  // skip to next i
                System.out.print("(" + i + "," + j + ") ");
            }
            System.out.print("[end of i=" + i + "] ");  // never reached
        }
        System.out.println();
        // Output: (0,0) (1,0) (2,0)
        // [end of i=...] never prints — continue outer skips it
    }
}
```

---

### Example 6 — Infinite Loop and Unreachable Code

```java
public class InfiniteLoops {
    public static void main(String[] args) {
        // while(true) compiles — body is reachable
        int count = 0;
        while (true) {
            System.out.print(count++ + " ");
            if (count >= 5) break;
        }
        System.out.println();  // 0 1 2 3 4

        // Code after while(true) WITHOUT break — unreachable:
        // while (true) { }
        // System.out.println("unreachable");  // COMPILE ERROR

        // while(false) — body is unreachable:
        // while (false) {
        //     System.out.println("unreachable");  // COMPILE ERROR
        // }
    }
}
```

---

### Example 7 — `continue` in `for` — Update Still Executes

```java
public class ContinueUpdate {
    public static void main(String[] args) {
        // continue skips body but update STILL runs
        for (int i = 0; i < 8; i++) {
            if (i % 2 == 0) continue;  // skip even
            System.out.print(i + " ");
            // When i=0: continue → skip print → i++ (i becomes 1)
            // When i=1: print "1" → i++ (i becomes 2)
            // etc.
        }
        System.out.println();  // 1 3 5 7

        // Contrast with while — must manually update or infinite loop:
        int j = 0;
        while (j < 8) {
            j++;              // update BEFORE continue or infinite loop!
            if (j % 2 == 0) continue;
            System.out.print(j + " ");
        }
        System.out.println();  // 1 3 5 7
    }
}
```

---

### Example 8 — `do-while` Classic Use Case

```java
import java.util.Scanner;

public class DoWhileUseCase {
    public static void main(String[] args) {
        // do-while perfect for "execute at least once" scenarios:
        // Menu display, input validation, etc.

        // Simulate input validation loop:
        int[] validInputs = {-1, 0, 3};  // simulate user inputs
        int index = 0;
        int input;

        do {
            input = validInputs[index++];
            System.out.println("Got input: " + input);
        } while (input <= 0);

        System.out.println("Valid input accepted: " + input);
        // Output:
        // Got input: -1
        // Got input: 0
        // Got input: 3
        // Valid input accepted: 3
    }
}
```

---

### Example 9 — Complex `for` Loop Tracing

```java
public class ComplexForTrace {
    public static void main(String[] args) {
        int sum = 0;

        // What does this print?
        for (int i = 1, j = 10; i <= 5; i++, j -= 2) {
            sum += i * j;
        }
        // i=1,j=10: sum += 1*10 = 10, then i=2,j=8
        // i=2,j=8:  sum += 2*8  = 26, then i=3,j=6
        // i=3,j=6:  sum += 3*6  = 44, then i=4,j=4
        // i=4,j=4:  sum += 4*4  = 60, then i=5,j=2
        // i=5,j=2:  sum += 5*2  = 70, then i=6 → condition false → exit
        System.out.println(sum);  // 70
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
    if (i == 3) continue;
    System.out.print(i + " ");
}
```
- A. `0 1 2 3 4`
- B. `0 1 2 4`
- C. `0 1 2`
- D. `0 1 2 3`

**Answer: B**
When `i==3`, `continue` skips the print. `i` continues to 4. Output: `0 1 2 4`.

---

**Q2 — MCQ**
What is the output?
```java
int i = 0;
while (i < 5) {
    if (i == 3) continue;
    System.out.print(i + " ");
    i++;
}
```
- A. `0 1 2 4`
- B. `0 1 2`
- C. Infinite loop
- D. `0 1 2 3 4`

**Answer: C**
When `i==3`, `continue` jumps back to the condition check — but `i++` is AFTER `continue`, so it never executes. `i` stays at 3 forever → infinite loop.

---

**Q3 — MCQ**
What is the output?
```java
int x = 10;
do {
    System.out.print(x + " ");
    x -= 3;
} while (x > 5);
```
- A. `10 7`
- B. `10`
- C. `10 7 4`
- D. Nothing printed

**Answer: A**
`x=10`: print `10`, x→7. Condition: `7>5` true. `x=7`: print `7`, x→4. Condition: `4>5` false. Exit. Output: `10 7`.

---

**Q4 — Select All That Apply**
Which loop constructs guarantee the body executes at least once?

- A. `for (int i=0; i<0; i++)`
- B. `while (false)`
- C. `do { } while (false)`
- D. `do { } while (true)`
- E. `for (;;) { break; }`

**Answer: C, D, E**
`do-while` always executes once. `for(;;){break;}` executes body once then breaks. A and B never execute — condition false before first iteration.

---

**Q5 — MCQ**
What is the output?
```java
outer:
for (int i = 0; i < 3; i++) {
    for (int j = 0; j < 3; j++) {
        if (j == 1) continue outer;
        System.out.print(i + "" + j + " ");
    }
}
```
- A. `00 01 02 10 11 12 20 21 22`
- B. `00 10 20`
- C. `00 01 10 11 20 21`
- D. `00`

**Answer: B**
Each time `j==1`, `continue outer` skips to next `i`. Only `j==0` prints for each `i`. Output: `00 10 20`.

---

**Q6 — MCQ**
Which causes a compile error?
```java
A. for (int i = 0, j = 0; i < 5; i++) { }
B. for (int i = 0, long l = 0; i < 5; i++) { }
C. for (int i = 0; ; i++) { break; }
D. for (;;) { break; }
```

- A. Line A
- B. Line B
- C. Line C
- D. Line D

**Answer: B**
Cannot declare variables of different types in `for` initialization. `int i = 0, long l = 0` mixes types — compile error.

---

**Q7 — MCQ**
```java
int[] arr = {1, 2, 3, 4, 5};
for (int n : arr) {
    n = n * 2;
}
System.out.println(arr[2]);
```
- A. `6`
- B. `3`
- C. Compile error
- D. `0`

**Answer: B**
`for-each` loop variable `n` is a local copy. Modifying `n` does NOT modify `arr`. `arr[2]` remains `3`.

---

**Q8 — MCQ**
What is the output?
```java
int i = 0;
for (; i < 3; ) {
    System.out.print(i + " ");
    i++;
}
System.out.println(i);
```
- A. `0 1 2 3`
- B. `0 1 2` then `3` on new line
- C. `0 1 2` then nothing
- D. Compile error

**Answer: B**
Loop prints `0 1 2 ` (with trailing space), then `i++` makes `i=3`, condition `3<3` false, exits. `i` is still in scope (declared outside). `println(i)` prints `3` on new line.

---

**Q9 — MCQ**
```java
for (int i = 0; i < 5; i++) {
    System.out.print(i + " ");
    if (i == 2) break;
    System.out.print("x ");
}
```
- A. `0 x 1 x 2`
- B. `0 x 1 x 2 x`
- C. `0 x 1 x 2 3 4`
- D. `0 1 2`

**Answer: A**
`i=0`: print `0 `, `i!=2`, print `x `. `i=1`: print `1 `, `i!=2`, print `x `. `i=2`: print `2 `, `i==2` → `break`. No `x` for `i==2`.

---

**Q10 — MCQ**
Which code causes a compile error?
```java
A: while (false) { System.out.println("x"); }
B: while (true) { break; }
C: for(;;) { System.out.println("x"); break; }
D: do { } while (true);
```

- A. A only
- B. B only
- C. A and D
- D. None

**Answer: A**
`while(false)` body is unreachable — compile error. B, C, D all compile (the loops can terminate or continue normally).

---

**Q11 — MCQ**
What is the output?
```java
int sum = 0;
for (int i = 1; i <= 10; i++) {
    if (i % 2 != 0) continue;
    sum += i;
}
System.out.println(sum);
```
- A. `55`
- B. `25`
- C. `30`
- D. `20`

**Answer: C**
Sum of even numbers from 1 to 10: `2+4+6+8+10 = 30`.

---

**Q12 — MCQ**
```java
int i = 0;
loop:
while (i < 5) {
    i++;
    switch (i) {
        case 3: break loop;
        default: System.out.print(i + " ");
    }
}
System.out.println("done");
```
- A. `1 2 done`
- B. `1 2 3 done`
- C. `1 2 4 5 done`
- D. Compile error

**Answer: A**
`i=1`: switch default → print `1`. `i=2`: switch default → print `2`. `i=3`: switch `case 3` → `break loop` → exits the `while` loop (not just the switch). Prints `done`. Output: `1 2 done`.

---

## 4. Trick Analysis

### Trap 1: `continue` in `while` — infinite loop if update is after `continue`
```java
int i = 0;
while (i < 5) {
    if (i == 3) continue;   // i never increments past 3!
    i++;
}
```
When `i==3`, `continue` jumps back to the condition — `i++` never runs. Infinite loop. The fix: put `i++` BEFORE the `continue` check. In `for` loops this isn't an issue because the update section runs automatically.

---

### Trap 2: `for-each` doesn't modify the array
`for (int n : arr) { n *= 2; }` — `n` is a local copy. The array is unchanged. Candidates expect the array to be doubled. To modify: use a traditional indexed `for` loop.

---

### Trap 3: `do-while` requires semicolon after `while(condition)`
```java
do { } while (true)   // COMPILE ERROR — missing ;
do { } while (true);  // correct
```
The most common `do-while` syntax error. Easy to miss when reading exam code quickly.

---

### Trap 4: `while(false)` body is unreachable — compile error
`while(false)` is detected at compile time and its body is flagged as unreachable. But `while(someVariable)` compiles even if the variable is always `false` at runtime — the compiler can only detect literal `false`.

---

### Trap 5: `break` in switch inside a loop — exits switch only
```java
while (true) {
    switch (x) {
        case 1: break;  // breaks switch, NOT the while loop!
    }
    // still inside while loop after break
}
```
`break` without a label exits the **innermost** enclosing loop OR switch. If `break` is inside a `switch` inside a loop, it exits the switch — not the loop. Use `break labelName` to exit the outer loop.

---

### Trap 6: Labeled `break` exits the labeled construct, code after it runs
```java
outer:
for (...) { ... break outer; ... }
System.out.println("after");  // this IS reachable — outer loop just exited
```
After `break outer`, execution resumes AFTER the labeled loop. The code after the loop is reachable and runs normally.

---

### Trap 7: `for` loop variable cannot shadow outer variable
```java
int i = 10;
for (int i = 0; i < 5; i++) { }  // COMPILE ERROR — i already declared
```
The `for` initialization cannot re-declare a variable already in scope. Use a different name or don't declare in the `for` if using the outer variable.

---

### Trap 8: Multiple variables in `for` init must be same type
```java
for (int i = 0, j = 0; ...)   // OK
for (int i = 0, long l = 0; ...) // COMPILE ERROR — mixed types
```
All variables in the initialization section must be declared with a single type specifier.

---

### Trap 9: `continue outer` skips rest of outer iteration body
```java
outer:
for (int i = 0; i < 3; i++) {
    for (int j = 0; j < 3; j++) {
        if (j == 1) continue outer;
    }
    System.out.println("end of i=" + i);  // NEVER prints
}
```
`continue outer` skips to the next iteration of `outer` — including any code AFTER the inner loop in the outer iteration body. That `println` never runs.

---

### Trap 10: Null array or collection in `for-each` → NPE
```java
int[] arr = null;
for (int n : arr) { }  // NullPointerException
```
The for-each loop dereferences the iterable (accesses `.length` for arrays, calls `.iterator()` for collections). If null, NPE immediately. The body never even gets a chance to run.

---

## 5. Summary Sheet

### Loop Types Comparison

| Feature | `for` | `for-each` | `while` | `do-while` |
|---|---|---|---|---|
| Condition checked | Before each iter | Before each iter | Before each iter | **After** each iter |
| Minimum executions | 0 | 0 | 0 | **1** |
| Counter variable | ✅ Built-in | ❌ | Manual | Manual |
| Index access | ✅ | ❌ | Manual | Manual |
| Modify array elements | ✅ | ❌ | ✅ | ✅ |
| Works with `Iterable` | ❌ | ✅ | Via Iterator | Via Iterator |
| `break`/`continue` | ✅ | ✅ | ✅ | ✅ |

---

### `break` and `continue` Behavior

| Statement | Effect |
|---|---|
| `break` | Exit innermost loop or switch |
| `continue` | Skip rest of current iteration, go to next |
| `break label` | Exit the labeled loop |
| `continue label` | Skip to next iteration of labeled loop |
| `break` in switch inside loop | Exits switch only — NOT loop |

---

### `continue` — Where Execution Goes

| Loop Type | After `continue` goes to... |
|---|---|
| `for` | Update expression (`i++`), then condition |
| `while` | Condition check directly |
| `do-while` | Condition check directly |
| `for-each` | Next element |

---

### Unreachable Code Rules

| Code | Compiles? |
|---|---|
| `while(true) { }` followed by code | ❌ Unreachable |
| `while(false) { body }` | ❌ Unreachable body |
| `while(true) { break; }` followed by code | ✅ Reachable |
| `for(;;) { }` followed by code | ❌ Unreachable |
| Code after `break` in loop | ❌ Unreachable |
| Code after `return` in loop | ❌ Unreachable |

---

### `for` Loop — Valid Initialization Forms

| Form | Valid? | Why |
|---|---|---|
| `int i = 0, j = 0` | ✅ | Same type |
| `int i = 0, long l = 0` | ❌ | Mixed types |
| `i = 0` (i declared outside) | ✅ | Assignment, not declaration |
| `int i = 0` (i declared outside) | ❌ | Redeclaration |
| Empty | ✅ | All sections optional |

---

### Key Rules to Memorize

| Rule | Detail |
|---|---|
| `do-while` semicolon | Required after `while(condition)` |
| `while(false)` | Compile error — unreachable body |
| `for-each` modifies array? | ❌ — local copy only |
| Null iterable in for-each | NPE immediately |
| `continue` in `while` | Update must be BEFORE `continue` |
| `break` in switch in loop | Exits switch only |
| Labeled `break`/`continue` | Targets the named loop |
| `for` init variables | Must all be same type |
| Loop var shadows outer var | COMPILE ERROR |
| Non-final var in for-each lambda | COMPILE ERROR — not effectively final |

---

Say **"Next topic"** when ready for **Topic 2.11 — `break`, `continue`, `return` (with Labels)**.
