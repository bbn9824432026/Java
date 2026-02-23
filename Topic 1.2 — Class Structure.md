# Topic 1.2 — Class Structure
## (Fields, Methods, Constructors, Initializers)

---

## 1. Conceptual Explanation

### What is a Class?

A class is a **blueprint** for creating objects. It defines the **state** (fields) and **behavior** (methods) that objects of that type will have. Understanding the anatomy of a class is foundational — every other OCP topic builds on this.

A Java class can contain:
- **Fields** (instance variables & static variables)
- **Methods** (instance methods & static methods)
- **Constructors**
- **Initializer blocks** (static & instance)
- **Nested classes/interfaces**
- **Comments & annotations**

---

### The Anatomy of a Class

```java
public class BankAccount {              // Class declaration

    // --- FIELDS ---
    private static int totalAccounts;   // static field
    private String owner;               // instance field
    private double balance;             // instance field

    // --- STATIC INITIALIZER ---
    static {
        totalAccounts = 0;              // runs once when class is loaded
    }

    // --- INSTANCE INITIALIZER ---
    {
        balance = 0.0;                  // runs every time an object is created
    }

    // --- CONSTRUCTOR ---
    public BankAccount(String owner) {
        this.owner = owner;
    }

    // --- METHOD ---
    public double getBalance() {
        return balance;
    }
}
```

Let's break every part down deeply.

---

### Fields

A **field** is a variable declared directly inside a class body (not inside a method).

#### Instance Fields
- Each object gets its **own copy**
- Stored on the **heap** alongside the object
- Initialized to **default values** if not explicitly assigned

#### Static Fields
- **Shared** across all instances — only one copy exists
- Stored in the **method area** of the JVM (part of heap in modern JVMs)
- Belong to the **class**, not to any object
- Accessible via class name: `BankAccount.totalAccounts`

#### Default Values (Critical for exam)

| Type | Default Value |
|---|---|
| `byte` | `0` |
| `short` | `0` |
| `int` | `0` |
| `long` | `0L` |
| `float` | `0.0f` |
| `double` | `0.0` |
| `char` | `'\u0000'` (null char) |
| `boolean` | `false` |
| `Object reference` | `null` |

> **Critical exam trap:** These defaults apply to **instance and static fields only**. **Local variables have NO default value** — using an uninitialized local variable is a **compile error**.

```java
public class Demo {
    int instanceVar;          // default: 0
    static int staticVar;     // default: 0

    void method() {
        int localVar;         // NO default
        System.out.println(localVar);  // COMPILE ERROR: variable might not be initialized
    }
}
```

---

### Methods

A method defines **behavior**. It has:
- Access modifier (`public`, `private`, `protected`, package-private)
- Optional non-access modifiers (`static`, `final`, `abstract`, `synchronized`)
- Return type (or `void`)
- Method name
- Parameter list (can be empty)
- Optional `throws` clause
- Method body

```java
public static int add(int a, int b) throws ArithmeticException {
    return a + b;
}
```

#### Method Signature
The **method signature** consists of the **method name + parameter types** (order matters). It does **NOT** include:
- Return type
- Access modifiers
- Parameter names
- `throws` clause

This is critical for understanding **overloading** (same name, different signature).

---

### Constructors

A constructor is a special block of code that **initializes an object** when it is created with `new`.

Rules:
- Constructor name **must exactly match** the class name
- Constructors have **no return type** — not even `void`
- Constructors **can be overloaded**
- If you declare no constructor, the compiler inserts a **default no-arg constructor**
- If you declare **any** constructor, the compiler does **NOT** insert a default constructor

```java
public class Cat {
    String name;

    public Cat(String name) {     // Constructor — no return type
        this.name = name;
    }

    public void Cat() {           // This is a METHOD, not a constructor (has return type void)
        // legal but confusing — exam will test this
    }
}
```

> **Exam trap:** A method with the same name as the class but with a `void` return type is a **regular method**, not a constructor. This is legal and the exam uses it to trick you.

---

### The Default Constructor

If you write **no constructors at all**, Java inserts:

```java
public ClassName() { }   // if class is public
ClassName() { }          // if class is package-private
```

The default constructor:
- Has the **same access modifier as the class**
- Has **no parameters**
- Has an **empty body** (implicitly calls `super()`)

Once you add **any** constructor with parameters, the default constructor disappears:

```java
public class Dog {
    Dog(String name) { }     // You defined a constructor
}

Dog d = new Dog();           // COMPILE ERROR — no no-arg constructor available
```

---

### Initializer Blocks

Java has two types of initializer blocks.

#### Instance Initializer Block
```java
{
    // runs every time a new instance is created
    // runs BEFORE the constructor body
    // runs AFTER super() call
}
```

