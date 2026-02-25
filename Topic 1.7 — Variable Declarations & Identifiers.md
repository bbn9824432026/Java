# Topic 1.7 — Variable Declarations & Identifiers

---

## 1. Conceptual Explanation

### What is an Identifier?

An identifier is any **name** you give to a variable, method, class, interface, package, or label in Java. The compiler uses identifiers to distinguish one element from another. The rules for what constitutes a legal identifier are strict and tested precisely on the OCP exam.

---

### Identifier Rules — Legal vs Illegal

An identifier in Java:

**MUST:**
- Start with a **letter** (`A-Z`, `a-z`), a **currency symbol** (`$`, `€`, `£`, `¥`), or an **underscore** (`_`)
- Contain only letters, digits (`0-9`), currency symbols, or underscores after the first character
- Not be a **reserved keyword**
- Not be `true`, `false`, or `null` (these are literals, not keywords, but still reserved)

**MUST NOT:**
- Start with a digit
- Contain spaces, hyphens, dots, or special characters (except `$` and `_`)
- Be a Java keyword

```java
// LEGAL identifiers
int age;
int _count;
int $price;
int myVar123;
int MAX_VALUE;
int café;        // Unicode letters are allowed
int π;           // Unicode — technically legal

// ILLEGAL identifiers
int 1stPlace;    // COMPILE ERROR — starts with digit
int my-var;      // COMPILE ERROR — hyphen not allowed
int my var;      // COMPILE ERROR — space not allowed
int class;       // COMPILE ERROR — reserved keyword
int true;        // COMPILE ERROR — reserved literal
int null;        // COMPILE ERROR — reserved literal
int for;         // COMPILE ERROR — reserved keyword
```

---

### Reserved Keywords

Java has **55 reserved keywords** in Java 17 that cannot be used as identifiers. The exam expects you to recognize common ones, especially tricky ones:

**Primitive type keywords:**
`byte`, `short`, `int`, `long`, `float`, `double`, `char`, `boolean`

**Control flow:**
`if`, `else`, `switch`, `case`, `default`, `for`, `while`, `do`, `break`, `continue`, `return`

**Class structure:**
`class`, `interface`, `enum`, `extends`, `implements`, `abstract`, `final`, `static`, `new`, `this`, `super`

**Access modifiers:**
`public`, `private`, `protected`

**Exception handling:**
`try`, `catch`, `finally`, `throw`, `throws`

**Other:**
`void`, `import`, `package`, `instanceof`, `native`, `strictfp`, `synchronized`, `transient`, `volatile`

**Module-related (Java 9+, context-sensitive):**
`module`, `requires`, `exports`, `opens`, `uses`, `provides`, `to`, `with`, `transitive`

**Special:**
`var` — **NOT a reserved keyword** — it is a reserved type name. This distinction matters.

```java
// var is NOT a keyword — these are legal:
int var = 5;          // variable named "var" — LEGAL
String var = "hello"; // LEGAL (but terrible practice)

// BUT you cannot use var as a type name for a class:
// class var { }      // COMPILE ERROR — var is a reserved type name
```

**`record`, `sealed`, `permits`, `yield`** — also context-sensitive keywords in Java 17.

---

### `_` (Single Underscore) as Identifier — Java 9+

In Java 8 and earlier, a single underscore `_` was a valid identifier. Starting from **Java 9**, using `_` alone as an identifier generates a compile error (it is reserved for future use as a pattern variable in Java 21+):

```java
int _ = 5;    // COMPILE ERROR in Java 9+ — single underscore reserved
int __ = 5;   // OK — two underscores is fine
int _x = 5;   // OK — underscore followed by other characters is fine
```

---

### Variable Declaration Syntax

A variable declaration has the general form:

```
[modifiers] type variableName [= initializer];
```

#### Local Variable Declaration

```java
void method() {
    int x;               // declared, not initialized
    int y = 10;          // declared and initialized
    int a, b, c;         // multiple declarations in one statement
    int d = 1, e = 2;    // multiple with initializers
    final int MAX = 100; // final local variable
}
```

