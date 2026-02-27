# Topic 2.5 — Comparison Operators
## (`==`, `!=`, `instanceof`)

---

## 1. Conceptual Explanation

### What Are Comparison Operators?

Comparison operators evaluate two operands and return a `boolean` result. Java has two categories in this topic:

1. **Equality operators** — `==`, `!=`
2. **Type-check operator** — `instanceof`

These operators appear in virtually every OCP exam section — not just here. They interact with inheritance, generics, autoboxing, the String pool, and pattern matching (Java 16+). Mastering them here pays dividends throughout the entire exam.

---

### The `==` Operator

`==` compares the **values** of its two operands. What "value" means depends entirely on the operand types:

#### For Primitives: Compares Actual Values

```java
int a = 5, b = 5;
System.out.println(a == b);   // true — same numeric value

double x = 3.14, y = 3.14;
System.out.println(x == y);   // true

char c1 = 'A', c2 = 65;
System.out.println(c1 == c2); // true — 'A' has value 65
```

#### For References: Compares Memory Addresses

```java
String s1 = new String("hello");
String s2 = new String("hello");
System.out.println(s1 == s2);  // false — different objects in memory

String s3 = s1;
System.out.println(s1 == s3);  // true — same object reference
```

`==` on references does NOT compare object contents. It compares whether both variables point to the **exact same object** in memory. Use `.equals()` for content comparison.

---

### The `!=` Operator

`!=` is the exact logical inverse of `==`:

```java
int a = 5, b = 6;
System.out.println(a != b);    // true — different values

String s1 = new String("hi");
String s2 = new String("hi");
System.out.println(s1 != s2);  // true — different objects
```

---

### `==` with `null`

`null` can be compared with `==` and `!=` without throwing `NullPointerException`:

```java
String s = null;
System.out.println(s == null);   // true
System.out.println(s != null);   // false

String t = "hello";
System.out.println(t == null);   // false
```

> **Exam note:** `null == null` is `true`. Comparing two null references with `==` is perfectly valid and returns `true`.

```java
String a = null;
String b = null;
System.out.println(a == b);  // true — both are null (same "address": nothing)
```

---

### `==` and the String Pool

String literals are **interned** — the JVM keeps a pool of unique string literals and reuses them. This means string literals with the same content share the same object:

```java
String s1 = "hello";
String s2 = "hello";
System.out.println(s1 == s2);   // true — both from String pool, same object

String s3 = new String("hello");
System.out.println(s1 == s3);   // false — s3 forced new object off pool

String s4 = s3.intern();
System.out.println(s1 == s4);   // true — intern() returns pool reference
```

**Key rules:**
- String **literals** → always from pool → `==` may return `true` for same content
- `new String(...)` → always a **new object** → `==` always returns `false` vs pool
- String **concatenation of literals** at compile time → also from pool

```java
String s1 = "hello";
String s2 = "hel" + "lo";      // compile-time constant → pool → same as "hello"
System.out.println(s1 == s2);   // true

String part = "hel";
String s3 = part + "lo";        // runtime concatenation → new object
System.out.println(s1 == s3);   // false
```

> **Critical distinction:** Concatenation of two **string literals** or `final` string variables is a compile-time constant and goes to the pool. Concatenation involving non-final variables happens at runtime and creates a new object.

---

### `==` with Wrapper Types and Autoboxing — The Integer Cache

This is one of the most tested `==` scenarios on the OCP exam.

When Java autoboxes primitive values into wrapper objects, it **caches** commonly used values:

- `Integer` caches values **-128 to 127** (always)
- `Byte`, `Short`, `Long` cache **-128 to 127**
- `Character` caches **0 to 127**
- `Boolean` caches `true` and `false`
- `Float` and `Double` do **NOT** cache any values

```java
Integer a = 100;   // autoboxed — uses cached object
Integer b = 100;   // autoboxed — uses SAME cached object
System.out.println(a == b);  // true — same cached object

Integer c = 200;   // autoboxed — outside cache range
Integer d = 200;   // autoboxed — different object
System.out.println(c == d);  // false — different objects

Integer e = new Integer(100);   // explicit new — bypasses cache (deprecated API)
Integer f = 100;                 // autoboxed — from cache
System.out.println(e == f);  // false — new always creates new object
```