#### Static Initializer Block
```java
static {
    // runs ONCE when the class is first loaded by the JVM
    // runs BEFORE any instance is created
    // runs BEFORE the main method if in the same class
}
```

#### Execution Order — The Most Tested Concept Here

When you call `new MyClass()`, the order is:

1. **Static fields** and **static initializer blocks** (in order of appearance) — only on first class load
2. **Instance fields** initialized to defaults
3. **Instance fields** explicit assignments and **instance initializer blocks** (in order of appearance)
4. **Constructor body** executes

> This order is tested heavily. Let's see it in action.

```java
public class Order {
    static int s = staticMethod();      // step 1a
    int i = instanceMethod();           // step 3a

    static {
        System.out.println("Static block");   // step 1b
    }

    {
        System.out.println("Instance block"); // step 3b
    }

    Order() {
        System.out.println("Constructor");    // step 4
    }

    static int staticMethod() {
        System.out.println("Static field init");
        return 1;
    }

    int instanceMethod() {
        System.out.println("Instance field init");
        return 1;
    }
}
```

When `new Order()` is called (first time):
```
Static field init
Static block
Instance field init
Instance block
Constructor
```

---

### Access Modifiers on Class Members

| Modifier | Same Class | Same Package | Subclass (diff pkg) | Any Class |
|---|---|---|---|---|
| `public` | ✅ | ✅ | ✅ | ✅ |
| `protected` | ✅ | ✅ | ✅ | ❌ |
| (package) | ✅ | ✅ | ❌ | ❌ |
| `private` | ✅ | ❌ | ❌ | ❌ |

> **Exam note:** The default (package-private) access has **no keyword** — you simply omit the modifier. Candidates confuse this with `protected` constantly.

---

### `this` Keyword

`this` refers to the **current instance** of the class.

Uses:
1. Distinguish instance field from local variable with same name
2. Call another constructor in the same class (`this(...)`)
3. Pass current object as argument

```java
public class Person {
    String name;

    Person(String name) {
        this.name = name;     // this.name = field, name = parameter
    }

    Person() {
        this("Unknown");      // calls Person(String name) — must be FIRST statement
    }
}
```

> **Exam trap:** `this()` constructor call must be the **first statement** in a constructor. You cannot have both `this()` and `super()` in the same constructor.

---

### Variable Hiding (Static Fields)

When a subclass declares a static field with the same name as a superclass static field, it **hides** (not overrides) the superclass field. This is distinct from instance field behavior and is tested in Domain 5, but worth planting the seed here.

---

## 2. Code Examples

### Example 1 — Default Values

```java
public class DefaultValues {
    boolean b;
    int i;
    double d;
    String s;
    char c;

    public static void main(String[] args) {
        DefaultValues dv = new DefaultValues();
        System.out.println(dv.b);  // false
        System.out.println(dv.i);  // 0
        System.out.println(dv.d);  // 0.0
        System.out.println(dv.s);  // null
        System.out.println(dv.c);  // (blank — null char \u0000)
    }
}
```

---

### Example 2 — Local Variable Has No Default

```java
public class LocalVarDemo {
    public static void main(String[] args) {
        int x;
        if (true) {
            x = 5;
        }
        System.out.println(x);  // COMPILE ERROR
        // compiler cannot guarantee x is assigned in all paths
        // even though condition is always true, compiler doesn't evaluate it
    }
}
```

> This is a subtle point. Even though `if (true)` always executes, the compiler doesn't perform flow analysis for `true`/`false` literals in this context — it sees a conditional branch and flags `x` as potentially uninitialized.

Actually, let me be precise here: modern `javac` **does** recognize `if (true)` as a constant and this specific case may compile. However, the exam typically tests:

```java
int x;
if (someCondition) {
    x = 5;
}
System.out.println(x);  // COMPILE ERROR — not definitely assigned
```

---

### Example 3 — Constructor vs Method Trap

```java
public class Trick {
    int value;

    Trick() {                   // Constructor
        value = 1;
    }

    public void Trick() {       // Method named like constructor — LEGAL
        value = 2;
    }

    public static void main(String[] args) {
        Trick t = new Trick();
        System.out.println(t.value);  // 1 — constructor ran, not the method
        t.Trick();                    // calling the method
        System.out.println(t.value);  // 2
    }
}
```

---

### Example 4 — Initializer Execution Order

```java
public class InitOrder {
    static { System.out.println("Static block 1"); }
    int x = printAndReturn("Instance field");
    static int y = printAndReturnStatic("Static field");
    { System.out.println("Instance block"); }
    static { System.out.println("Static block 2"); }

    InitOrder() { System.out.println("Constructor"); }

    static int printAndReturnStatic(String msg) {
        System.out.println(msg);
        return 0;
    }

    int printAndReturn(String msg) {
        System.out.println(msg);
        return 0;
    }

    public static void main(String[] args) {
        System.out.println("--- Creating first object ---");
        new InitOrder();
        System.out.println("--- Creating second object ---");
        new InitOrder();
    }
}
```

