# Topic 1.9 — Variable Scope & Lifetime

---

## 1. Conceptual Explanation

### What is Scope?

**Scope** defines the region of source code where a variable is **accessible by name**. Outside its scope, a variable simply does not exist as far as the compiler is concerned — referencing it causes a compile error.

**Lifetime** (also called **extent**) defines how long a variable's **storage exists in memory** at runtime. A variable's lifetime may extend beyond its scope in some cases (particularly with closures and effectively final captures), but for the exam, scope and lifetime are closely related.

Understanding scope is critical because it underlies:
- Where variables can be referenced
- When variables are created and destroyed
- What the compiler accepts and rejects
- How variable shadowing works
- Lambda capture rules (effectively final)

---

### The Four Scopes in Java

#### 1. Class Scope (Static Variables)
- Static variables exist for the **entire lifetime of the class**
- Created when the class is **first loaded** by the JVM
- Destroyed when the class is **unloaded** (effectively: when the JVM shuts down for most cases)
- Accessible anywhere in the class, and externally based on access modifier

```java
public class Counter {
    static int count = 0;    // class scope — lives as long as the class
}
```

#### 2. Instance Scope (Instance Variables)
- Instance variables exist for the **lifetime of the object**
- Created when `new` allocates the object
- Destroyed when the object becomes **eligible for garbage collection**
- Accessible through any reference to the object

```java
public class Person {
    String name;    // instance scope — lives as long as the Person object
    int age;
}
```

#### 3. Method Scope (Local Variables)
- Local variables exist from their **declaration** to the **end of the method**
- Created when the declaration is reached during execution
- Destroyed when the method returns (stack frame is popped)
- NOT accessible outside the method

```java
void calculate() {
    int result = 42;    // method scope — from here to end of method
    System.out.println(result);
}   // result destroyed here
```

#### 4. Block Scope
- Variables declared inside a block `{ }` exist only within that block
- Created when the declaration is reached
- Destroyed when the closing `}` is reached
- Applies to: `if`, `else`, `for`, `while`, `do-while`, `switch`, `try`, `catch`, `finally`, and any standalone `{ }` block

```java
void method() {
    if (true) {
        int x = 10;    // block scope
        System.out.println(x);  // OK
    }
    System.out.println(x);  // COMPILE ERROR — x out of scope
}
```

---

### Scope Rules — The Complete Picture

#### Rule 1: Variables are accessible from their declaration to end of their scope

A variable doesn't exist **before** its declaration in source code:

```java
void method() {
    System.out.println(x);  // COMPILE ERROR — x not yet declared
    int x = 5;
}
```

#### Rule 2: Inner blocks can access outer variables, but not vice versa

```java
void method() {
    int outer = 10;
    {
        int inner = 20;
        System.out.println(outer);  // OK — inner block sees outer
        System.out.println(inner);  // OK
    }
    System.out.println(outer);  // OK
    System.out.println(inner);  // COMPILE ERROR — outer can't see inner
}
```

#### Rule 3: You cannot declare a local variable with the same name as a local in an enclosing scope

```java
void method() {
    int x = 5;
    {
        int x = 10;  // COMPILE ERROR — x already defined in enclosing scope
    }
}
```

This is different from other languages (C++ allows this). Java prohibits it to avoid confusion.

#### Rule 4: You CAN reuse names in non-overlapping scopes

```java
void method() {
    {
        int x = 5;
    }  // x goes out of scope

    {
        int x = 10;  // OK — previous x is completely gone
    }
}
```

#### Rule 5: Local variables CAN shadow instance/static fields

```java
public class Demo {
    int x = 100;

    void method() {
        int x = 200;               // shadows instance field
        System.out.println(x);     // 200 — local
        System.out.println(this.x); // 100 — instance field
    }
}
```

Unlike the case with two locals in nested blocks, a local variable IS allowed to shadow a field. Access the field via `this.fieldName` or `ClassName.fieldName` for statics.

---

### Scope in Control Flow Structures

#### `if` / `else` Blocks

```java
void method(boolean flag) {
    if (flag) {
        int x = 10;
        System.out.println(x);   // OK
    } else {
        int x = 20;              // OK — different block, no conflict
        System.out.println(x);   // OK
    }
    // System.out.println(x);    // COMPILE ERROR — x out of scope
}
```

