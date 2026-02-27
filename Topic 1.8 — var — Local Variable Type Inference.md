# Topic 1.8 — `var` — Local Variable Type Inference

---

## 1. Conceptual Explanation

### What is `var`?

Introduced in **Java 10**, `var` is a **reserved type name** that allows the compiler to **infer the type** of a local variable from its initializer. Instead of explicitly writing the type, you let the compiler figure it out from the right-hand side of the assignment.

```java
// Traditional explicit typing
ArrayList<String> list = new ArrayList<String>();

// With var — compiler infers ArrayList<String>
var list = new ArrayList<String>();
```

`var` does **not** make Java dynamically typed. Java remains **statically typed**. The type is still determined at **compile time** — `var` is just syntactic sugar that lets the compiler do the type-writing for you. Once inferred, the type is **fixed** and cannot change.

---

### How Type Inference Works

The compiler looks at the **initializer** (the right-hand side of the `=`) and infers the type:

```java
var x = 42;           // inferred as int
var d = 3.14;         // inferred as double
var s = "hello";      // inferred as String
var list = new ArrayList<String>();  // inferred as ArrayList<String>
var arr = new int[5]; // inferred as int[]
```

After inference, these are **exactly the same** as:
```java
int x = 42;
double d = 3.14;
String s = "hello";
ArrayList<String> list = new ArrayList<String>();
int[] arr = new int[5];
```

The bytecode is identical. `var` has zero runtime cost and zero runtime presence — it's purely a compile-time feature.

---

### Where `var` CAN Be Used

`var` is valid **only for local variables** in these specific contexts:

#### 1. Local variables in methods
```java
void method() {
    var x = 10;              // OK
    var name = "Alice";      // OK
    var list = new ArrayList<String>();  // OK
}
```

#### 2. `for` loop index variable
```java
for (var i = 0; i < 10; i++) {   // OK — i inferred as int
    System.out.println(i);
}
```

#### 3. Enhanced `for-each` loop variable
```java
List<String> names = List.of("Alice", "Bob");
for (var name : names) {    // OK — name inferred as String
    System.out.println(name);
}
```

#### 4. `try-with-resources` variable
```java
try (var reader = new BufferedReader(new FileReader("file.txt"))) {
    // OK — reader inferred as BufferedReader
}
```

---

### Where `var` CANNOT Be Used

This is where the exam focuses. There are many restrictions:

#### 1. Instance fields
```java
public class Demo {
    var name = "Alice";   // COMPILE ERROR — var not allowed for fields
}
```

#### 2. Static fields
```java
public class Demo {
    static var count = 0;  // COMPILE ERROR — var not allowed for fields
}
```

#### 3. Method parameters
```java
void method(var x) {       // COMPILE ERROR — var not allowed for parameters
}
```

#### 4. Constructor parameters
```java
Demo(var x) {              // COMPILE ERROR
}
```

#### 5. Method return type
```java
var getAge() {             // COMPILE ERROR — var not allowed as return type
    return 25;
}
```

#### 6. Without an initializer
```java
var x;                     // COMPILE ERROR — no initializer, cannot infer type
```

#### 7. Initialized to `null` without a cast
```java
var x = null;              // COMPILE ERROR — cannot infer type from null
```

#### 8. Array initializer shorthand
```java
var arr = {1, 2, 3};       // COMPILE ERROR — array initializer without new
var arr2 = new int[]{1, 2, 3};  // OK
```

#### 9. Lambda expressions directly
```java
var lambda = x -> x + 1;  // COMPILE ERROR — lambda has no standalone type
```

#### 10. Method references directly
```java
var ref = String::length;  // COMPILE ERROR — method reference needs a target type
```

---

### Why These Restrictions Exist

The reason `var` requires an initializer and cannot be used in certain places comes down to one fundamental requirement: **the compiler must be able to determine an unambiguous type at the point of declaration.**

- **Fields** don't have initializers at the field level always, and their type is part of the class's public API.
- **Parameters** are typed by the caller — the compiler needs to know the type to check call sites.
- **`null`** has no type — it is compatible with any reference type, so there's nothing to infer.
- **Lambda expressions** don't have a standalone type — they are always typed by their target functional interface.
- **Array initializers** (`{1, 2, 3}`) don't carry type information on their own.

---

### `var` and Type Inference — What Type Gets Inferred?

The inferred type is the **declared type of the right-hand side**, which can sometimes surprise you:

#### Inferring the concrete type, not the interface
```java
var list = new ArrayList<String>();
// inferred as ArrayList<String>, NOT List<String>

list.trimToSize();  // OK — ArrayList-specific method accessible
```

If you had written `List<String> list = new ArrayList<>()`, you'd only have access to `List` methods. With `var`, you get the full `ArrayList` type.

#### Numeric literals infer their default type
```java
var i = 42;     // int — not byte, not short, not long
var d = 3.14;   // double — not float
var f = 3.14f;  // float — because of f suffix
var l = 100L;   // long — because of L suffix
```

#### `var` with a String literal
```java
var s = "hello";  // String
s = 42;           // COMPILE ERROR — s is String, not Object
```

Once inferred, the type is locked. `var` does not become `Object` — it becomes whatever the specific inferred type is.

---

### `var` and Polymorphism

Because `var` infers the **concrete type** (not a supertype or interface), it affects what methods are accessible:

```java
// Using interface type — only interface methods accessible
Runnable r = () -> System.out.println("run");

// Using var with anonymous class — more methods accessible
var obj = new Object() {
    String name = "anonymous";
    void greet() { System.out.println("Hello " + name); }
};

obj.greet();   // OK! — var captures the anonymous class type
obj.name;      // OK! — direct field access
```

This is a unique capability of `var` — accessing members of anonymous classes that have no named type.

---

### `var` in Loops — Detailed

```java
// for loop
for (var i = 0; i < 5; i++) {
    var squared = i * i;   // inferred as int
    System.out.println(squared);
}

// enhanced for
var numbers = List.of(1, 2, 3, 4, 5);
for (var n : numbers) {    // n inferred as Integer (boxed type from List)
    System.out.println(n);
}

// With map entries
var map = Map.of("a", 1, "b", 2);
for (var entry : map.entrySet()) {  // entry inferred as Map.Entry<String, Integer>
    System.out.println(entry.getKey() + "=" + entry.getValue());
}
```

---

### `var` is Not `dynamic` or `Object`

A critical distinction:

```java
var x = "hello";
x = 42;            // COMPILE ERROR — x is String, not Object or dynamic
```

`var` is NOT:
- `Object x` (which could hold anything)
- Dynamic typing (which allows type changes at runtime)
- Like JavaScript's `var`

`var` locks in a **single specific type** at compile time. Period.

---

### `var` and Generics — Diamond Operator Interaction

```java
// With explicit type — diamond operator works fine
List<String> list1 = new ArrayList<>();    // infers String from left side

// With var — diamond infers Object, which may not be what you want
var list2 = new ArrayList<>();             // inferred as ArrayList<Object>!
list2.add("hello");   // OK
list2.add(42);        // OK — because it's ArrayList<Object>

// Better — be explicit about generic type with var
var list3 = new ArrayList<String>();       // inferred as ArrayList<String>
// list3.add(42);  // COMPILE ERROR — correctly typed
```

> **Exam trap:** `var list = new ArrayList<>()` infers `ArrayList<Object>` because the diamond operator with no type context defaults to `Object`. Always specify the generic type when using `var` with collections.

---

### `var` and `final`

`var` and `final` can be combined:

```java
final var x = 42;    // x is final int — cannot be reassigned
final var name = "Alice";  // name is final String
```

This is valid. `final var` is a `final` variable with an inferred type.

---

### `var` as a Variable Name (Revisited from 1.7)

Because `var` is a reserved type name (not a keyword), you can use it as a variable name:

```java
var var = 5;          // legal but terrible — var named "var" with inferred type int
System.out.println(var);  // 5
```

The compiler can parse this because `var` on the left is the type inference keyword (in type position), and `var` on the right-hand... wait — here the left `var` is the type and `var` on the left of `=` is the variable name. Let me be precise:

```java
var var = 5;
// first var = type inference keyword (in type position)
// second var = variable name
// inferred type: int
// variable name: var
```

This is legal. Confusing, but legal. The exam may test whether this compiles.

---

### Scope and Reassignment of `var`

Once inferred, the type is fixed, but the value can change (unless `final`):

```java
var x = 10;    // x is int
x = 20;        // OK — reassigning int to int
x = "hello";   // COMPILE ERROR — x is int, not String
x = 3.14;      // COMPILE ERROR — x is int, not double
```

---

### `var` and Null Handling