**Output:**
```
Static block 1
Static field
Static block 2
--- Creating first object ---
Instance field
Instance block
Constructor
--- Creating second object ---
Instance field
Instance block
Constructor
```

Key observations:
- Static parts run **once** (class load)
- Instance parts run **every time** an object is created
- Within each group, top-to-bottom order is maintained

---

### Example 5 — `this()` Must Be First Statement

```java
public class Chain {
    int a, b;

    Chain(int a) {
        this.a = a;
    }

    Chain(int a, int b) {
        this(a);              // OK — first statement
        this.b = b;
    }

    Chain() {
        System.out.println("hi"); // anything before this() = COMPILE ERROR
        this(0);                  // COMPILE ERROR: this() must be first statement
    }
}
```

---

### Example 6 — No Default Constructor After Defining One

```java
public class Point {
    int x, y;

    Point(int x, int y) {
        this.x = x;
        this.y = y;
    }
}

class Test {
    public static void main(String[] args) {
        Point p1 = new Point(1, 2);  // ✅
        Point p2 = new Point();      // ❌ COMPILE ERROR — no no-arg constructor
    }
}
```

---

### Example 7 — Static vs Instance Field Access

```java
public class Counter {
    static int count = 0;
    int id;

    Counter() {
        count++;
        id = count;
    }

    public static void main(String[] args) {
        Counter c1 = new Counter();
        Counter c2 = new Counter();
        Counter c3 = new Counter();
        System.out.println(Counter.count);  // 3
        System.out.println(c1.id);          // 1
        System.out.println(c2.id);          // 2
        System.out.println(c3.id);          // 3
    }
}
```

---

## 3. Exam-Style Questions

---

**Q1 — MCQ**
What is the output?
```java
public class Test {
    int x = 5;
    { x = 10; }
    Test() { x = 15; }

    public static void main(String[] args) {
        Test t = new Test();
        System.out.println(t.x);
    }
}
```
- A. 5
- B. 10
- C. 15
- D. Compile error

**Answer: C**
Execution order: field init (`x=5`) → instance block (`x=10`) → constructor (`x=15`). Final value is 15.

---

**Q2 — MCQ**
What is the output?
```java
public class Test {
    static int x;
    int y;

    public static void main(String[] args) {
        Test t = new Test();
        System.out.println(x + " " + t.y);
    }
}
```
- A. `0 0`
- B. `null null`
- C. Compile error
- D. Runtime error

**Answer: A**
Both `x` (static int) and `y` (instance int) default to `0`.

---

**Q3 — Select All That Apply**
Which of the following are true about constructors? (Choose all that apply)

- A. A constructor must have a return type of `void`
- B. A constructor name must match the class name
- C. A class can have multiple constructors
- D. If you define a constructor, the compiler no longer provides a default constructor
- E. Constructors can be `static`

**Answer: B, C, D**
- A: Constructors have **no** return type (not even void)
- B: Correct
- C: Correct — overloading
- D: Correct
- E: Constructors cannot be `static`

---

**Q4 — Code Output**
```java
public class Foo {
    static int a = 1;
    int b = 2;

    static {
        a = 3;
    }

    {
        b = 4;
    }

    Foo() {
        a = 5;
        b = 6;
    }

    public static void main(String[] args) {
        Foo f = new Foo();
        System.out.println(Foo.a + " " + f.b);
    }
}
```
- A. `1 2`
- B. `3 4`
- C. `5 6`
- D. `5 4`

**Answer: C**
After constructor: `a=5`, `b=6`. Static block set `a=3`, constructor overwrites to `5`. Instance block set `b=4`, constructor overwrites to `6`.

---

**Q5 — Compile Error?**
```java
public class MyClass {
    public MyClass() {
        System.out.println("Default");
    }
    public MyClass(int x) {
        this();
        System.out.println("Int: " + x);
    }
    public MyClass(int x, int y) {
        System.out.println("Two ints");
        this(x);   // Line A
    }
}
```
Does line A cause a compile error?

**Answer: Yes — compile error.**
`this(x)` at line A is not the **first statement** in the constructor. The `System.out.println` appears before it, which is illegal.

---

**Q6 — MCQ**
Which of the following has the correct description of a method signature?

- A. Method name + return type
- B. Method name + parameter types + return type
- C. Method name + parameter types
- D. Method name + parameter names + return type

**Answer: C**
The method signature is **name + parameter types only**. Return type and parameter names are NOT part of the signature.

---

**Q7 — Select All That Apply**
Which variables get a default value automatically? (Choose all that apply)

- A. Local variables inside a method
- B. Instance fields
- C. Static fields
- D. Parameters of a method
- E. Variables declared inside a `for` loop