Note: you can declare `x` in both `if` and `else` with the same name because they are separate, non-overlapping blocks.

---

#### `for` Loop

```java
void method() {
    for (int i = 0; i < 5; i++) {
        int temp = i * 2;        // block scope — per iteration
        System.out.println(temp);
    }
    // System.out.println(i);    // COMPILE ERROR — i out of scope
    // System.out.println(temp); // COMPILE ERROR — temp out of scope
}
```

The loop variable (`i`) is scoped to the entire `for` construct — including the condition, update, and body. It is NOT accessible outside the `for`.

Can you declare the same variable name in a nested inner block of a for loop?

```java
for (int i = 0; i < 5; i++) {
    // int i = 10;   // COMPILE ERROR — i already in scope
}
```

#### `while` Loop

```java
void method() {
    int count = 0;
    while (count < 5) {
        int doubled = count * 2;  // block scope — inside while
        count++;
    }
    System.out.println(count);    // OK — count declared outside while
    // System.out.println(doubled); // COMPILE ERROR
}
```

#### `switch` Statement — Special Case

The entire `switch` body is **one block**. Variables declared in one case are in scope for all subsequent cases:

```java
void method(int x) {
    switch (x) {
        case 1:
            int y = 10;      // y in scope for entire switch
            break;
        case 2:
            y = 20;          // OK — y still in scope (but might not be initialized!)
            System.out.println(y);
            break;
        case 3:
            System.out.println(y);  // Might be compile error — y possibly uninitialized
    }
}
```

> **Exam trap:** In the `switch` above, if execution starts at `case 2` or `case 3`, `y` was never initialized. The compiler detects this and may reject it. The exact behavior depends on whether the compiler can determine if `y` is definitely assigned.

---

### Effectively Final — The Lambda Scope Rule

This is one of the most important scope-related concepts for the OCP exam.

#### The Rule

Lambda expressions and anonymous classes can **capture** local variables from their enclosing scope, but only if those variables are **final or effectively final**.

**Effectively final** means: a variable that is never reassigned after its initial assignment, even without the `final` keyword.

```java
void method() {
    int x = 10;          // effectively final — never reassigned
    Runnable r = () -> System.out.println(x);  // OK — captured
    r.run();
}
```

```java
void method() {
    int x = 10;
    x = 20;              // x is reassigned — NO LONGER effectively final
    Runnable r = () -> System.out.println(x);  // COMPILE ERROR
}
```

```java
void method() {
    int x = 10;
    Runnable r = () -> System.out.println(x);  // OK — x not yet reassigned
    x = 20;              // COMPILE ERROR — x is captured, cannot be reassigned
}
```

#### Why This Restriction Exists

Lambda expressions may execute on a different thread or at a later point in time. If the captured variable could change, the lambda would see inconsistent values. Making the variable effectively final guarantees the lambda always sees the same value.

#### What "Effectively Final" Covers

- Simple local variables that are never reassigned
- Parameters that are never reassigned inside the method
- Variables assigned in one branch and used — if the compiler can verify single assignment

```java
void method(String param) {           // param is effectively final if not reassigned
    Runnable r = () -> System.out.println(param);  // OK
}

void method2(String param) {
    param = "new value";              // param reassigned — NOT effectively final
    Runnable r = () -> System.out.println(param);  // COMPILE ERROR
}
```

#### Effectively Final with Loops

```java
void method() {
    for (int i = 0; i < 5; i++) {
        int copy = i;               // copy is effectively final per iteration
        Runnable r = () -> System.out.println(copy);  // OK
        // Runnable r2 = () -> System.out.println(i); // COMPILE ERROR — i changes
    }
}
```

`i` is not effectively final (it changes each iteration). `copy` is effectively final (assigned once per iteration, never changed).

---

### Variable Lifetime vs Scope

These are related but distinct:

| Concept | Definition | Controlled By |
|---|---|---|
| **Scope** | Where a variable can be named/referenced | Compiler (static) |
| **Lifetime** | How long the variable's storage exists | JVM (runtime) |

For local primitives: scope and lifetime effectively coincide — the variable exists from declaration to end of block.

For local references: the reference variable's lifetime ends with the block, but the **object it points to** may live longer (if other references to it exist).