> **Exam trap:** The cache boundary of 127/128 is a classic OCP question. Values ≤ 127 → `==` returns `true` (cached). Values ≥ 128 → `==` returns `false` (not cached). The exam deliberately uses values just above and below 127.

```java
Integer x = 127;
Integer y = 127;
System.out.println(x == y);  // true — cached

Integer p = 128;
Integer q = 128;
System.out.println(p == q);  // false — not cached
```

---

### `==` Between Primitive and Wrapper — Unboxing

When you compare a primitive with a wrapper using `==`, the wrapper is **unboxed** to a primitive first:

```java
Integer boxed = 200;
int prim = 200;
System.out.println(boxed == prim);  // true — boxed unboxed to 200, then 200==200

Integer boxed2 = null;
// System.out.println(boxed2 == prim);  // NullPointerException! unboxing null
```

> **Exam trap:** Comparing a `null` wrapper with a primitive using `==` throws `NullPointerException` because Java tries to unbox `null`.

---

### Type Compatibility with `==`

You can only use `==` between types that are **compatible** — the compiler rejects comparisons between completely unrelated types:

```java
// Primitives — numeric promotion applies
int i = 5;
long l = 5L;
System.out.println(i == l);   // true — i promoted to long

// References — must be in same inheritance hierarchy
String s = "hello";
Integer n = 5;
// System.out.println(s == n);  // COMPILE ERROR — unrelated types

Object o = "hello";
String s2 = "hello";
System.out.println(o == s2);   // compiles — Object and String are related
                               // true if same pool object
```

**The compiler checks:** Can one type be cast to the other? If neither can be cast to the other (no inheritance relationship), `==` is a compile error.

```java
String s = "hello";
Object o = new Object();
System.out.println(s == o);    // compiles — String extends Object

// Interfaces are tricky — almost anything can be compared via interface:
Comparable c = "hello";
Runnable r = () -> {};
// System.out.println(c == r);  // Actually compiles — both could refer to same object
                               // that implements both interfaces
```

---

### The `instanceof` Operator

`instanceof` tests whether an object is an instance of a specific type (class, interface, or enum). It returns `boolean`.

**Syntax:**
```java
objectReference instanceof TypeName
```

```java
Object obj = "Hello";
System.out.println(obj instanceof String);   // true
System.out.println(obj instanceof Object);   // true — String IS-A Object
System.out.println(obj instanceof Integer);  // false — String is not Integer

String s = "Hello";
System.out.println(s instanceof String);     // true
System.out.println(s instanceof Comparable); // true — String implements Comparable
System.out.println(s instanceof Object);     // true — everything extends Object
```

---

### `instanceof` with `null`

`instanceof` **always returns `false`** when the left operand is `null` — it never throws `NullPointerException`:

```java
String s = null;
System.out.println(s instanceof String);   // false — null is not an instance of anything
System.out.println(s instanceof Object);   // false
System.out.println(null instanceof String); // false
```

> **Exam trap:** `null instanceof AnyType` is **always `false`**, never throws NPE. This is tested in every OCP. Contrast with calling a method on null which DOES throw NPE.

---

### `instanceof` Compile-Time Check

The compiler checks `instanceof` for **type compatibility**. If it's impossible for the left operand to ever be an instance of the right type, it's a compile error:

```java
String s = "hello";
System.out.println(s instanceof String);    // true — always true
System.out.println(s instanceof Integer);   // COMPILE ERROR — String can never be Integer
// String and Integer are both final classes with no relationship

// With interface — always compiles (runtime check needed):
System.out.println(s instanceof Runnable);  // false — but compiles!
// A String could theoretically be cast — it's a runtime check
```

**Rule:** If the left type is a class and the right type is a class with no inheritance relationship AND at least one is final, the compiler rejects it. If either type is an interface, the compiler generally allows it (assuming runtime might hold a class implementing both).

```java
String s = "hello";
// System.out.println(s instanceof Integer);   // COMPILE ERROR — String is final
// System.out.println(s instanceof Double);    // COMPILE ERROR — String is final

Integer i = 5;
// System.out.println(i instanceof String);    // COMPILE ERROR — Integer is final

Object o = new Object();
System.out.println(o instanceof String);   // false — compiles because Object is not final
```

---