```java
var x = null;               // COMPILE ERROR — cannot infer from null

var x = (String) null;      // OK — inferred as String (cast provides type)
System.out.println(x);      // null
x = "hello";                // OK
x = 42;                     // COMPILE ERROR — x is String
```

Casting `null` gives it a type, which `var` can then infer.

---

## 2. Code Examples

### Example 1 — Basic Inference

```java
public class VarBasics {
    public static void main(String[] args) {
        var age = 25;                    // int
        var price = 9.99;                // double
        var name = "Claude";             // String
        var isActive = true;             // boolean
        var letter = 'A';               // char

        System.out.println(age);         // 25
        System.out.println(price);       // 9.99
        System.out.println(name);        // Claude
        System.out.println(isActive);    // true
        System.out.println(letter);      // A
    }
}
```

---

### Example 2 — `var` is Not Dynamic

```java
public class VarNotDynamic {
    public static void main(String[] args) {
        var x = "hello";     // inferred as String
        System.out.println(x.toUpperCase());  // HELLO

        x = "world";         // OK — still String
        // x = 42;           // COMPILE ERROR — x is String forever
        // x = null;         // Actually OK — null can be assigned to String reference
        x = null;            // Fine — String reference can be null
    }
}
```

---

### Example 3 — `var` with Collections

```java
import java.util.*;

public class VarCollections {
    public static void main(String[] args) {
        var list = new ArrayList<String>();  // ArrayList<String>
        list.add("one");
        list.add("two");
        list.trimToSize();   // ArrayList-specific method — accessible!

        var map = new HashMap<String, Integer>();
        map.put("a", 1);

        for (var entry : map.entrySet()) {
            // entry is Map.Entry<String, Integer>
            System.out.println(entry.getKey() + ": " + entry.getValue());
        }
    }
}
```

---

### Example 4 — Diamond Operator Trap

```java
import java.util.*;

public class DiamondTrap {
    public static void main(String[] args) {
        var list1 = new ArrayList<>();          // ArrayList<Object> — untyped!
        list1.add("hello");
        list1.add(42);
        list1.add(3.14);    // all OK — Object accepts anything

        var list2 = new ArrayList<String>();    // ArrayList<String> — correct
        list2.add("hello");
        // list2.add(42);   // COMPILE ERROR — correctly typed as String
    }
}
```

---

### Example 5 — `var` with Anonymous Class

```java
public class VarAnonymous {
    public static void main(String[] args) {
        var obj = new Object() {
            String greeting = "Hello";
            int count = 42;

            void display() {
                System.out.println(greeting + " " + count);
            }
        };

        obj.display();           // Hello 42
        System.out.println(obj.greeting);  // Hello
        System.out.println(obj.count);     // 42
        // All anonymous class members are accessible via var!
    }
}
```

---

### Example 6 — Compile Errors with `var`

```java
public class VarErrors {
    // var field = "hello";         // ERROR — no var for fields

    public static void main(String[] args) {
        // var x;                   // ERROR — no initializer
        // var y = null;            // ERROR — cannot infer from null
        // var arr = {1, 2, 3};    // ERROR — array initializer needs new
        // var lam = x -> x;       // ERROR — lambda needs target type

        var good = new int[]{1, 2, 3};  // OK
        var str = (String) null;         // OK — null with cast

        for (var i = 0; i < 3; i++) {   // OK
            System.out.println(i);
        }
    }
}
```

---

### Example 7 — `var` in try-with-resources

```java
import java.io.*;

public class VarTryWith {
    public static void main(String[] args) throws IOException {
        try (var reader = new BufferedReader(new FileReader("test.txt"))) {
            var line = reader.readLine();   // String
            System.out.println(line);
        }
    }
}
```

---

### Example 8 — `final var`

```java
public class FinalVar {
    public static void main(String[] args) {
        final var x = 42;      // final int
        // x = 50;             // COMPILE ERROR — final

        final var name = "Alice";  // final String
        // name = "Bob";       // COMPILE ERROR — final

        System.out.println(x + " " + name);  // 42 Alice
    }
}
```

---

### Example 9 — `var` Infers Concrete Type

```java
import java.util.*;

public class VarConcreteType {
    public static void main(String[] args) {
        // var infers the concrete type, not the interface
        var list = new LinkedList<String>();   // LinkedList<String>, not List<String>
        list.addFirst("first");   // LinkedList-specific method
        list.addLast("last");     // LinkedList-specific method

        // vs. using interface type
        List<String> list2 = new LinkedList<String>();
        // list2.addFirst("first");  // COMPILE ERROR — List has no addFirst
    }
}
```