#### Instance Variable Declaration

```java
public class Person {
    private String name;           // instance field
    private int age = 0;           // with initializer
    private final String ID;       // blank final — must be initialized in constructor
    static int count = 0;          // static field
    static final int LIMIT = 10;   // constant
}
```

---

### Multiple Variable Declarations

You can declare multiple variables of the **same type** in a single statement:

```java
int a, b, c;              // three int variables, all uninitialized
int x = 1, y = 2, z = 3; // three ints with individual initializers
```

**Critical rule:** The type applies to ALL variables in the statement. You cannot mix types:

```java
int a = 1, long b = 2;   // COMPILE ERROR — cannot mix types
```

**Array declarations with multiple variables — a tricky case:**

```java
int[] a, b;     // BOTH a and b are int arrays
int a[], b;     // a is int array, b is plain int — bracket position matters!
int[] a, b[];   // a is int[], b is int[][] — very tricky!
```

> **Exam trap:** When the bracket is after the type (`int[]`), both variables are arrays. When the bracket is after the variable name, only that variable gets the array dimension.

---

### `final` Variables

A `final` variable can only be assigned **once**. After that, its value cannot change.

#### `final` Local Variables
```java
void method() {
    final int x = 10;
    x = 20;           // COMPILE ERROR — cannot reassign final
    
    final int y;      // OK — declared but not yet assigned
    y = 5;            // OK — first assignment
    y = 10;           // COMPILE ERROR — second assignment
}
```

#### `final` Instance Fields — Blank Finals
A `final` instance field doesn't have to be initialized at declaration. But it **must** be initialized in every constructor:

```java
public class Circle {
    final double radius;     // blank final

    Circle(double r) {
        radius = r;          // OK — assigned in constructor
    }

    Circle() {
        // COMPILE ERROR if radius not assigned here
        radius = 1.0;        // must be assigned
    }
}
```

#### `final` and References

`final` on a reference variable means the reference cannot be reassigned — it does NOT mean the object is immutable:

```java
final List<String> list = new ArrayList<>();
list.add("hello");    // OK — modifying the object is fine
list = new ArrayList<>();  // COMPILE ERROR — cannot reassign final reference
```

---

### Naming Conventions (Not Rules — But Tested for Recognition)

The exam presents code and sometimes asks which follows Java conventions. Know these:

| Element | Convention | Example |
|---|---|---|
| Variable | camelCase | `firstName`, `accountBalance` |
| Method | camelCase | `getBalance()`, `calculateTax()` |
| Class | PascalCase | `BankAccount`, `HttpRequest` |
| Interface | PascalCase | `Comparable`, `Runnable` |
| Constant | UPPER_SNAKE_CASE | `MAX_VALUE`, `DEFAULT_TIMEOUT` |
| Package | all lowercase | `com.example.util` |
| Generic type parameter | Single uppercase letter | `T`, `E`, `K`, `V` |

Conventions are **not enforced by the compiler** — code with unconventional names compiles fine. But the exam may present choices and ask which follows standard conventions.

---

### Variable Types and Where They Live

Java has four kinds of variables:

#### 1. Local Variables
- Declared inside a method, constructor, or block
- No default value — **must be initialized before use**
- Scope: from declaration to end of enclosing block
- Cannot have access modifiers (`public`, `private`, etc.)
- Can be `final`

```java
void method() {
    int x;                 // local — no default
    System.out.println(x); // COMPILE ERROR — not initialized
}
```

#### 2. Instance Variables (Fields)
- Declared in class body, outside any method
- Have default values (0, false, null)
- Scope: entire class (accessible through reference)
- Can have access modifiers
- Can be `final`, `transient`, `volatile`

#### 3. Static Variables (Class Variables)
- Declared with `static` keyword in class body
- Have default values
- Shared across all instances
- Scope: entire class, accessible via class name
- Can have access modifiers
- Can be `final`, `transient`, `volatile`