### `instanceof` with Arrays

Arrays are objects — they can be checked with `instanceof`:

```java
int[] arr = new int[5];
System.out.println(arr instanceof Object);   // true — arrays extend Object

String[] strArr = new String[3];
System.out.println(strArr instanceof Object);       // true
System.out.println(strArr instanceof String[]);     // true
System.out.println(strArr instanceof Object[]);     // true — String[] IS-A Object[]
System.out.println(strArr instanceof Cloneable);    // true — arrays implement Cloneable
System.out.println(strArr instanceof Serializable); // true — arrays implement Serializable
```

---

### Pattern Matching for `instanceof` (Java 16+)

Java 16 introduced **pattern matching** for `instanceof`, allowing you to declare a binding variable in the same expression:

```java
// Old way (before Java 16)
Object obj = "Hello";
if (obj instanceof String) {
    String s = (String) obj;   // explicit cast needed
    System.out.println(s.length());
}

// New way — pattern matching (Java 16+)
Object obj = "Hello";
if (obj instanceof String s) {   // s is bound as String if condition is true
    System.out.println(s.length());  // s is directly available
}
```

**Rules for pattern matching `instanceof`:**

1. The binding variable (`s` above) is in scope only where the condition is known to be `true`
2. If the object is `null`, the pattern does NOT match and the binding variable is NOT bound
3. The binding variable is **effectively final** in its scope
4. The pattern variable's type must be a **subtype** of the tested reference type (or compiler warns/errors)

```java
Object obj = "Hello, World!";

if (obj instanceof String s && s.length() > 5) {
    // s is in scope here — both conditions must be true
    System.out.println(s.toUpperCase());  // HELLO, WORLD!
}

// Short-circuit — s used in same condition:
if (obj instanceof String s && s.startsWith("Hello")) {
    System.out.println("Starts with Hello");
}

// Cannot use s on the || right side:
if (obj instanceof String s || s.isEmpty()) {  // COMPILE ERROR — s not guaranteed bound
    // s might not be bound if instanceof is false
}
```

---

### Pattern Matching — Scope Nuances

```java
Object obj = "test";

// s in scope in if-body:
if (obj instanceof String s) {
    System.out.println(s);  // OK
}
// s NOT in scope here

// s in scope in else when negated:
if (!(obj instanceof String s)) {
    // s is NOT in scope here — condition is false means s not bound
} else {
    System.out.println(s);  // s IS in scope — condition was true
}
```

---

### Combining `instanceof` with `==`

Classic pattern — check type then compare:

```java
class Point {
    int x, y;
    Point(int x, int y) { this.x = x; this.y = y; }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;           // same reference
        if (!(obj instanceof Point)) return false; // null-safe type check
        Point other = (Point) obj;
        return this.x == other.x && this.y == other.y;
    }
}
```

With Java 16+ pattern matching:

```java
@Override
public boolean equals(Object obj) {
    if (this == obj) return true;
    if (!(obj instanceof Point other)) return false;  // pattern matching
    return this.x == other.x && this.y == other.y;   // other already cast
}
```

---

## 2. Code Examples

### Example 1 — `==` on Primitives vs References

```java
public class EqualityBasics {
    public static void main(String[] args) {
        // Primitives
        int a = 42, b = 42;
        System.out.println(a == b);   // true — same value

        // References — different objects
        String s1 = new String("hello");
        String s2 = new String("hello");
        System.out.println(s1 == s2);        // false — different objects
        System.out.println(s1.equals(s2));   // true — same content

        // References — same object
        String s3 = s1;
        System.out.println(s1 == s3);  // true — same reference

        // null comparisons
        String n = null;
        System.out.println(n == null);   // true
        System.out.println(null == null); // true
    }
}
```

---

### Example 2 — String Pool

```java
public class StringPool {
    public static void main(String[] args) {
        String a = "hello";
        String b = "hello";
        System.out.println(a == b);   // true — same pool object

        String c = new String("hello");
        System.out.println(a == c);   // false — c is off-pool

        String d = c.intern();
        System.out.println(a == d);   // true — intern returns pool reference

        // Compile-time concatenation → pool
        String e = "hel" + "lo";
        System.out.println(a == e);   // true — constant folded to "hello" at compile time

        // Runtime concatenation → new object
        String part = "hel";
        String f = part + "lo";
        System.out.println(a == f);   // false — runtime concat

        // Final variable → compile-time constant
        final String PART = "hel";
        String g = PART + "lo";
        System.out.println(a == g);   // true — PART is final constant
    }
}
```