```java
void method() {
    {
        String s = "hello";     // s in scope here
        // some other code stores s somewhere
    }   // s reference goes out of scope (lifetime ends)
    // the String "hello" object may still be alive if referenced elsewhere
}
```

---

### Class Loading and Static Variable Lifetime

Static variables have a lifetime tied to the class, not any object:

1. Class is first referenced → class loader loads it → static initializers run → static variables created
2. JVM runs
3. JVM shuts down → static variables destroyed

For most applications, static variables live for the **entire duration of program execution**.

---

### Instance Variable Lifetime

Instance variables live from object creation to garbage collection eligibility:

```java
void method() {
    Person p = new Person("Alice");  // Person object created, fields initialized
    p = null;                        // reference dropped — Person eligible for GC
    // Person object's fields (name, age) will be destroyed by GC
}
```

---

### Scope in `try-catch-finally`

Each clause has its own block scope:

```java
void method() {
    try {
        int x = 10;
        System.out.println(x);   // OK
    } catch (Exception e) {
        System.out.println(e);   // OK — e in scope inside catch
        // System.out.println(x); // COMPILE ERROR — x not in scope here
    } finally {
        // System.out.println(x); // COMPILE ERROR
        // System.out.println(e); // COMPILE ERROR
    }
}
```

> The exception variable `e` is scoped to its `catch` block only. It's not accessible in `finally` or other `catch` blocks.

---

### Multi-catch and Variable Scope

```java
void method() {
    try {
        // risky code
    } catch (IOException | SQLException e) {
        // e is effectively final in multi-catch — cannot be reassigned
        // e = new IOException(); // COMPILE ERROR
        System.out.println(e.getMessage());
    }
}
```

In a multi-catch, the exception variable is **implicitly final** — you cannot reassign it.

---

## 2. Code Examples

### Example 1 — Basic Scope Hierarchy

```java
public class ScopeHierarchy {
    static int staticVar = 1;    // class scope
    int instanceVar = 2;         // instance scope

    void method() {
        int localVar = 3;        // method scope

        if (true) {
            int blockVar = 4;    // block scope
            System.out.println(staticVar);   // 1 — OK
            System.out.println(instanceVar); // 2 — OK
            System.out.println(localVar);    // 3 — OK
            System.out.println(blockVar);    // 4 — OK
        }

        System.out.println(staticVar);   // OK
        System.out.println(instanceVar); // OK
        System.out.println(localVar);    // OK
        // System.out.println(blockVar); // COMPILE ERROR
    }
}
```

---

### Example 2 — Shadowing Fields with Locals

```java
public class Shadowing {
    int value = 100;
    static int count = 50;

    void demonstrate() {
        int value = 200;              // shadows instance field
        int count = 75;               // shadows static field

        System.out.println(value);           // 200 — local
        System.out.println(this.value);      // 100 — instance field
        System.out.println(count);           // 75 — local
        System.out.println(Shadowing.count); // 50 — static field
    }
}
```

---

### Example 3 — Loop Variable Scope

```java
public class LoopScope {
    public static void main(String[] args) {
        // for loop — i scoped to the for construct
        for (int i = 0; i < 3; i++) {
            System.out.print(i + " ");
        }
        // System.out.println(i);  // COMPILE ERROR

        // while loop — count scoped outside
        int count = 0;
        while (count < 3) {
            System.out.print(count + " ");
            count++;
        }
        System.out.println(count);  // 3 — still in scope

        // for-each
        int[] arr = {10, 20, 30};
        for (int num : arr) {
            System.out.print(num + " ");
        }
        // System.out.println(num);  // COMPILE ERROR
    }
}
```

---

### Example 4 — Effectively Final Capture

```java
import java.util.function.Supplier;

public class EffectivelyFinal {
    public static void main(String[] args) {
        int a = 10;               // effectively final
        int b = 20;
        b = 30;                   // b reassigned — NOT effectively final

        Supplier<Integer> s1 = () -> a + 5;  // OK
        // Supplier<Integer> s2 = () -> b + 5;  // COMPILE ERROR

        System.out.println(s1.get());  // 15
    }
}
```

---

### Example 5 — Effectively Final in Loop