#### 4. Parameters (Method/Constructor Parameters)
- Variables declared in method/constructor signature
- Value provided by the caller
- Treated like local variables inside the method
- No default value (caller provides it)
- Can be `final`

```java
public void setName(final String name) {  // final parameter
    // name = "other";  // COMPILE ERROR — final parameter
    this.name = name;   // reading is fine
}
```

---

### Variable Scope

Scope defines where a variable is accessible. Java has block scope:

```java
public class ScopeDemo {
    int instanceVar = 1;          // class scope

    void method() {
        int localVar = 2;         // method scope

        if (true) {
            int blockVar = 3;     // block scope — only inside this if
            System.out.println(instanceVar); // OK
            System.out.println(localVar);    // OK
            System.out.println(blockVar);    // OK
        }

        System.out.println(blockVar);  // COMPILE ERROR — out of scope
    }
}
```

#### Variable Shadowing

A local variable can **shadow** (hide) an instance field with the same name:

```java
public class Shadow {
    int x = 10;

    void method() {
        int x = 20;                // shadows instance field
        System.out.println(x);     // 20 — local variable
        System.out.println(this.x); // 10 — instance field via this
    }
}
```

#### You Cannot Redeclare a Local Variable in Overlapping Scope

```java
void method() {
    int x = 5;
    {
        int x = 10;  // COMPILE ERROR — x already defined in enclosing scope
    }
}
```

But you CAN reuse names in **non-overlapping** scopes:

```java
void method() {
    {
        int x = 5;
    }   // x goes out of scope here

    {
        int x = 10;  // OK — previous x is gone
    }
}
```

---

### Declaring Variables in Different Contexts

#### In `for` Loops

```java
for (int i = 0; i < 10; i++) {
    System.out.println(i);  // i in scope
}
System.out.println(i);  // COMPILE ERROR — i out of scope
```

#### In `switch` Statements

```java
switch (x) {
    case 1:
        int y = 10;    // y is declared here
        break;
    case 2:
        y = 20;        // y is STILL IN SCOPE — the whole switch is one block
        break;
}
```

> **Exam trap:** Variables declared inside a `switch` case are scoped to the **entire switch block**, not just that case. You can reference them in other cases. But initializing and using them across cases without assignment can cause compile errors.

---

### Constants — `static final`

The idiomatic Java constant:

```java
public class Config {
    public static final int MAX_CONNECTIONS = 100;
    public static final String APP_NAME = "MyApp";
    public static final double PI = 3.14159265358979;
}
```

Used as: `Config.MAX_CONNECTIONS`

- `static` — belongs to class, not instance
- `final` — cannot be reassigned
- `UPPER_SNAKE_CASE` — convention
- Must be initialized at declaration OR in a static initializer block

---

## 2. Code Examples

### Example 1 — Legal and Illegal Identifiers

```java
public class IdentifierDemo {
    int age = 25;          // legal
    int _count = 0;        // legal
    int $price = 10;       // legal
    int myVar2 = 5;        // legal
    // int 2ndPlace = 1;   // COMPILE ERROR — starts with digit
    // int my-var = 1;     // COMPILE ERROR — hyphen
    // int class = 1;      // COMPILE ERROR — keyword
    // int true = 1;       // COMPILE ERROR — literal
}
```

---

### Example 2 — Multiple Declarations, Array Trick

```java
public class MultiDecl {
    public static void main(String[] args) {
        int a = 1, b = 2, c = 3;     // all ints
        System.out.println(a + b + c); // 6

        int[] x, y;    // BOTH are int arrays
        int z[], w;    // z is int[], w is plain int

        x = new int[3];
        y = new int[5];
        z = new int[2];
        w = 10;        // plain int

        // int[] p, q[];  // p is int[], q is int[][]
    }
}
```

---

### Example 3 — `final` Variable Rules

```java
public class FinalDemo {
    public static void main(String[] args) {
        final int x = 10;
        // x = 20;           // COMPILE ERROR

        final int y;
        y = 5;               // OK — first assignment
        // y = 10;           // COMPILE ERROR — second assignment

        final int z;
        if (true) z = 1;
        // System.out.println(z);  // COMPILE ERROR
        // compiler can't guarantee z is assigned in all paths
        // even with if(true) — depends on javac version/settings
    }
}
```

