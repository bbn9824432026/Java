# Topic 1.3 — Main Method & Program Entry Points

---

## 1. Conceptual Explanation

### What is the Entry Point?

When you run a Java program with `java ClassName`, the JVM needs to know **where to start execution**. It looks for a specific method signature — the **main method**. If it doesn't find it in the specified class, you get a runtime error, not a compile error.

This is an important distinction: **the absence of a valid main method is a runtime error**, not a compile-time error. The class compiles fine — the JVM just can't find the entry point when you try to run it.

---

### The Standard Main Method Signature

```java
public static void main(String[] args) {
    // entry point
}
```

Every part of this signature has a reason:

**`public`** — The JVM is an external caller. The method must be accessible from outside the class, so it must be `public`.

**`static`** — The JVM calls `main` without creating an instance of the class. No object exists yet when the program starts, so the method must belong to the class itself — hence `static`.

**`void`** — The JVM doesn't use a return value from `main`. Any value returned would be discarded. The program's exit code is communicated via `System.exit(int)` instead.

**`main`** — This is the name the JVM looks for by convention. It is not a keyword — you can have other methods named `main` with different signatures — but the JVM specifically looks for this exact name.

**`String[] args`** — Command-line arguments passed to the program. This is always a `String` array. It can be empty but never `null` when provided by the JVM.

---

### Valid Variations of the Main Method

The exam tests which variations of `main` are accepted by the JVM. Several variations are legal:

```java
// Standard form
public static void main(String[] args) { }

// Variation 1: args can be named anything
public static void main(String[] parameters) { }

// Variation 2: array brackets can go after the variable name
public static void main(String args[]) { }

// Variation 3: varargs is allowed
public static void main(String... args) { }

// Variation 4: modifiers can be in different order
static public void main(String[] args) { }

// Variation 5: final modifier on args is allowed
public static void main(final String[] args) { }
```

All five variations above are **valid entry points** that the JVM will accept.

---

### Invalid Main Method Signatures

```java
// Missing public
static void main(String[] args) { }          // Runtime error (not compile error)

// Missing static
public void main(String[] args) { }          // Runtime error

// Wrong return type
public static int main(String[] args) { }    // Runtime error

// Wrong parameter type
public static void main(String args) { }     // Runtime error

// No parameter
public static void main() { }               // Runtime error
```

> **Critical exam point:** All of the above **compile perfectly fine**. They are valid Java methods. The JVM simply won't recognize them as entry points. The error is at **runtime**, not compile time.

The runtime error you receive:
```
Error: Main method not found in class Demo,
please define the main method as:
   public static void main(String[] args)
```

---

### Overloading `main`

You can **overload** the `main` method. The JVM will still call the one with `String[]` or `String...` parameter:

```java
public class Overloaded {
    public static void main(String[] args) {
        System.out.println("JVM calls this one");
        main(42);
    }

    public static void main(int x) {
        System.out.println("Called manually: " + x);
    }
}
```

Output:
```
JVM calls this one
Called manually: 42
```

The JVM specifically seeks `main(String[])`. Other `main` overloads are regular static methods you call yourself.

---

### Command-Line Arguments

Command-line arguments are passed as strings regardless of what they look like:

```bash
java MyApp 42 hello true
```

Inside the program:
```java
public static void main(String[] args) {
    System.out.println(args[0]);  // "42" — a String, not an int
    System.out.println(args[1]);  // "hello"
    System.out.println(args[2]);  // "true" — a String, not a boolean

    int num = Integer.parseInt(args[0]);  // must parse explicitly
}
```

**`args.length`** tells you how many arguments were passed.

If you access `args[0]` when no arguments were provided:
```
Exception in thread "main" java.lang.ArrayIndexOutOfBoundsException: Index 0 out of bounds for length 0
```

This is a **runtime** exception, not a compile error.

---

### `main` in Java 21+ (Instance Main Methods — Preview)

Java 21 introduced **instance main methods** as a preview feature (not on OCP Java 17 exam, but worth knowing the boundary). For OCP 17, the standard `public static void main(String[] args)` is what's tested.

---

### Single-File Programs and `main` (Java 11+)

Recap from Topic 1.1: when you run `java Hello.java`, the JVM looks for `main` in the **first class** in the file. This is a source-launcher feature and follows the same signature rules.

---

### `System.exit()`

The JVM shuts down when:
1. The `main` method returns normally
2. `System.exit(int status)` is called
3. An uncaught exception propagates out of `main`

`System.exit(0)` indicates normal termination. Any non-zero value indicates abnormal termination. This is a convention inherited from Unix/C.

```java
public static void main(String[] args) {
    System.out.println("Before exit");
    System.exit(0);
    System.out.println("Never printed");  // unreachable, but compiles
}
```

> **Exam note:** Code after `System.exit()` is **unreachable** but does **not** cause a compile error in Java (unlike some other unreachable code scenarios). The compiler doesn't track `System.exit()` as a definite exit point.

---

### The `main` Method and Inheritance