---

### Example 3 — Integer Cache Boundaries

```java
public class IntegerCache {
    public static void main(String[] args) {
        // Values in cache range (-128 to 127)
        Integer a = 127;
        Integer b = 127;
        System.out.println(a == b);   // true — cached

        // Values outside cache range
        Integer c = 128;
        Integer d = 128;
        System.out.println(c == d);   // false — not cached

        // Boundary case
        Integer e = -128;
        Integer f = -128;
        System.out.println(e == f);   // true — -128 is in cache range

        Integer g = -129;
        Integer h = -129;
        System.out.println(g == h);   // false — outside cache range

        // Primitive vs wrapper — unboxing occurs
        int prim = 200;
        Integer box = 200;
        System.out.println(box == prim);  // true — box unboxed to 200

        // Null wrapper unboxed — NPE!
        Integer nullBox = null;
        // System.out.println(nullBox == prim);  // NullPointerException!
    }
}
```

---

### Example 4 — `instanceof` Fundamentals

```java
public class InstanceofDemo {
    public static void main(String[] args) {
        Object obj = "Hello";

        System.out.println(obj instanceof String);      // true
        System.out.println(obj instanceof Object);      // true
        System.out.println(obj instanceof Comparable);  // true
        System.out.println(obj instanceof Integer);     // false

        // null is never instanceof anything
        String s = null;
        System.out.println(s instanceof String);        // false — no NPE
        System.out.println(s instanceof Object);        // false

        // Array instanceof
        int[] arr = {1, 2, 3};
        System.out.println(arr instanceof Object);      // true
        System.out.println(arr instanceof Cloneable);   // true

        String[] sarr = {"a", "b"};
        System.out.println(sarr instanceof Object[]);   // true
    }
}
```

---

### Example 5 — Pattern Matching instanceof (Java 16+)

```java
public class PatternMatching {
    static void process(Object obj) {
        // Traditional
        if (obj instanceof Integer) {
            Integer i = (Integer) obj;
            System.out.println("Integer: " + i * 2);
        } else if (obj instanceof String) {
            String s = (String) obj;
            System.out.println("String: " + s.toUpperCase());
        } else {
            System.out.println("Other: " + obj);
        }
    }

    static void processModern(Object obj) {
        // Pattern matching
        if (obj instanceof Integer i) {
            System.out.println("Integer: " + i * 2);
        } else if (obj instanceof String s) {
            System.out.println("String: " + s.toUpperCase());
        } else {
            System.out.println("Other: " + obj);
        }
    }

    public static void main(String[] args) {
        process(42);          // Integer: 84
        process("hello");     // String: HELLO
        process(3.14);        // Other: 3.14

        processModern(42);    // Integer: 84
        processModern("hello"); // String: HELLO
    }
}
```

---

### Example 6 — Pattern Variable Scope

```java
public class PatternScope {
    public static void main(String[] args) {
        Object obj = "Hello World";

        // Scope in if-body
        if (obj instanceof String s) {
            System.out.println(s.length());   // 11 — s in scope
        }
        // s NOT in scope here

        // Scope with && — short-circuit ensures both conditions
        if (obj instanceof String s && s.length() > 5) {
            System.out.println("Long string: " + s);  // s in scope
        }

        // Negated — s in scope in else branch
        if (!(obj instanceof String s)) {
            System.out.println("Not a string");
        } else {
            System.out.println("Is string: " + s);   // s in scope here
        }

        // Cannot use in || right side
        // if (obj instanceof String s || s.isEmpty()) { }  // COMPILE ERROR
    }
}
```

---

### Example 7 — `==` Type Compatibility

```java
public class TypeCompatibility {
    public static void main(String[] args) {
        // Same hierarchy — compiles
        Object o = "hello";
        String s = "hello";
        System.out.println(o == s);   // true (same pool object)

        // Unrelated final classes — compile error:
        // String str = "hi";
        // Integer num = 5;
        // System.out.println(str == num);  // COMPILE ERROR

        // Interface — compiles (runtime check)
        Comparable c = "hello";
        Runnable r = () -> {};
        // c and r could theoretically be the same object implementing both
        System.out.println(c == r);   // false — different objects (compiles!)

        // null with any reference type
        String nullStr = null;
        System.out.println(nullStr == null);  // true
        System.out.println(null == null);     // true
    }
}
```