---

### Example 4 — `final` Reference

```java
import java.util.ArrayList;
import java.util.List;

public class FinalRef {
    public static void main(String[] args) {
        final List<String> list = new ArrayList<>();

        list.add("one");     // OK — modifying object content
        list.add("two");     // OK

        // list = new ArrayList<>();  // COMPILE ERROR — reassigning reference

        System.out.println(list);  // [one, two]
    }
}
```

---

### Example 5 — Scope Demo

```java
public class ScopeTest {
    int x = 100;   // instance field

    void demonstrate() {
        System.out.println(x);   // 100 — instance field

        int x = 200;             // local variable shadows field
        System.out.println(x);   // 200 — local variable
        System.out.println(this.x); // 100 — explicit field access

        for (int i = 0; i < 3; i++) {
            int loopVar = i * 2;
            System.out.println(loopVar);
        }
        // System.out.println(i);       // COMPILE ERROR
        // System.out.println(loopVar); // COMPILE ERROR
    }
}
```

---

### Example 6 — Variable in switch Scope

```java
public class SwitchScope {
    public static void main(String[] args) {
        int val = 2;

        switch (val) {
            case 1:
                int result;          // declared here
                result = 10;
                System.out.println(result);
                break;
            case 2:
                result = 20;         // same result variable — still in scope
                System.out.println(result);
                break;
        }
    }
}
```

---

### Example 7 — `var` is Not a Keyword

```java
public class VarIdentifier {
    public static void main(String[] args) {
        int var = 42;            // legal — var used as variable name
        System.out.println(var); // 42

        String var2 = "hello";
        // legal — var only special as a type context
    }

    // void method(var x) { }   // var can be a param name but confusing
}
```

---

### Example 8 — Blank Final Field

```java
public class Circle {
    final double radius;      // blank final

    Circle(double r) {
        this.radius = r;     // assigned in constructor — OK
    }

    Circle() {
        this.radius = 1.0;   // must assign in EVERY constructor
    }

    void method() {
        // radius = 5.0;     // COMPILE ERROR — final already set in constructor
    }
}
```

---

### Example 9 — Redeclaration in Overlapping Scope

```java
public class RedeclarationDemo {
    public static void main(String[] args) {
        int x = 5;

        {
            // int x = 10;  // COMPILE ERROR — x already in scope
            x = 10;         // OK — assigning to outer x
            System.out.println(x);  // 10
        }

        System.out.println(x);  // 10

        // Non-overlapping scopes
        {
            int y = 1;
        }
        {
            int y = 2;   // OK — previous y is out of scope
        }
    }
}
```

---

## 3. Exam-Style Questions

---

**Q1 — Select All That Apply**
Which of the following are legal identifiers in Java? (Choose all that apply)

- A. `_myVar`
- B. `$amount`
- C. `2ndPlace`
- D. `my-variable`
- E. `_`
- F. `MyClass123`
- G. `false`
- H. `TOTAL_COUNT`

**Answer: A, B, F, H**
C starts with a digit. D contains a hyphen. E is single underscore — illegal in Java 9+. G is a reserved literal.

---

**Q2 — MCQ**
What is the output?
```java
public class Test {
    static int x = 5;

    public static void main(String[] args) {
        int x = 10;
        {
            int y = 20;
            System.out.println(x + y);
        }
        System.out.println(x);
    }
}
```
- A. `30` then `5`
- B. `30` then `10`
- C. `15` then `10`
- D. Compile error

**Answer: B**
Local `x = 10` shadows static `x = 5`. `x + y = 10 + 20 = 30`. Second print uses local `x = 10`.

---

**Q3 — MCQ**
```java
public class Test {
    public static void main(String[] args) {
        int[] a, b[];
        a = new int[3];
        b = new int[2][3];
        System.out.println(a.length + " " + b.length);
    }
}
```
- A. `3 3`
- B. `3 2`
- C. `3 6`
- D. Compile error