---

## 3. Exam-Style Questions

---

**Q1 — Select All That Apply**
In which of the following contexts can `var` be used? (Choose all that apply)

- A. Local variable in a method
- B. Instance field
- C. `for` loop variable
- D. Method parameter
- E. Enhanced `for-each` variable
- F. `try-with-resources` variable
- G. Method return type

**Answer: A, C, E, F**
B (fields), D (parameters), G (return types) are all illegal uses of `var`.

---

**Q2 — MCQ**
What is the inferred type of `x`?
```java
var x = new ArrayList<>();
```
- A. `List`
- B. `ArrayList`
- C. `ArrayList<Object>`
- D. `Object`

**Answer: C**
The diamond operator with no type context and `var` infers `ArrayList<Object>`.

---

**Q3 — MCQ**
Which line causes a compile error?
```java
var a = 42;           // Line 1
var b = 3.14;         // Line 2
var c = null;         // Line 3
var d = "hello";      // Line 4
```

- A. Line 1
- B. Line 2
- C. Line 3
- D. Line 4

**Answer: C**
`null` has no type — `var` cannot infer a type from `null`.

---

**Q4 — Select All That Apply**
Which of the following are compile errors? (Choose all that apply)

- A. `var x = 5; x = 10;`
- B. `var x = 5; x = "hello";`
- C. `var x; x = 5;`
- D. `var x = (String) null;`
- E. `var x = {1, 2, 3};`
- F. `final var x = 100;`

**Answer: B, C, E**
A: reassigning int to int — OK. B: reassigning int to String — ERROR. C: no initializer — ERROR. D: null with cast — OK. E: array initializer without `new` — ERROR. F: `final var` is legal.

---

**Q5 — MCQ**
```java
public class Test {
    public static void main(String[] args) {
        var list = new ArrayList<String>();
        list.add("hello");
        list.add("world");
        var size = list.size();
        System.out.println(size);
    }
}
```
What is the output?

- A. Compile error
- B. Runtime error
- C. `2`
- D. `[hello, world]`

**Answer: C**
`size` is inferred as `int`. `list.size()` returns 2. Prints `2`.

---

**Q6 — MCQ**
What is the inferred type of `x`?
```java
var x = 3.14f;
```
- A. `double`
- B. `float`
- C. `Float`
- D. Compile error

**Answer: B**
The `f` suffix makes `3.14f` a `float` literal. `x` is inferred as `float`.

---

**Q7 — MCQ**
```java
public class Test {
    public static void main(String[] args) {
        var x = "Hello";
        x = null;
        System.out.println(x);
    }
}
```
What is the result?

- A. Compile error — cannot assign null to var
- B. Compile error — x already typed as String
- C. Prints `null`
- D. NullPointerException

**Answer: C**
`x` is inferred as `String`. A `String` variable can hold `null`. Assigning `null` to `x` is legal. `println(null String reference)` prints `"null"`.

---

**Q8 — MCQ**
```java
public class Test {
    public static void main(String[] args) {
        var obj = new Object() {
            int value = 99;
        };
        System.out.println(obj.value);
    }
}
```
What is the result?

- A. Compile error — anonymous class members not accessible
- B. `99`
- C. `0`
- D. Runtime error

**Answer: B**
`var` captures the anonymous class type, making its members directly accessible. This is a unique feature of `var`.

---

**Q9 — MCQ**
Which of the following compiles without error?

- A. `var x = y -> y * 2;`
- B. `var x = new int[]{1, 2, 3};`
- C. `var x = {1, 2, 3};`
- D. `var x;`

**Answer: B**
A: lambda without target type — error. B: array with `new` — OK. C: array initializer without `new` — error. D: no initializer — error.

---

**Q10 — MCQ**
```java
for (var i = 0; i < 3; i++) {
    System.out.print(i + " ");
}
```
What is the inferred type of `i`?

- A. `Integer`
- B. `Object`
- C. `int`
- D. `long`

**Answer: C**
`0` is an `int` literal. `i` is inferred as `int`.

---

**Q11 — Tricky MCQ**
```java
var var = 5;
System.out.println(var);
```
What is the result?

