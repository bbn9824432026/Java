# Topic 1.1 — Understanding the Java Environment
## (JDK, JRE, JVM, Compilation Model)

---

## 1. Conceptual Explanation

### The Big Picture

When you write Java code, it goes through a multi-stage process before it actually runs on your machine. Understanding this pipeline is fundamental — and Oracle tests it in subtle ways.

```
Your .java file
      ↓  (javac — Java Compiler)
   .class file (bytecode)
      ↓  (JVM — Java Virtual Machine)
Native machine execution
```

---

### The Three Core Components

#### JDK — Java Development Kit
The JDK is the **complete development environment**. It contains everything you need to **write, compile, and run** Java programs.

What's inside the JDK:
- `javac` — the compiler (converts `.java` → `.class`)
- `java` — the launcher (runs `.class` files via JVM)
- `jar` — archive tool
- `javadoc` — documentation generator
- `jshell` — REPL (interactive shell, Java 9+)
- `jdeps` — dependency analyzer
- `jlink` — custom runtime image creator
- `jmod` — module packaging tool
- The JRE (included within the JDK)

> **Key exam point:** As of Java 11+, the JRE is **no longer distributed separately**. You use the JDK directly. Oracle removed the standalone JRE download. This is tested.

---

#### JRE — Java Runtime Environment
The JRE is everything needed to **run** (but not develop) Java programs.

What's inside the JRE:
- The JVM
- Core class libraries (`java.lang`, `java.util`, etc.)
- Supporting files

> **Exam trap:** Pre-Java 11, developers could ship just the JRE to end users. Post-Java 11, you use `jlink` to create a **custom minimal runtime image** instead. The concept of a standalone JRE is effectively deprecated.

---

#### JVM — Java Virtual Machine
The JVM is the **engine** that executes bytecode. It is:

- **Platform-specific** — there is a different JVM implementation for Windows, Linux, macOS
- **Bytecode-agnostic** — it doesn't care if the bytecode came from Java, Kotlin, Scala, or Groovy
- **Responsible for:** memory management, garbage collection, JIT compilation, security sandboxing

Key JVM responsibilities tested on the exam:
- Loading classes (Class Loader Subsystem)
- Verifying bytecode
- Executing bytecode (interpreter + JIT compiler)
- Managing the heap and stack
- Running the Garbage Collector

---

### The Compilation Model — Deep Dive

#### Step 1: Writing Source Code
You write a file named `Hello.java`. The filename **must match the public class name** inside it. This is a compile-time rule.

```java
// File: Hello.java
public class Hello {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}
```

**Rule:** One public class per `.java` file. The file name must match the public class name exactly (case-sensitive).

---

#### Step 2: Compilation with `javac`

```bash
javac Hello.java
```

The compiler:
1. Parses the source code
2. Resolves symbols and checks types
3. Performs semantic analysis
4. Generates **bytecode** in a `.class` file

Output: `Hello.class`

The `.class` file contains **platform-independent bytecode**, not machine code. This is the "write once, run anywhere" promise of Java.

**What `javac` catches (compile-time errors):**
- Syntax errors
- Type mismatches
- Missing methods/classes
- Access modifier violations
- Unreachable code (in some cases)
- Missing `return` statements

**What `javac` does NOT catch:**
- `NullPointerException`
- `ArrayIndexOutOfBoundsException`
- `ClassCastException`
- Logic errors

---

#### Step 3: Execution with `java`

```bash
java Hello
```

Notice: **no `.class` extension** when running. You provide the class name, not the filename.

The JVM:
1. Loads `Hello.class` via the **Class Loader**
2. Verifies the bytecode (security check)
3. **Interprets** the bytecode OR **JIT-compiles** hot paths to native code
4. Executes `main(String[] args)`
5. Performs garbage collection as needed
6. Shuts down when `main` returns (or `System.exit()` is called)

---

### JIT — Just-In-Time Compilation

The JVM doesn't just interpret bytecode line by line. It monitors which code paths are executed frequently ("hot spots") and compiles those to **native machine code** at runtime. This is the **HotSpot JVM** behavior.