**Answer: B**
`int[] a, b[]` — `a` is `int[]`, `b` is `int[][]`. `a.length = 3`, `b.length = 2` (number of rows).

---

**Q4 — MCQ**
```java
public class Test {
    public static void main(String[] args) {
        final int x;
        x = 10;
        x = 20;   // Line A
        System.out.println(x);
    }
}
```
What is the result?

- A. Prints `20`
- B. Prints `10`
- C. Compile error at Line A
- D. Runtime error

**Answer: C**
`x` is `final`. After the first assignment `x = 10`, it cannot be reassigned. Line A causes a compile error.

---

**Q5 — Select All That Apply**
Which of the following are true about `final` variables? (Choose all that apply)

- A. A `final` local variable must be initialized at declaration
- B. A `final` instance field must be assigned in every constructor
- C. A `final` reference prevents modification of the referenced object
- D. A `final` variable can be assigned exactly once
- E. `final` parameters cannot be reassigned inside the method

**Answer: B, D, E**
A is false — `final` locals can be declared then assigned once later (blank final local). C is false — `final` on reference only prevents reassignment of the reference, not mutation of the object. B, D, E are correct.

---

**Q6 — MCQ**
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
- B. `0 1 2` then compile error at Line A
- C. `0 1 2 2`
- D. Compile error at Line A only

**Answer: D**
`i` is declared in the `for` loop and is out of scope after the loop. Line A causes a compile error — but this is detected at compile time so the program doesn't run at all.

---

**Q7 — MCQ**
Which of the following is true about `var`?

- A. `var` is a reserved keyword and cannot be used as an identifier
- B. `var` can be used as a variable name but not as a class name
- C. `var` can be used as both a class name and variable name
- D. `var` cannot appear anywhere in valid Java 17 code except as a type

**Answer: B**
`var` is a **reserved type name**, not a keyword. It can be used as a variable name (`int var = 5` is legal) but cannot be used as a class, interface, or enum name.

---

**Q8 — MCQ**
```java
public class Test {
    int x = 10;

    void method() {
        int x = 20;
        System.out.println(x);
        System.out.println(this.x);
    }

    public static void main(String[] args) {
        new Test().method();
    }
}
```
What is the output?

- A. `10` then `20`
- B. `20` then `10`
- C. `20` then `20`
- D. Compile error

**Answer: B**
Local `x = 20` shadows instance `x = 10`. `x` prints local (20), `this.x` prints instance field (10).

---

**Q9 — Select All That Apply**
Which declarations are valid? (Choose all that apply)

- A. `int a, b, c;`
- B. `int a = 1, b = 2;`
- C. `int a = 1, long b = 2;`
- D. `int[] a, b;`
- E. `int a[], b;`
- F. `int a = b = 5;`

**Answer: A, B, D, E**
C mixes types — illegal. F: `b = 5` would require `b` to be already declared — illegal as a declaration. A, B, D, E are all valid.

---

**Q10 — MCQ**
```java
public class Test {
    public static void main(String[] args) {
        int x = 0;
        switch (x) {
            case 0:
                int y = 10;
            case 1:
                y = 20;         // Line A
                System.out.println(y);
        }
    }
}
```
What is the result?

- A. Compile error at Line A — `y` not in scope
- B. Prints `20` — fall-through executes both cases
- C. Prints `10`
- D. Compile error — `y` might not be initialized

**Answer: D**
`y` is declared in case 0 but initialized there. Due to fall-through, case 1 runs. However, the compiler sees that if `x = 1`, case 0 is skipped and `y` is never initialized — so `y` at Line A might be uninitialized. This is a compile error. The whole switch shares one scope, but initialization paths matter.

---

## 4. Trick Analysis

### Trap 1: `_` alone is illegal in Java 9+
A single underscore used as an identifier is a compile error in Java 9+. Two underscores `__` is fine. `_x` is fine. Just `_` alone is reserved. This changed between Java 8 and 9, and the exam tests Java 17 behavior.