- A. Compile error — `var` cannot be used as a variable name
- B. Compile error — `var var` is ambiguous
- C. Prints `5`
- D. Runtime error

**Answer: C**
`var` is a reserved type name, not a keyword. Using it as a variable name (`var var = 5`) is legal. The first `var` is the type inference keyword, the second is the variable name. This compiles and prints `5`.

---

## 4. Trick Analysis

### Trap 1: `var x = null` doesn't compile
The single most tested `var` restriction. `null` has no type so inference fails. The fix is `var x = (SomeType) null` which gives `null` a type the compiler can use.

---

### Trap 2: `var` with empty diamond infers `Object`
`var list = new ArrayList<>()` gives `ArrayList<Object>`, not `ArrayList<String>` or any other typed version. You must write `new ArrayList<String>()` explicitly when using `var`. The diamond operator needs a type context to infer from — with `var`, there's no left-hand type to look at.

---

### Trap 3: `var` infers concrete type, not interface
If you write `var x = new LinkedList<String>()`, `x` is `LinkedList<String>`, not `List<String>` or `Collection<String>`. This means you get access to `LinkedList`-specific methods like `addFirst`, `peekLast`, etc. This is sometimes desirable, sometimes a trap.

---

### Trap 4: Lambda and method references need a target type
`var f = x -> x + 1` is a compile error. Lambda expressions don't have a standalone type in Java — they acquire their type from context. With `var`, there's no explicit type context. You must write: `Function<Integer, Integer> f = x -> x + 1`.

---

### Trap 5: `var` is not `Object`
After `var x = "hello"`, you CANNOT do `x = 42`. `var` is not like writing `Object x` which could hold anything. The type is inferred and **locked** to `String`. The exam presents code reassigning a `var` to a different type and asks if it compiles — it doesn't.

---

### Trap 6: `var` cannot be used for fields — anywhere
Not instance fields, not static fields. Period. The exam embeds this in class declarations and tests whether you catch it. Only local variables get `var`.

---

### Trap 7: `var var = 5` is legal
Counterintuitive but true. `var` as a variable name is legal because `var` is not a keyword. The compiler disambiguates: the first `var` in a declaration statement is always the type inference keyword, the second is the variable name.

---

### Trap 8: `final var` is legal
Candidates assume `var` cannot be combined with modifiers. It can be combined with `final`. `final var x = 42` is a perfectly valid `final int` local variable.

---

## 5. Summary Sheet

### `var` at a Glance

| Property | Detail |
|---|---|
| Introduced | Java 10 |
| Type of feature | Reserved type name (NOT a keyword) |
| Type determined | Compile-time — statically typed |
| Runtime presence | None — zero overhead |
| Type locked after inference | Yes — cannot change type |

---

### Where `var` Is Allowed

| Context | Allowed? |
|---|---|
| Local variable in method | ✅ |
| `for` loop variable | ✅ |
| Enhanced `for-each` variable | ✅ |
| `try-with-resources` | ✅ |
| Instance field | ❌ |
| Static field | ❌ |
| Method parameter | ❌ |
| Constructor parameter | ❌ |
| Method return type | ❌ |
| Without initializer | ❌ |
| Initialized to `null` alone | ❌ |
| Lambda expression | ❌ |
| Method reference | ❌ |
| Array shorthand `{1,2,3}` | ❌ |

---

### Inference Rules

| Literal / Expression | Inferred Type |
|---|---|
| `42` | `int` |
| `42L` | `long` |
| `3.14` | `double` |
| `3.14f` | `float` |
| `'A'` | `char` |
| `true` / `false` | `boolean` |
| `"hello"` | `String` |
| `new ArrayList<String>()` | `ArrayList<String>` |
| `new ArrayList<>()` | `ArrayList<Object>` ⚠️ |
| `null` | **ERROR** ❌ |
| `(String) null` | `String` |
| `x -> x` | **ERROR** ❌ |
| `new int[]{1,2,3}` | `int[]` |
| `{1, 2, 3}` | **ERROR** ❌ |

---

### Key Rules to Memorize

| Rule | Detail |
|---|---|
| `var` = keyword? | No — reserved type name |
| `var` as variable name | Legal (bad practice) |
| `final var` | Legal |
| `var` + diamond `<>` | Infers `Object` — be explicit |
| `var` = dynamic typing? | No — type locked at compile time |
| `var` infers | Concrete type (not supertype/interface) |
| After inference | Cannot assign different type |

---