This is why Java performance improves the longer a program runs — the JIT compiler optimizes over time.

> **Exam relevance:** You won't be asked to write JIT code, but you may be asked conceptual questions about whether Java is compiled, interpreted, or both. The correct answer is **both** — compiled to bytecode by `javac`, then interpreted/JIT-compiled by the JVM.

---

### Compile-Time vs Runtime — Critical Distinction

This distinction is tested **constantly** throughout the entire OCP exam, not just in this topic.

| What happens at | Examples |
|---|---|
| **Compile-time** | Syntax check, type checking, access checks, `final` variable assignment check |
| **Runtime** | Actual method dispatch (polymorphism), casting validation, exception throwing, GC |

**Key mindset for the exam:** When you see a code snippet, always ask:
1. Does this **compile**? (javac errors)
2. If it compiles, does it **run without exception**?
3. If it runs, what is the **output**?

---

### One `.java` File, Multiple Classes

A single `.java` file can contain **multiple classes**, but with strict rules:

```java
// File: Animal.java
class Dog {       // package-private, OK
    void bark() {}
}

class Cat {       // package-private, OK
    void meow() {}
}

// public class Animal { } // There can only be ONE public class
```

- At most **one** `public` class per file
- The `public` class name **must match** the filename
- Each class gets its own `.class` file after compilation
  - `Animal.java` with `Dog` and `Cat` → produces `Dog.class` and `Cat.class`

> **Exam trap:** If you have `class Dog` and `class Cat` in `Animal.java` with no public class, the file compiles fine. You'd run `java Dog` or `java Cat`. This surprises many candidates.

---

### The `--source`, `--target`, and `--release` Flags

```bash
javac --release 17 Hello.java
```

- `--release 17` ensures the code compiles targeting Java 17 APIs and bytecode version
- Replaces the older `-source` and `-target` flags
- Prevents accidentally using APIs unavailable in your target version

---

### Java's "Write Once, Run Anywhere" — The Real Mechanism

Platform independence comes from the fact that:
- `javac` produces **bytecode** (not native code)
- Each platform has its own **JVM implementation** that understands bytecode
- So the **same `.class` file** runs on Windows JVM, Linux JVM, macOS JVM

The JVM itself is **platform-dependent**. The bytecode is **platform-independent**.

This is a common exam distinction.

---

## 2. Code Examples

### Example 1 — Filename Must Match Public Class

```java
// File: MyApp.java
public class MyApp {          // ✅ matches filename
    public static void main(String[] args) {
        System.out.println("Running");
    }
}
```

```java
// File: MyApp.java
public class WrongName {      // ❌ COMPILE ERROR
    // class name doesn't match filename
}
```
**Error:** `class WrongName is public, should be declared in a file named WrongName.java`

---

### Example 2 — Multiple Classes in One File

```java
// File: Runner.java
class Helper {
    static void help() {
        System.out.println("Helping");
    }
}

public class Runner {
    public static void main(String[] args) {
        Helper.help();        // ✅ perfectly legal
    }
}
```

After `javac Runner.java`, you get:
- `Runner.class`
- `Helper.class`

Both `.class` files are produced from one `.java` file.

---

### Example 3 — Running a Class (No .class Extension)

```bash
# Correct
java Runner

# Wrong — common beginner mistake (not exam focus but good to know)
java Runner.class    # Error: Could not find or load main class Runner.class
```

---

### Example 4 — Compile-Time vs Runtime Error Distinction

```java
public class Demo {
    public static void main(String[] args) {
        int x = "hello";    // COMPILE-TIME ERROR: incompatible types
    }
}
```

```java
public class Demo {
    public static void main(String[] args) {
        String s = null;
        System.out.println(s.length());  // Compiles fine, RUNTIME: NullPointerException
    }
}
```

---

### Example 5 — Single-File Source-Code Programs (Java 11+)

Java 11 introduced the ability to run a single `.java` file directly without explicit compilation:

```bash
java Hello.java   # Compiles and runs in one step
```

Rules:
- Only works for **single-file programs**
- The class to run must be the **first class** in the file
- No `javac` step needed — `java` handles it internally