```java
import java.util.*;
import java.util.function.Supplier;

public class LoopCapture {
    public static void main(String[] args) {
        List<Supplier<Integer>> suppliers = new ArrayList<>();

        for (int i = 0; i < 5; i++) {
            int copy = i;             // effectively final per iteration
            suppliers.add(() -> copy); // OK — captures copy
            // suppliers.add(() -> i); // COMPILE ERROR — i not effectively final
        }

        for (var s : suppliers) {
            System.out.print(s.get() + " ");  // 0 1 2 3 4
        }
    }
}
```

---

### Example 6 — Switch Scope

```java
public class SwitchScopeDemo {
    public static void main(String[] args) {
        int x = 1;

        switch (x) {
            case 1:
                int result = 100;
                System.out.println(result);
                break;
            case 2:
                result = 200;          // same result — still in scope
                System.out.println(result);
                break;
            case 3:
                // using result here — might not be initialized
                // System.out.println(result);  // might COMPILE ERROR
                result = 300;          // OK — assign first
                System.out.println(result);
                break;
        }
    }
}
```

---

### Example 7 — try-catch Scope

```java
public class TryCatchScope {
    public static void main(String[] args) {
        try {
            int x = Integer.parseInt("abc");  // throws NumberFormatException
            System.out.println(x);            // never reached
        } catch (NumberFormatException e) {
            System.out.println("Caught: " + e.getMessage());
            // x is NOT accessible here
        } finally {
            // neither x nor e is accessible here
            System.out.println("Finally");
        }
    }
}
```

---

### Example 8 — Non-Overlapping Scopes Reuse Names

```java
public class NonOverlapping {
    public static void main(String[] args) {
        // Block 1
        {
            int x = 10;
            System.out.println(x);  // 10
        }  // x gone

        // Block 2 — same name reused
        {
            int x = 20;             // OK — previous x gone
            System.out.println(x);  // 20
        }

        // Same in if-else
        boolean flag = true;
        if (flag) {
            int result = 1;
            System.out.println(result);
        } else {
            int result = 2;         // OK — different block
            System.out.println(result);
        }
    }
}
```

---

### Example 9 — Parameter Capture

```java
import java.util.function.Predicate;

public class ParamCapture {
    static Predicate<String> makeChecker(String prefix) {
        // prefix is effectively final (never reassigned)
        return s -> s.startsWith(prefix);  // OK
    }

    static Predicate<String> makeChecker2(String prefix) {
        prefix = prefix.toLowerCase();     // reassigned — NOT effectively final
        // return s -> s.startsWith(prefix); // COMPILE ERROR
        String finalPrefix = prefix;       // workaround — new effectively final var
        return s -> s.startsWith(finalPrefix);  // OK
    }

    public static void main(String[] args) {
        var checker = makeChecker("Hello");
        System.out.println(checker.test("Hello World"));  // true
        System.out.println(checker.test("Goodbye"));      // false
    }
}
```

---

## 3. Exam-Style Questions

---

**Q1 — MCQ**
```java
public class Test {
    public static void main(String[] args) {
        int x = 1;
        {
            int x = 2;    // Line A
            System.out.println(x);
        }
    }
}
```
What is the result?

- A. Prints `2`
- B. Prints `1` then `2`
- C. Compile error at Line A
- D. Runtime error

**Answer: C**
You cannot declare a local variable with the same name as one in an enclosing scope. Line A causes a compile error.

---

**Q2 — MCQ**
```java
public class Test {
    int x = 10;

    void method() {
        int x = 20;
        System.out.println(x);
    }

    public static void main(String[] args) {
        new Test().method();
    }
}
```
What is the output?

- A. `10`
- B. `20`
- C. Compile error
- D. `10` then `20`

**Answer: B**
Local `x = 20` shadows instance field `x = 10`. `System.out.println(x)` uses local. No error — shadowing field with local is allowed.

---

**Q3 — Select All That Apply**
Which variables are accessible at the point marked `// HERE`?

```java
public class Test {
    static int a = 1;
    int b = 2;

    void method() {
        int c = 3;
        for (int d = 0; d < 1; d++) {
            int e = 5;
            // HERE
        }
    }
}
```

- A. `a`
- B. `b`
- C. `c`
- D. `d`
- E. `e`

**Answer: A, B, C, D, E**
All variables are accessible at `// HERE`. Static field `a`, instance field `b`, local `c`, loop var `d`, and block var `e` are all in scope inside the for loop body.

---