---

### Example 8 — equals() Contract

```java
public class EqualsVsDoubleEquals {
    public static void main(String[] args) {
        // String
        String s1 = new String("hello");
        String s2 = new String("hello");
        System.out.println(s1 == s2);        // false — different objects
        System.out.println(s1.equals(s2));   // true — same content

        // Integer
        Integer i1 = 200;
        Integer i2 = 200;
        System.out.println(i1 == i2);        // false — not cached
        System.out.println(i1.equals(i2));   // true — same value

        // Object (no override)
        Object o1 = new Object();
        Object o2 = new Object();
        System.out.println(o1 == o2);        // false
        System.out.println(o1.equals(o2));   // false — Object.equals uses ==
    }
}
```

---

## 3. Exam-Style Questions

---

**Q1 — MCQ**
What is the output?
```java
Integer a = 127;
Integer b = 127;
Integer c = 128;
Integer d = 128;
System.out.println(a == b);
System.out.println(c == d);
```
- A. `true` / `true`
- B. `true` / `false`
- C. `false` / `false`
- D. `false` / `true`

**Answer: B**
127 is within Integer cache range → same object → `true`. 128 is outside cache range → different objects → `false`.

---

**Q2 — MCQ**
What is the output?
```java
String s = null;
System.out.println(s instanceof String);
System.out.println(s == null);
```
- A. `NullPointerException`
- B. `true` / `true`
- C. `false` / `true`
- D. `false` / `false`

**Answer: C**
`null instanceof String` is always `false` — no NPE. `s == null` is `true`.

---

**Q3 — MCQ**
What is the output?
```java
String s1 = "hello";
String s2 = "hel" + "lo";
String part = "hel";
String s3 = part + "lo";
System.out.println(s1 == s2);
System.out.println(s1 == s3);
```
- A. `true` / `true`
- B. `true` / `false`
- C. `false` / `false`
- D. `false` / `true`

**Answer: B**
`"hel"+"lo"` is compile-time constant → pool → same as `s1` → `true`. `part` is non-final, `part+"lo"` is runtime → new object → `false`.

---

**Q4 — Select All That Apply**
Which statements are true about `instanceof`? (Choose all that apply)

- A. `null instanceof Object` returns `false`
- B. `null instanceof Object` throws `NullPointerException`
- C. `"hello" instanceof String` is always `true`
- D. `"hello" instanceof Integer` is a compile error
- E. An array reference is `instanceof Object`
- F. `instanceof` can only be used with class types, not interfaces

**Answer: A, C, D, E**
B is false — `instanceof` never throws NPE for null. F is false — `instanceof` works with interfaces. A, C, D, E are all correct.

---

**Q5 — MCQ**
```java
Integer x = null;
int y = 5;
System.out.println(x == y);
```
- A. `false`
- B. `true`
- C. `NullPointerException`
- D. Compile error

**Answer: C**
Comparing `Integer` with `int` causes unboxing of `x`. Unboxing `null` → `NullPointerException`.

---

**Q6 — MCQ**
What is the output?
```java
Object obj = "Hello";
if (obj instanceof String s && s.length() > 3) {
    System.out.println(s.toUpperCase());
} else {
    System.out.println("No match");
}
```
- A. `HELLO`
- B. `Hello`
- C. `No match`
- D. Compile error

**Answer: A**
`obj` is a `String` → pattern matches. `s = "Hello"`. `s.length() = 5 > 3`. Prints `"HELLO"`.

---

**Q7 — Select All That Apply**
Which produce compile errors? (Choose all that apply)
```java
A. String s = "hi"; System.out.println(s instanceof String);
B. String s = "hi"; System.out.println(s instanceof Integer);
C. Object o = new Object(); System.out.println(o instanceof String);
D. String s = "hi"; Integer i = 5; System.out.println(s == i);
E. Integer a = 5; int b = 5; System.out.println(a == b);
F. Comparable c = "hi"; Runnable r = ()->{};System.out.println(c == r);
```