> **Exam trap:** This is different from normal execution. `java Hello` (no extension) runs a compiled `.class`. `java Hello.java` (with extension) compiles and runs in one step. These are different commands with different behaviors.

---

### Example 6 — What `javac` Produces for Inner Structure

```java
// File: Outer.java
public class Outer {
    class Inner { }
    static class StaticNested { }
}
```

Compilation produces:
- `Outer.class`
- `Outer$Inner.class`
- `Outer$StaticNested.class`

The `$` separator is used for nested/inner classes. This is a fact the exam has referenced.

---

## 3. Exam-Style Questions

---

**Q1 — MCQ**
Which of the following tools is included in the JDK but NOT available as a standalone JRE component in Java 17?

- A. `java`
- B. `javac`
- C. The JVM
- D. Core class libraries

**Answer: B**
`javac` is the compiler — a development tool. It is only in the JDK. The JVM and core libraries are part of the runtime (JRE). Note: In Java 17 the standalone JRE no longer exists, but conceptually `javac` was always JDK-only.

---

**Q2 — Code Output / Compilation**
```java
// File: Test.java
public class Animal {
    public static void main(String[] args) {
        System.out.println("Animal");
    }
}
```
What happens when you compile this with `javac Test.java`?

- A. Compiles successfully, produces `Test.class`
- B. Compiles successfully, produces `Animal.class`
- C. Compile error: class name does not match file name
- D. Runtime error when executed

**Answer: C**
The public class `Animal` must be in a file named `Animal.java`. Since the file is `Test.java`, this is a compile error.

---

**Q3 — Select All That Apply**
Which statements about the JVM are correct? (Choose all that apply)

- A. The JVM is platform-independent
- B. The JVM executes bytecode
- C. The JVM can execute code written in languages other than Java
- D. The JVM performs garbage collection
- E. The JVM converts `.java` files to `.class` files

**Answer: B, C, D**