`main` can be inherited. If a subclass doesn't define its own `main`, and you run `java SubClass`, the JVM will find and run the inherited `main` from the superclass:

```java
public class Parent {
    public static void main(String[] args) {
        System.out.println("Parent main");
    }
}

public class Child extends Parent {
    // no main defined
}
```

```bash
java Child   # prints: Parent main
```

This works because `main` is static and static methods are inherited (though not overridden — they're hidden). The JVM finds `main` in the class hierarchy.

---

## 2. Code Examples

### Example 1 — Valid Variations

```java
public class ValidMain {
    // All of these are valid entry points

    // Variation A — standard
    public static void main(String[] args) {
        System.out.println("Standard");
    }
}
```

```java
public class ValidMain2 {
    // varargs variation
    public static void main(String... args) {
        System.out.println("Varargs: " + args.length);
    }
}
```

```java
public class ValidMain3 {
    // reversed modifier order
    static public void main(String[] args) {
        System.out.println("Reversed modifiers");
    }
}
```

All three compile and run correctly.

---

### Example 2 — Accessing Command-Line Arguments

```java
public class Greeter {
    public static void main(String[] args) {
        if (args.length == 0) {
            System.out.println("Hello, World!");
        } else {
            System.out.println("Hello, " + args[0] + "!");
        }
    }
}
```

```bash
java Greeter           # Hello, World!
java Greeter Alice     # Hello, Alice!
```

---

### Example 3 — Parsing Arguments

```java
public class Calculator {
    public static void main(String[] args) {
        // args[0] and args[1] are Strings even if they look like numbers
        int a = Integer.parseInt(args[0]);
        int b = Integer.parseInt(args[1]);
        System.out.println("Sum: " + (a + b));
    }
}
```

```bash
java Calculator 10 20   # Sum: 30
java Calculator         # RuntimeException: ArrayIndexOutOfBoundsException
java Calculator abc 5   # RuntimeException: NumberFormatException
```

---

### Example 4 — Overloading main

```java
public class MainOverload {
    public static void main(String[] args) {
        System.out.println("Entry point, args: " + args.length);
        main(100);
        main("hello");
    }

    public static void main(int x) {
        System.out.println("int main: " + x);
    }

    public static void main(String s) {
        System.out.println("String main: " + s);
    }
}
```

Output when run with `java MainOverload`:
```
Entry point, args: 0
int main: 100
String main: hello
```

---

### Example 5 — Tricky: Non-public main compiles but doesn't run as entry

```java
public class NoPublic {
    static void main(String[] args) {     // package-private, not public
        System.out.println("Won't run as entry point");
    }
}
```

This **compiles** without errors. But:
```bash
java NoPublic
# Error: Main method not found in class NoPublic
```

---

### Example 6 — System.exit() behavior

```java
public class ExitDemo {
    public static void main(String[] args) {
        System.out.println("Line 1");
        System.exit(0);
        System.out.println("Line 2");  // never executes, but compiles
    }
}
```

Output:
```
Line 1
```

---

### Example 7 — args is never null but can be empty

```java
public class ArgsCheck {
    public static void main(String[] args) {
        // args is NEVER null when called by JVM
        // but it CAN be an empty array
        System.out.println(args == null);   // always false
        System.out.println(args.length);    // 0 if no args passed
    }
}
```

---

## 3. Exam-Style Questions

---

**Q1 — MCQ**
Which of the following `main` method signatures will be recognized by the JVM as the program entry point?

- A. `public static void main(String args)`
- B. `public static void main(String[] args)`
- C. `public static int main(String[] args)`
- D. `public void main(String[] args)`

**Answer: B**
A has wrong parameter type (missing `[]`). C has wrong return type. D is missing `static`.

---

**Q2 — Select All That Apply**
Which of the following are valid entry-point signatures for the JVM? (Choose all that apply)

- A. `public static void main(String[] args)`
- B. `static public void main(String[] args)`
- C. `public static void main(String... args)`
- D. `public static void main(String args[])`
- E. `public static void main()`
- F. `public static final void main(String[] args)`

**Answer: A, B, C, D, F**
E has no parameters so it won't be found by the JVM. All others are valid — modifier order doesn't matter (B), varargs works (C), bracket position doesn't matter (D), `final` is allowed (F).

Wait — F: `final` on a `void` return type is syntactically invalid. Let me correct: `final` can be placed on the method in the sense of `public static final void main(String[] args)` — this IS valid. A method can be `final`. So F is valid.

E is the only invalid one.

---

**Q3 — MCQ**
What happens when you try to run a class that has no `main` method?

- A. Compile error
- B. Runtime error
- C. The program runs with no output
- D. The JVM calls the constructor instead

**Answer: B**
The class compiles fine. The JVM throws an error at runtime when it cannot find the entry-point method.

---

**Q4 — Code Output**
```java
public class Test {
    public static void main(String[] args) {
        System.out.println(args.length);
    }
}
```
Run with: `java Test one two three`

- A. 0
- B. 3
- C. 4
- D. NullPointerException

**Answer: B**
Three arguments are passed: `"one"`, `"two"`, `"three"`. `args.length` is 3.

---

**Q5 — MCQ**
```java
public class Test {
    public static void main(String[] args) {
        System.out.println(args[0]);
    }
}
```
Run with: `java Test`

What is the result?

- A. Compile error
- B. `null`
- C. `ArrayIndexOutOfBoundsException`
- D. `IndexOutOfBoundsException`

**Answer: C**
No arguments are passed, so `args` is an empty array. Accessing index `0` throws `ArrayIndexOutOfBoundsException` at runtime.

---

**Q6 — MCQ**
```java
public class Parent {
    public static void main(String[] args) {
        System.out.println("Parent");
    }
}

public class Child extends Parent { }
```
What is the result of running `java Child`?

- A. Compile error
- B. Runtime error — no main in Child
- C. Prints `Parent`
- D. Prints nothing

**Answer: C**
`Child` inherits `main` from `Parent`. The JVM finds it and runs it.

---

**Q7 — MCQ**
```java
public class Demo {
    static void main(String[] args) {    // package-private
        System.out.println("Hello");
    }
}
```
What is the result of `javac Demo.java` followed by `java Demo`?

- A. Compile error
- B. Prints `Hello`
- C. Runtime error — main method not found
- D. Runtime error — access violation

**Answer: C**
The code compiles. The JVM requires `main` to be `public`. Since it's package-private, the JVM won't recognize it as an entry point.

---

**Q8 — Drag-and-Drop Style**
Arrange these elements to form a valid `main` method signature:

Elements: `void`, `public`, `main`, `static`, `(String[] args)`

**Answer:** `public static void main(String[] args)`
Note: `static public void main(String[] args)` is also valid — modifier order is flexible.

---

## 4. Trick Analysis

### Trap 1: Missing `public` or `static` compiles fine
The most tested trap. Oracle writes a `main` method missing `public` or `static` and asks "what happens?" Many candidates say compile error. The answer is always **runtime error** — it compiles, the JVM just won't run it as an entry point.

---

### Trap 2: `int` return type
`public static int main(String[] args)` is a perfectly legal Java method. It compiles. It will not be called by the JVM. This is a runtime-only distinction.

---

### Trap 3: `String args` vs `String[] args`
`String args` (no brackets) is a valid method parameter but won't be recognized as main. `String[] args` and `String... args` both work. `String args[]` (C-style brackets) also works — the brackets can go after the type or after the variable name.

---

### Trap 4: `args` is always non-null
The JVM always provides an array object, even if no arguments are given. `args` will be an empty array (`length == 0`), never `null`. Accessing `args[0]` without checking length causes `ArrayIndexOutOfBoundsException`, not `NullPointerException`.

---

### Trap 5: Code after `System.exit()` compiles
Unlike some genuinely unreachable code (after `return`, `throw`, infinite loop), code after `System.exit()` compiles without warning. The compiler doesn't flag it as unreachable because `System.exit()` is a regular method call — the compiler doesn't give it special semantic meaning. The code simply never executes at runtime.

---

### Trap 6: `main` with no parameters is just an overload
`public static void main()` compiles fine. It's just another method. The JVM won't call it as an entry point. If you have this AND `main(String[] args)`, the JVM calls the one with the `String[]` parameter.

---

## 5. Summary Sheet

### Valid Main Signatures

| Signature | Valid Entry Point? |
|---|---|
| `public static void main(String[] args)` | ✅ |
| `static public void main(String[] args)` | ✅ |
| `public static void main(String... args)` | ✅ |
| `public static void main(String args[])` | ✅ |
| `public static final void main(String[] args)` | ✅ |
| `public static void main(final String[] args)` | ✅ |
| `static void main(String[] args)` | ❌ (runtime error) |
| `public void main(String[] args)` | ❌ (runtime error) |
| `public static int main(String[] args)` | ❌ (runtime error) |
| `public static void main(String args)` | ❌ (runtime error) |
| `public static void main()` | ❌ (runtime error) |

---

### Key Rules

| Rule | Detail |
|---|---|
| Missing main | Runtime error, not compile error |
| `args` nullability | Never `null` — may be empty array |
| `args` types | Always `String[]` — must parse to other types manually |
| Overloading main | Legal — JVM picks `String[]` version |
| Modifier order | Flexible — `static public` = `public static` |
| Inherited main | JVM will find and run inherited `main` |
| `System.exit()` | Terminates JVM; code after it compiles but never runs |
| Single-file launch | `java Hello.java` — first class in file must have valid main |

---

### When Each Error Occurs

| Scenario | Compile Error? | Runtime Error? |
|---|---|---|
| `main` is not `public` | ❌ | ✅ |
| `main` is not `static` | ❌ | ✅ |
| `main` returns `int` | ❌ | ✅ |
| `main` has wrong params | ❌ | ✅ |
| No `main` at all | ❌ | ✅ |
| `args[0]` with no args | ❌ | ✅ (AIOOBE) |
| Syntax error anywhere | ✅ | ❌ |

---