---

### Trap 2: `var` is not a keyword
`int var = 5;` is legal. You can name a variable `var`. You cannot name a class `var`. Many candidates think `var` is fully reserved like `int` or `class` — it's not. It's a context-sensitive reserved type name.

---

### Trap 3: `int[] a, b` vs `int a[], b`
Bracket position in declarations changes the type:
- `int[] a, b` → both `a` and `b` are `int[]`
- `int a[], b` → `a` is `int[]`, `b` is plain `int`
- `int[] a, b[]` → `a` is `int[]`, `b` is `int[][]`

This is a favorite exam trick because the difference is purely visual.

---

### Trap 4: `final` local doesn't need initialization at declaration
`final int x;` followed later by `x = 5;` is perfectly legal — as long as it's assigned exactly once before use. Candidates assume `final` requires initialization at declaration. It doesn't — it requires exactly one assignment total.

---

### Trap 5: `final` reference ≠ immutable object
`final List<String> list` means you can't do `list = otherList`. But you can still call `list.add()`, `list.remove()`, etc. The reference is frozen, not the object.

---

### Trap 6: Switch scope — one block, not per case
A variable declared in one case is in scope for ALL subsequent cases in the same switch. But initialization only happens if execution reaches the declaration. This can lead to "might not be initialized" compile errors for fall-through scenarios.

---

### Trap 7: `true`, `false`, `null` are not keywords but still reserved
They are **literals**, not keywords. But they are just as reserved — you cannot use them as identifiers. The distinction matters only in the sense that they're sometimes listed separately from keywords in Java spec documents. For the exam, treat them the same: cannot be identifiers.

---

### Trap 8: Local variable shadowing a field
```java
int x = 10;         // field
void method() {
    int x = 20;     // shadows field — not a compile error
}
```
This compiles fine. Java allows local variables to shadow instance fields. `this.x` accesses the field. This is different from redeclaring a local in the same scope, which IS an error.

---

## 5. Summary Sheet

### Identifier Rules

| Rule | Detail |
|---|---|
| Must start with | Letter, `$`, or `_` |
| Can contain | Letters, digits, `$`, `_` |
| Cannot start with | Digit |
| Cannot contain | Spaces, `-`, `.`, `@`, etc. |
| Cannot be | Java keyword, `true`, `false`, `null` |
| Single `_` | Illegal in Java 9+ |
| `var` | Legal as variable/method name; illegal as class/interface/enum name |
| Unicode | Unicode letters and digits are technically legal |

---

### Variable Categories

| Type | Default Value | Access Modifiers | `final` Allowed | Scope |
|---|---|---|---|---|
| Local | None (compile error) | ❌ | ✅ | Block |
| Instance field | Yes | ✅ | ✅ | Class |
| Static field | Yes | ✅ | ✅ | Class |
| Parameter | From caller | ❌ | ✅ | Method |

---

### `final` Variable Rules

| Context | Rule |
|---|---|
| `final` local | Assign exactly once before use |
| `final` instance field | Assign in every constructor (or at declaration) |
| `final` static field | Assign at declaration or in static initializer |
| `final` parameter | Cannot reassign inside method |
| `final` reference | Reference frozen; object still mutable |

---

### Array Declaration Trick

| Declaration | Types |
|---|---|
| `int[] a, b;` | Both `int[]` |
| `int a[], b;` | `a` is `int[]`, `b` is `int` |
| `int[] a, b[];` | `a` is `int[]`, `b` is `int[][]` |
| `int a[], b[];` | Both `int[]` |

---

### Naming Conventions

| Element | Convention | Example |
|---|---|---|
| Variable/Method | camelCase | `firstName`, `getAge()` |
| Class/Interface | PascalCase | `BankAccount` |
| Constant | UPPER_SNAKE_CASE | `MAX_SIZE` |
| Package | lowercase | `com.example` |
| Generic param | Single uppercase | `T`, `E`, `K`, `V` |

---