- A is wrong — the JVM is **platform-specific** (there's a different JVM for each OS)
- B is correct — the JVM executes bytecode
- C is correct — Kotlin, Scala, Groovy all compile to JVM bytecode
- D is correct — GC is a core JVM responsibility
- E is wrong — `javac` does that, not the JVM

---

**Q4 — MCQ**
You have the following file:

```java
// File: Zoo.java
class Lion { }
class Tiger { }
class Zoo { }
```

After running `javac Zoo.java`, which `.class` files are produced?

- A. Only `Zoo.class`
- B. `Zoo.class` and `Lion.class` only
- C. `Lion.class`, `Tiger.class`, and `Zoo.class`
- D. Compile error — only one class allowed per file

**Answer: C**
All three classes compile successfully because none are `public`. Each class gets its own `.class` file.

---

**Q5 — MCQ**
Which command correctly runs a compiled Java program whose main class is `com.example.Main`?

- A. `java com/example/Main.class`
- B. `java com/example/Main`
- C. `java com.example.Main`
- D. `javac com.example.Main`

**Answer: C**
You use the **fully qualified class name with dots**, not slashes, and no `.class` extension. `javac` is for compiling, not running.

---

**Q6 — True/False Style MCQ**
Java is best described as:

- A. A purely compiled language
- B. A purely interpreted language
- C. Both compiled and interpreted
- D. A language that compiles directly to machine code

**Answer: C**
`javac` compiles source to bytecode (compiled step). The JVM interprets/JIT-compiles bytecode to native code at runtime (interpreted/compiled step).

---

**Q7 — Tricky Scenario**
```java
// File: Hello.java
class Hello {
    public static void main(String[] args) {
        System.out.println("Hi");
    }
}
```
Which commands will successfully compile AND run this program?

- A. `javac Hello.java` then `java Hello`
- B. `javac Hello.java` then `java Hello.class`
- C. `java Hello.java`
- D. Both A and C

**Answer: D**
- A works: standard compile then run
- C works: Java 11+ single-file execution (`java Hello.java`)
- B fails: `.class` extension causes an error

---

**Q8 — Select All That Apply**
Which of the following are detected at **compile time**? (Choose all that apply)

- A. `NullPointerException`
- B. Calling a method that doesn't exist on a class
- C. `ClassCastException`
- D. Assigning a `String` to an `int` variable
- E. `StackOverflowError`

**Answer: B, D**
- A, C, E are runtime issues
- B and D are caught by `javac`

---

## 4. Trick Analysis

### Trap 1: "The JVM is platform-independent"
This is **false** and a classic exam trick. The **bytecode** is platform-independent. The JVM itself must be a native application for each OS. Oracle writes a Windows JVM, a Linux JVM, etc. The exam will present this statement as a true/false or in an MCQ, and many candidates choose it because they associate "platform independence" with Java generically.

---

### Trap 2: "java Hello.class"
Many candidates assume you add `.class` when running. You don't. `java Hello` is correct. `java Hello.class` throws:
```
Error: Could not find or load main class Hello.class
Caused by: java.lang.ClassNotFoundException: Hello.class
```
The exam uses this to catch candidates who confuse the compile and run steps.

---

### Trap 3: Multiple public classes in one file
If you try to declare two `public` classes in one `.java` file, it's always a compile error — even if both class names are "reasonable." Only one public class, and it must match the filename.

---

### Trap 4: Non-public class doesn't need to match filename
```java
// File: Zoo.java — compiles fine!
class Lion { }   // not public, no filename constraint
```
Candidates assume the filename rule applies to ALL classes. It only applies to `public` classes.

---

### Trap 5: JRE availability in Java 17
Questions framed as "which tool do you need to distribute to end users in Java 17" — the answer is a custom runtime image via `jlink`, not a standalone JRE (which no longer exists as a separate download).

---

### Trap 6: `java Hello.java` vs `java Hello`
Both can work, but they are doing different things:
- `java Hello` → runs the already-compiled `Hello.class`
- `java Hello.java` → compiles and runs in one step (Java 11+, single-file programs only)

If `Hello.class` doesn't exist and you run `java Hello`, you get a `ClassNotFoundException`. If you run `java Hello.java`, it works fine because it compiles first.

---

## 5. Summary Sheet

### Key Rules to Memorize

| Rule | Detail |
|---|---|
| File name must match | The **public class** name must exactly match the `.java` filename (case-sensitive) |
| One public class per file | Only one `public` class allowed per `.java` file |
| Multiple non-public classes | Allowed in one file — each gets its own `.class` file |
| Running a class | `java ClassName` — no `.class` extension, use dots for packages |
| Single-file execution | `java Hello.java` — Java 11+, first class in file is run |
| JDK contains | `javac`, `java`, `jar`, `javadoc`, `jshell`, `jdeps`, `jlink`, `jmod` + JRE |
| JRE contains | JVM + core libraries (no compiler) |
| JVM is | Platform-**specific** (different per OS) |
| Bytecode is | Platform-**independent** |
| Java execution model | Compiled (to bytecode) + Interpreted/JIT (to native) = **both** |
| Standalone JRE | **Removed** from Java 11+ distribution — use `jlink` instead |
| Inner class files | Named with `$` separator — e.g., `Outer$Inner.class` |

---

### Compile-Time vs Runtime Quick Reference

| Error Type | When Caught | Example |
|---|---|---|
| Type mismatch | Compile-time | `int x = "hi"` |
| Missing method | Compile-time | `str.unknownMethod()` |
| Access violation | Compile-time | Accessing `private` field |
| `NullPointerException` | Runtime | `null.length()` |
| `ClassCastException` | Runtime | `(Dog) new Cat()` |
| `ArrayIndexOutOfBoundsException` | Runtime | `arr[10]` on size-5 array |
| `StackOverflowError` | Runtime | Infinite recursion |

---

### JDK Tool Quick Reference

| Tool | Purpose |
|---|---|
| `javac` | Compile `.java` → `.class` |
| `java` | Run `.class` files |
| `jar` | Package `.class` files into archives |
| `javadoc` | Generate HTML documentation |
| `jshell` | Interactive REPL (Java 9+) |
| `jdeps` | Analyze class/module dependencies |
| `jlink` | Create custom minimal runtime images |
| `jmod` | Create and inspect `.jmod` files |

---