**Answer: B, C**
Only **instance fields** and **static fields** get default values. Local variables (A, E) and parameters (D — though parameters get their value from the caller) don't get automatic defaults from the JVM.

---

**Q8 — What is the output?**
```java
public class StaticTest {
    static {
        System.out.println("A");
    }

    public static void main(String[] args) {
        System.out.println("B");
        new StaticTest();
        new StaticTest();
    }

    static {
        System.out.println("C");
    }

    StaticTest() {
        System.out.println("D");
    }
}
```
- A. `A C B D D`
- B. `A B C D D`
- C. `A C B D`
- D. `B A C D D`

**Answer: A**
Static blocks run at class load time (before `main`), top to bottom: `A` then `C`. Then `main` runs: `B`, then two constructor calls: `D`, `D`.

---

## 4. Trick Analysis

### Trap 1: `void ClassName()` looks like a constructor
Oracle writes methods that look exactly like constructors — same name as the class — but with a `void` return type. These are **regular methods**. The `new` keyword invokes the constructor, not these methods. This is one of the most common tricks on class structure questions.

---

### Trap 2: Initializer order within the same "level"
Fields and initializer blocks at the **same level** (both instance, or both static) execute in **top-to-bottom textual order**. This means:

```java
{ x = 10; }   // instance block first
int x = 5;    // field init second
```
Here `x` ends up as `5`, not `10`, because the block runs first (top-to-bottom), then the field initializer overwrites it. This surprises almost everyone.

---

### Trap 3: Static block runs before `main`
The `main` method is static, but the class still has to be **loaded** before `main` runs. Loading triggers static blocks. So static blocks always run **before** the first line of `main`. The exam places static blocks after `main` in the source code, which makes candidates think they run after — but **position in source doesn't matter for class loading order vs runtime**.

Wait — I need to be precise: static initializers run in **textual (top-to-bottom) order relative to each other and to static field initializers**, but ALL of them run before `main` starts. So two static blocks' relative order follows source order, but `main` always starts after all static initialization is complete.

---

### Trap 4: Default constructor access modifier
The default constructor's access modifier **matches the class**. A `public` class gets a `public` default constructor. A package-private class gets a package-private default constructor. This matters when subclassing across packages.

---

### Trap 5: `this()` and `super()` cannot coexist
Both `this()` and `super()` must be the first statement in a constructor. Since a constructor can only have one first statement, you **cannot use both**. If you use `this()`, the chained constructor will eventually call `super()` for you. This is tested in constructor chaining questions.

---

### Trap 6: Second object creation doesn't re-run static blocks
```java
new MyClass();   // static block runs here
new MyClass();   // static block does NOT run again
```
The static initializer runs **once per class load**, not once per object. Instance initializers run every time.

---

## 5. Summary Sheet

### Class Member Types

| Member | Per Instance or Shared? | Default Value? | Can be `static`? |
|---|---|---|---|
| Instance field | Per instance | ✅ Yes | ❌ |
| Static field | Shared (class-level) | ✅ Yes | ✅ |
| Local variable | Per method call | ❌ No | ❌ |
| Instance method | Per instance call | N/A | ❌ |
| Static method | Shared | N/A | ✅ |

---

### Execution Order (Full)

```
1. Static field initializers & static blocks  ← top to bottom, once per class load
2. Instance field initializers & instance blocks ← top to bottom, per object creation
3. Constructor body ← per object creation
```

When inheritance is involved (covered in Domain 5), parent class static → child class static → parent instance → parent constructor → child instance → child constructor.

---

### Default Values Quick Reference

| Type | Default |
|---|---|
| `int`, `short`, `byte`, `long` | `0` |
| `float`, `double` | `0.0` |
| `boolean` | `false` |
| `char` | `'\u0000'` |
| Any reference | `null` |
| **Local variable** | **NONE — compile error if used uninitialized** |

---

### Constructor Rules

| Rule | Detail |
|---|---|
| Name | Must match class name exactly |
| Return type | None — not even `void` |
| Default constructor | Provided only if **no** constructors are declared |
| `this()` call | Must be the **first** statement |
| `super()` call | Must be the **first** statement |
| `this()` + `super()` | **Cannot both appear** in same constructor |
| Access modifiers | All four allowed (`public`, `protected`, package, `private`) |
| `static` constructor | **Not allowed** — static initializers serve this role |

---

### Access Modifier Summary

| Modifier | Class | Package | Subclass | World |
|---|---|---|---|---|
| `public` | ✅ | ✅ | ✅ | ✅ |
| `protected` | ✅ | ✅ | ✅ | ❌ |
| (none) | ✅ | ✅ | ❌ | ❌ |
| `private` | ✅ | ❌ | ❌ | ❌ |

---