**Q4 — MCQ**
```java
public class Test {
    public static void main(String[] args) {
        int x = 10;
        x = 20;
        int y = x;
        Runnable r = () -> System.out.println(x);  // Line A
        r.run();
    }
}
```
What is the result?

- A. Prints `10`
- B. Prints `20`
- C. Compile error at Line A — `x` is not effectively final
- D. Runtime error

**Answer: C**
`x` is reassigned (`x = 20`) so it is NOT effectively final. The lambda at Line A cannot capture it.

---

**Q5 — MCQ**
```java
public class Test {
    public static void main(String[] args) {
        for (int i = 0; i < 3; i++) {
            System.out.print(i + " ");
        }
        System.out.println(i);  // Line A
    }
}
```
What is the result?

- A. `0 1 2 3`
- B. `0 1 2 ` then compile error
- C. Compile error only — program doesn't run
- D. `0 1 2 2`

**Answer: C**
Line A references `i` which is out of scope after the `for` loop. This is detected at compile time — the program never runs.

---

**Q6 — MCQ**
```java
public class Test {
    public static void main(String[] args) {
        try {
            int x = 1 / 0;
        } catch (ArithmeticException e) {
            System.out.println("Caught");
        } finally {
            System.out.println(e);   // Line A
        }
    }
}
```
What is the result?

- A. Prints `Caught` then the exception
- B. Prints `Caught` then `null`
- C. Compile error at Line A
- D. Runtime error

**Answer: C**
`e` is scoped to the `catch` block only. It is not accessible in `finally`. Compile error at Line A.

---

**Q7 — MCQ**
```java
import java.util.function.Supplier;

public class Test {
    public static void main(String[] args) {
        int x = 5;
        Supplier<Integer> s = () -> x * 2;
        x = 10;           // Line A
        System.out.println(s.get());
    }
}
```
What is the result?

- A. `10`
- B. `20`
- C. Compile error at Line A
- D. Runtime error

**Answer: C**
`x` is captured by the lambda. The compiler sees that `x` is reassigned at Line A, making it NOT effectively final. Compile error — you cannot reassign a variable after it's captured by a lambda.

---

**Q8 — Select All That Apply**
Which of the following are true about effectively final variables? (Choose all that apply)

- A. They must be declared with the `final` keyword
- B. They can be captured by lambda expressions
- C. They can be captured by anonymous inner classes
- D. They must never be reassigned after initial assignment
- E. Reassigning before the lambda declaration is OK as long as there's no reassignment after

**Answer: B, C, D**
A is false — `final` keyword not required, just no reassignment. B and C are correct. D is correct — any reassignment disqualifies it. E is false — any reassignment anywhere makes the variable not effectively final, even if before the lambda.

---

**Q9 — MCQ**
```java
public class Test {
    public static void main(String[] args) {
        {
            int x = 10;
        }
        {
            int x = 20;    // Line A
            System.out.println(x);
        }
    }
}
```
What is the result?

- A. Compile error at Line A
- B. Prints `20`
- C. Prints `10`
- D. Prints `10` then `20`

**Answer: B**
The two blocks are non-overlapping. The first `x` is out of scope by the time the second block starts. Reusing the name `x` in a separate non-overlapping block is perfectly legal.

---

**Q10 — MCQ**
```java
public class Test {
    public static void main(String[] args) {
        int x = 0;
        switch (x) {
            case 0:
                int y;
            case 1:
                y = 100;
                System.out.println(y);
                break;
        }
    }
}
```
What is the result?

- A. Compile error — `y` not in scope in case 1
- B. Prints `100`
- C. Compile error — `y` might not be initialized
- D. Prints `0`

**Answer: B**
`x = 0` matches `case 0`. Since there's no `break` in `case 0`, fall-through reaches `case 1`. `y` is declared in `case 0` (same switch scope), assigned `100` in `case 1`, and printed. Since execution DOES pass through case 0 (where `y` is declared) before case 1, `y` is in scope and the path guarantees the assignment. Result: `100`.

---

## 4. Trick Analysis

### Trap 1: Local variable shadowing local — illegal
```java
int x = 5;
{ int x = 10; }  // COMPILE ERROR
```
You cannot shadow a local with another local in a nested block. You CAN shadow a **field** with a local — that's allowed. The exam presents both scenarios and tests whether you know the difference.

---