**Answer: B, D**
B: `String` is final, can never be `Integer` — compile error.
D: `String` and `Integer` are unrelated final classes — compile error.
A: `s instanceof String` — always `true`, but compiles.
C: `Object` is not final, could hold a `String` at runtime — compiles.
E: `Integer` vs `int` — unboxing occurs, compiles.
F: Both are interface types, could theoretically be same object — compiles.

---

**Q8 — MCQ**
```java
final String A = "hel";
String B = "hel";
String s1 = A + "lo";
String s2 = B + "lo";
String literal = "hello";
System.out.println(literal == s1);
System.out.println(literal == s2);
```
- A. `true` / `true`
- B. `false` / `false`
- C. `true` / `false`
- D. `false` / `true`

**Answer: C**
`A` is `final` → `A + "lo"` is a compile-time constant → from pool → `true`.
`B` is non-final → `B + "lo"` is runtime concatenation → new object → `false`.

---

**Q9 — MCQ**
```java
Object obj = Integer.valueOf(42);
if (obj instanceof Integer i) {
    System.out.println(i + 10);
} else {
    System.out.println("not integer");
}
```
- A. `42`
- B. `52`
- C. `not integer`
- D. Compile error

**Answer: B**
`obj` holds an `Integer`. Pattern matches → `i = 42`. `i + 10 = 52`.

---

**Q10 — MCQ**
What is the output?
```java
Integer a = -128;
Integer b = -128;
Integer c = -129;
Integer d = -129;
System.out.println(a == b);
System.out.println(c == d);
```
- A. `true` / `true`
- B. `false` / `false`
- C. `true` / `false`
- D. `false` / `true`

**Answer: C**
`-128` IS in the Integer cache range (cache is -128 to 127) → same object → `true`.
`-129` is outside cache range → different objects → `false`.

---

**Q11 — MCQ**
```java
String s = null;
String t = null;
System.out.println(s == t);
System.out.println(s == null);
System.out.println(null == null);
```
- A. `true` / `true` / `true`
- B. `false` / `true` / `true`
- C. NullPointerException
- D. `false` / `false` / `true`

**Answer: A**
All three are `true`. `null == null` is always `true`. Comparing null references never throws NPE.

---

## 4. Trick Analysis

### Trap 1: Integer cache boundary 127/128
Values -128 to 127 use cached Integer objects. `Integer a = 127; Integer b = 127; a == b` is `true`. `Integer a = 128; Integer b = 128; a == b` is `false`. The exam uses exactly 127 and 128 to test this boundary. ALWAYS use `.equals()` for Integer comparison to avoid this trap.

---

### Trap 2: `null instanceof X` is always `false`, never NPE
Unlike method calls, `null instanceof X` never throws NPE — it returns `false`. This is by design: null is not an instance of any type. The exam presents `null instanceof SomeClass` and offers NPE as an option — it's always `false`.

---

### Trap 3: `String` literal `+` non-final variable = runtime, not pool
`"hel" + "lo"` → compile-time constant → pool. `part + "lo"` where `part` is non-final → runtime → new object. The difference is invisible in the code until you add `final`. Many candidates assume all string concatenation goes to the pool.

---

### Trap 4: `final String` variable → compile-time constant
`final String PART = "hel"; String s = PART + "lo";` — because `PART` is final, the concatenation is a compile-time constant and goes to the String pool. Without `final`, same code creates a new object.

---

### Trap 5: `null == null` is `true`
Two null references compared with `==` return `true`. This is logically consistent (both have "no address") but surprises some candidates. The exam includes `null == null` as an expression and asks what it evaluates to.

---

### Trap 6: Unboxing `null` wrapper throws NPE
`Integer i = null; int j = 5; i == j` → NPE. When comparing a wrapper with a primitive, Java unboxes the wrapper. Unboxing `null` throws NullPointerException. The exam presents this as a test of whether candidates know about unboxing.

---

### Trap 7: `String s = "hi"; s instanceof Integer` — compile error
`String` is `final` and has no relationship with `Integer` (also `final`). The compiler knows `s` can NEVER be an `Integer`. Compile error. But `Object o = ...; o instanceof String` compiles because `Object` is not final and could hold a `String` at runtime.

---