### Trap 2: Reassignment before lambda still kills effective finality
```java
int x = 5;
x = 10;                              // reassigned here
Runnable r = () -> System.out.println(x);  // COMPILE ERROR
```
Even though `x` is not reassigned AFTER the lambda, the earlier reassignment makes `x` not effectively final. The compiler considers the **entire scope** of the variable.

---

### Trap 3: Reassignment after lambda declaration kills effective finality
```java
int x = 5;
Runnable r = () -> System.out.println(x);  // seemingly OK
x = 10;   // COMPILE ERROR — x is captured, now tried to reassign
```
The lambda captures `x`. Then assigning `x = 10` makes the variable NOT effectively final retroactively. The compiler errors on the reassignment line.

---

### Trap 4: Loop variable `i` not effectively final
```java
for (int i = 0; i < 5; i++) {
    Runnable r = () -> System.out.println(i);  // COMPILE ERROR
}
```
`i` changes every iteration — it's not effectively final. The classic solution is `int copy = i;` inside the loop.

---

### Trap 5: Exception variable scope in try-catch
The `e` in `catch (Exception e)` is scoped to that `catch` block only. It's not accessible in `finally`, other `catch` blocks, or after the `try-catch` structure. The exam puts `e` in `finally` and asks if it compiles — it doesn't.

---

### Trap 6: `for` loop variable out of scope immediately after loop
`for (int i = 0; ...)` — `i` dies the moment the closing `}` of the for loop is reached. Any reference to `i` after the loop is a compile error. The exam places code after the loop using `i` and tests whether candidates know it's out of scope.

---

### Trap 7: Switch is one big scope
Unlike `if-else` where each block is separate, the entire `switch` body is one block. A variable declared in `case 1` is in scope in `case 2`, `case 3`, etc. But it might not be **initialized** in those cases, leading to "might not be initialized" compile errors.

---

### Trap 8: Non-overlapping scopes can reuse names
```java
{ int x = 1; }
{ int x = 2; }  // OK
```
This compiles fine. Many candidates apply the "can't redeclare in same scope" rule too broadly. The rule only applies when the earlier declaration is still in scope.

---

## 5. Summary Sheet

### The Four Scopes

| Scope | Created | Destroyed | Example |
|---|---|---|---|
| Class (static) | Class loaded | JVM shutdown | `static int x` |
| Instance | Object created (`new`) | Object GC'd | `int x` (field) |
| Method | Method called | Method returns | `int x` in method |
| Block | Block entered | Block exits `}` | `int x` in `if` |

---

### Scope Access Rules

| Rule | Legal? |
|---|---|
| Inner block accesses outer variable | ✅ |
| Outer block accesses inner variable | ❌ |
| Local shadows field | ✅ |
| Local shadows local in enclosing scope | ❌ |
| Same name in non-overlapping blocks | ✅ |
| Variable used before declaration | ❌ |

---

### Effectively Final Rules

| Scenario | Effectively Final? |
|---|---|
| Declared and never reassigned | ✅ |
| Declared with `final` keyword | ✅ |
| Reassigned once after declaration | ❌ |
| Reassigned before lambda — then used | ❌ |
| Assigned after lambda — then reassigned | ❌ |
| Loop variable (`for (int i...)`) | ❌ |
| Loop body copy (`int copy = i`) | ✅ (per iteration) |
| Method parameter never reassigned | ✅ |
| Method parameter reassigned inside | ❌ |
| Multi-catch exception variable | ✅ (implicitly final) |

---

### Special Scope Cases

| Structure | Scope Notes |
|---|---|
| `for` loop variable | Scoped to entire `for` construct — not accessible after |
| `catch` variable `e` | Scoped to that catch block only |
| `switch` body | One unified scope — variables span all cases |
| `if` / `else` | Separate blocks — can reuse names |
| Lambda capture | Must be effectively final |
| Anonymous class capture | Must be effectively final |

---

### Key Rules to Memorize

| Rule | Detail |
|---|---|
| No forward reference | Can't use variable before its declaration |
| No local shadowing local | Can't redeclare local in nested scope |
| CAN shadow field | Local can have same name as field |
| `this.x` | Accesses field when shadowed by local |
| `ClassName.x` | Accesses static field when shadowed |
| Effectively final | No reassignment anywhere in scope |
| Lambda capture timing | Compiler checks entire scope, not just capture point |

---