### Trap 8: Pattern variable scope with `||` vs `&&`
With `&&`: both sides must be true, so the pattern variable IS bound on the right side.
With `||`: either side could be true, so the pattern variable is NOT guaranteed to be bound — compile error if used on the right side of `||`.

```java
if (obj instanceof String s && s.length() > 0) { }  // OK
if (obj instanceof String s || s.isEmpty()) { }      // COMPILE ERROR
```

---

### Trap 9: `==` on `String` literals of same content
`"hello" == "hello"` is `true` because string literals are interned. But `new String("hello") == "hello"` is `false` because `new String(...)` forces a new heap object. The exam mixes these forms.

---

### Trap 10: `instanceof` compile check for final classes
If the left side is declared as a `final` class and the right side is another `final` class with no inheritance relationship, the compiler rejects it at compile time (knows it can never be true). But if the left side is declared as a non-final class or interface, the compiler allows it (must check at runtime).

---

## 5. Summary Sheet

### `==` Operator Behavior

| Operand Types | What Is Compared | Result |
|---|---|---|
| Both primitives | Actual values | Numeric equality |
| Both references | Memory addresses | Same object? |
| Mixed (wrapper + primitive) | Unboxes wrapper → compares values | Value equality (NPE if null) |
| null == null | N/A | Always `true` |
| Any reference == null | Is it null? | `true` if null |

---

### String Pool Rules

| Expression | Pool? | `== "hello"` result |
|---|---|---|
| `"hello"` | ✅ Pool | `true` |
| `"hel" + "lo"` | ✅ Pool (compile-time constant) | `true` |
| `new String("hello")` | ❌ Heap | `false` |
| `part + "lo"` (non-final `part`) | ❌ Heap (runtime) | `false` |
| `PART + "lo"` (final `PART`) | ✅ Pool (compile-time constant) | `true` |
| `s.intern()` | ✅ Returns pool reference | `true` |

---

### Integer Cache Boundaries

| Type | Cache Range | Outside Range |
|---|---|---|
| `Byte` | -128 to 127 (ALL values) | N/A |
| `Short` | -128 to 127 | Different objects |
| `Integer` | **-128 to 127** | Different objects |
| `Long` | -128 to 127 | Different objects |
| `Character` | 0 to 127 | Different objects |
| `Boolean` | `true` and `false` (both) | N/A |
| `Float` | None | Always different |
| `Double` | None | Always different |

---

### `instanceof` Key Rules

| Rule | Detail |
|---|---|
| `null instanceof X` | Always `false` — never NPE |
| `x instanceof Object` | Always `true` for any non-null reference |
| Compile-time check | If types are provably incompatible → compile error |
| Final class vs unrelated final class | Compile error |
| Non-final class vs anything | Compiles — runtime check |
| Pattern matching (`instanceof T t`) | `t` in scope only where condition is `true` |
| Pattern + `&&` | Pattern variable usable on right side of `&&` |
| Pattern + `\|\|` | Pattern variable NOT usable on right side of `\|\|` |
| Arrays instanceof Object | `true` — arrays extend Object |
| Arrays instanceof Cloneable | `true` |
| Arrays instanceof Serializable | `true` |

---

### `==` Type Compatibility Rules

| Comparison | Compiles? | Why |
|---|---|---|
| `String == String` | ✅ | Same type |
| `String == Object` | ✅ | Related hierarchy |
| `String == Integer` | ❌ | Both final, no relationship |
| `Object == Integer` | ✅ | Object is not final |
| `Comparable == Runnable` | ✅ | Both interfaces — same object possible |
| `String == Runnable` | ✅ | String could implement Runnable? No, but compiler allows interface comparison |
| Primitive == Wrapper | ✅ | Unboxing occurs |
| `null == null` | ✅ | Always valid, returns `true` |

---

### Pattern Matching Quick Reference

```java
// Basic
if (obj instanceof String s) { /* s is String */ }

// With condition
if (obj instanceof String s && s.length() > 0) { }

// Negated — s available in else
if (!(obj instanceof String s)) { } else { /* s available */ }

// Switch (Java 17 preview — covered in Topic 2.12)
switch (obj) {
    case String s -> System.out.println(s.length());
    case Integer i -> System.out.println(i * 2);
}
```

---
