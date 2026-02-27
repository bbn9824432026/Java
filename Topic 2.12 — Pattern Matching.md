# Topic 2.12 — Pattern Matching
## (`instanceof` and `switch`)

---

## 1. Conceptual Explanation

### What Is Pattern Matching?

Pattern matching is a feature that combines **type testing** and **variable binding** into a single concise operation. Instead of testing a type with `instanceof` and then separately casting, pattern matching does both in one step.

Java SE 17 includes two forms of pattern matching:
1. **Pattern matching for `instanceof`** — standard in Java 16, fully available in Java 17
2. **Pattern matching for `switch`** — **preview** in Java 17 (requires `--enable-preview`)

Both are tested on OCP Java SE 17. The `instanceof` form is standard; the `switch` form is preview but examinable.

---

### Pattern Matching for `instanceof` — Recap and Deep Dive

Introduced as preview in Java 14, standard in **Java 16**.

**Traditional approach (pre-Java 16):**
```java
Object obj = "Hello";
if (obj instanceof String) {
    String s = (String) obj;   // explicit cast required
    System.out.println(s.length());
}
```

**Pattern matching approach:**
```java
Object obj = "Hello";
if (obj instanceof String s) {   // type test + binding in one step
    System.out.println(s.length());  // s available — already a String
}
```

The variable `s` is called the **pattern variable** or **binding variable**. It is:
- Declared and initialized automatically if the pattern matches
- Scoped only to where the pattern is guaranteed to have matched
- **Effectively final** — it cannot be reassigned

---

### Pattern Variable Scope — The Definite Assignment Rules

The compiler tracks exactly where the pattern is guaranteed to have matched and makes the binding variable available only there. This is called **flow-sensitive typing**.

```java
Object obj = "Hello";

// Pattern variable 's' in scope in if-body:
if (obj instanceof String s) {
    System.out.println(s.toUpperCase());  // OK — s guaranteed to be String here
}
// s NOT in scope here

// Pattern variable available with &&:
if (obj instanceof String s && s.length() > 3) {
    // Both conditions must be true for body to execute
    // s is in scope on right side of && (short-circuit guarantees match)
    System.out.println(s);
}

// Pattern variable NOT available with ||:
// if (obj instanceof String s || s.isEmpty()) { }  // COMPILE ERROR
// s not guaranteed bound if instanceof is false
```

---

### Negated Pattern — `else` Branch Scope

When the pattern is negated with `!`, the pattern variable is available in the **`else` branch** (or after the `if` when negated):

```java
Object obj = "Hello";

if (!(obj instanceof String s)) {
    // s NOT in scope — pattern didn't match
    System.out.println("Not a String");
} else {
    // s IS in scope — pattern DID match
    System.out.println(s.toUpperCase());
}
```

And with early return (guard pattern):

```java
void process(Object obj) {
    if (!(obj instanceof String s)) {
        return;   // early exit if not String
    }
    // s IS in scope here — execution only reaches here if pattern matched
    System.out.println(s.toUpperCase());
}
```

---

### Scope with `&&` and `||` — Detailed Rules

**`&&` (AND) — pattern variable available on right side:**
```java
// s available on right side of && because:
// if instanceof is false, && short-circuits and body doesn't execute
// if we reach right side, instanceof was true and s is bound
if (obj instanceof String s && s.startsWith("H")) {
    System.out.println(s);  // s also in scope here
}
```

**`||` (OR) — pattern variable NOT available on right side:**
```java
// COMPILE ERROR:
// if instanceof is false, || evaluates right side — but s is not bound!
if (obj instanceof String s || s.isEmpty()) { }  // ERROR

// But negated pattern with || is OK:
if (!(obj instanceof String s) || s.isEmpty()) {
    // Hmm — if negation is true (not String), s.isEmpty() would fail
    // COMPILE ERROR still — s not guaranteed bound on right side of ||
}
```

Actually the precise rule:

- **`A && B`**: if A is `obj instanceof T t`, then `t` is in scope in B
- **`A || B`**: if A is `!(obj instanceof T t)`, then `t` is in scope in B (negated pattern flows to right of ||)

```java
// Negated pattern with || — valid:
if (!(obj instanceof String s) || s.length() > 0) {
    // If not a String (negated true): s not bound but || short-circuits to true
    // Wait — s would be used on right side... COMPILE ERROR still in Java 17
}
```

> For OCP 17, the safe rule is: pattern variable available **inside `&&` right side** and **inside if-body when non-negated**, and **inside else/after when negated with `!`**.

---

### Pattern Matching Cannot Be Used with `==`

Pattern variables must be in a binding context — you cannot use them with equality or other operators outside the `if` body:

```java
// No pattern variable outside if scope:
if (obj instanceof String s) {
    System.out.println(s);
}
// s not accessible here
```

---

### Pattern Variable Is Effectively Final

The pattern variable cannot be reassigned:

```java
if (obj instanceof String s) {
    // s = "other";   // COMPILE ERROR — pattern variable is effectively final
    System.out.println(s);
}
```

---

### Pattern Matching Does Not Work with Primitive Types

Pattern matching `instanceof` works only with **reference types**:

```java
Object obj = 42;
if (obj instanceof int i) { }     // COMPILE ERROR — int is primitive
if (obj instanceof Integer i) { } // OK — Integer is reference type
```

---

### Shadowing with Pattern Variables

Pattern variables can shadow outer variables:

```java
String s = "outer";
Object obj = "inner";
if (obj instanceof String s) {   // shadows outer 's'
    System.out.println(s);       // "inner" — pattern variable
}
System.out.println(s);           // "outer" — outer variable
```

---

### Pattern Matching for `switch` (Preview in Java 17)

Pattern matching for `switch` allows case labels to use type patterns, not just constants. This is a **preview feature** in Java 17 — requires `--enable-preview` to compile/run.

**Traditional switch limitations:**
```java
// Before: must use if-else chains for type checking
Object obj = getSomeObject();
String result;
if (obj instanceof Integer i) {
    result = "int: " + i;
} else if (obj instanceof String s) {
    result = "str: " + s;
} else if (obj instanceof Double d) {
    result = "dbl: " + d;
} else {
    result = "other";
}
```

**With pattern matching switch (Java 17 preview):**
```java
Object obj = getSomeObject();
String result = switch (obj) {
    case Integer i -> "int: " + i;
    case String s  -> "str: " + s;
    case Double d  -> "dbl: " + d;
    default        -> "other";
};
```

---

### Pattern Matching Switch — Key Rules

**1. Order of cases matters for type patterns:**

Unlike constant switch (where order doesn't matter for correctness), with type patterns, more specific types must come before more general types:

```java
Object obj = "hello";
String result = switch (obj) {
    case String s  -> "String: " + s;   // more specific — must come first
    case Object o  -> "Object: " + o;   // less specific — comes after
    // COMPILE ERROR if Object came first — String case would be unreachable
};
```

---

**2. Dominance check — compile error for unreachable patterns:**

```java
// COMPILE ERROR — second case is dominated by first:
switch (obj) {
    case Object o -> "any object";
    case String s -> "string";  // ERROR — always matched by Object above
}
```

---

**3. `null` handling in pattern switch:**

In Java 17 preview, `switch` with pattern matching can handle `null` explicitly:

```java
String result = switch (obj) {
    case null      -> "null value";
    case String s  -> "String: " + s;
    default        -> "other";
};
// No NPE even if obj is null — null case handles it
```

Without a `null` case, passing `null` still throws `NullPointerException`.

---

**4. Guarded patterns with `when` (Java 17 preview calls them guards):**

In Java 17 preview, guards use `when` keyword:

```java
String result = switch (obj) {
    case Integer i when i > 0 -> "positive int: " + i;
    case Integer i            -> "non-positive int: " + i;
    case String s  when s.length() > 5 -> "long string: " + s;
    case String s             -> "short string: " + s;
    default                   -> "other";
};
```

> **Note:** In Java 17 preview, the keyword is `when`. Earlier preview versions used `&&`. For OCP 17, know both syntaxes may appear.

---

**5. Exhaustiveness in pattern switch:**

A pattern switch expression must be exhaustive. Unlike constant switch, a `default` or total pattern (like `case Object o`) achieves exhaustiveness:

```java
// Exhaustive — default covers remaining:
String r = switch (obj) {
    case String s -> s;
    default -> "not string";
};

// Exhaustive — sealed hierarchy covered (Java 17 sealed classes):
sealed interface Shape permits Circle, Rectangle, Triangle {}

String describe = switch (shape) {
    case Circle c    -> "circle";
    case Rectangle r -> "rectangle";
    case Triangle t  -> "triangle";
    // No default needed — all permitted types covered
};
```

---

### Pattern Matching with Sealed Classes and Interfaces

Sealed classes (standard in Java 17) work powerfully with pattern matching switch because the compiler knows all permitted subtypes:

```java
sealed interface Shape permits Circle, Rectangle {}
record Circle(double radius) implements Shape {}
record Rectangle(double width, double height) implements Shape {}

Shape shape = new Circle(5.0);

double area = switch (shape) {
    case Circle c    -> Math.PI * c.radius() * c.radius();
    case Rectangle r -> r.width() * r.height();
    // No default needed — Shape is sealed, all subtypes covered
};
System.out.println(area);
```

---

### Pattern Matching `instanceof` vs Traditional — Performance

Pattern matching is not just syntactic sugar. The JVM can optimize type checks and casts together. However, the behavioral difference for the exam is purely about:
- Compile-time scope rules
- Null safety (pattern never matches null)
- Effectively final binding variable

---

### Combining Multiple Patterns

```java
void processAll(List<Object> items) {
    for (Object item : items) {
        if (item instanceof Integer i && i > 0) {
            System.out.println("Positive integer: " + i);
        } else if (item instanceof String s && !s.isEmpty()) {
            System.out.println("Non-empty string: " + s);
        } else if (item instanceof Double d) {
            System.out.println("Double: " + d);
        } else {
            System.out.println("Other: " + item);
        }
    }
}
```

---

### `instanceof` Pattern vs Cast — Behavioral Equivalence

These two are functionally equivalent:

```java
// Pattern matching:
if (obj instanceof String s) {
    System.out.println(s.length());
}

// Equivalent traditional:
if (obj instanceof String) {
    String s = (String) obj;
    System.out.println(s.length());
}
```

The pattern form is preferred because:
- Eliminates redundant `instanceof` + cast
- Reduces ClassCastException risk
- Cleaner code — no separate declaration needed

---

## 2. Code Examples

### Example 1 — Pattern Matching Basics

```java
public class PatternBasics {
    public static void main(String[] args) {
        Object[] objects = {42, "hello", 3.14, null, true};

        for (Object obj : objects) {
            if (obj instanceof Integer i) {
                System.out.println("Integer: " + i * 2);
            } else if (obj instanceof String s) {
                System.out.println("String: " + s.toUpperCase());
            } else if (obj instanceof Double d) {
                System.out.println("Double: " + d);
            } else {
                System.out.println("Other: " + obj);
            }
        }
        // Output:
        // Integer: 84
        // String: HELLO
        // Double: 3.14
        // Other: null    ← null matches no pattern — falls to else
        // Other: true
    }
}
```

---

### Example 2 — Scope Rules

```java
public class PatternScope {
    public static void main(String[] args) {
        Object obj = "Hello World";

        // s in scope inside if body:
        if (obj instanceof String s) {
            System.out.println(s.length());   // 11
        }
        // s out of scope here

        // s available in && right side:
        if (obj instanceof String s && s.contains(" ")) {
            System.out.println("Has space: " + s);
        }

        // Negated — s available in else:
        if (!(obj instanceof String s)) {
            System.out.println("Not string");
        } else {
            System.out.println("Is string: " + s);  // s in scope
        }

        // Early return guard pattern:
        processObject(obj);
        processObject(42);
    }

    static void processObject(Object obj) {
        if (!(obj instanceof String s)) {
            System.out.println("Skipping non-string");
            return;
        }
        // s available here — execution guaranteed to be past the check
        System.out.println("Processing: " + s.toLowerCase());
    }
}
```

---

### Example 3 — Pattern Variable Is Effectively Final

```java
public class PatternFinal {
    public static void main(String[] args) {
        Object obj = "hello";

        if (obj instanceof String s) {
            System.out.println(s);   // "hello"

            // Cannot reassign pattern variable:
            // s = "world";   // COMPILE ERROR

            // But can use s in calculations:
            String upper = s.toUpperCase();
            System.out.println(upper);
        }

        // Shadowing:
        String s = "outer";
        if (obj instanceof String s) {   // shadows outer 's'
            System.out.println(s);       // pattern variable: "hello"
        }
        System.out.println(s);           // outer variable: "outer"
    }
}
```

---

### Example 4 — Pattern with Sealed Classes

```java
public class SealedPattern {
    sealed interface Vehicle permits Car, Truck, Motorcycle {}
    record Car(String brand, int seats) implements Vehicle {}
    record Truck(String brand, double payload) implements Vehicle {}
    record Motorcycle(String brand, boolean hasSidecar) implements Vehicle {}

    static String describe(Vehicle v) {
        return switch (v) {
            case Car c       -> c.brand() + " car with " + c.seats() + " seats";
            case Truck t     -> t.brand() + " truck, payload: " + t.payload() + "t";
            case Motorcycle m -> m.brand() + (m.hasSidecar() ? " with sidecar" : "");
            // No default needed — all sealed subtypes covered
        };
    }

    public static void main(String[] args) {
        System.out.println(describe(new Car("Toyota", 5)));
        System.out.println(describe(new Truck("Ford", 2.5)));
        System.out.println(describe(new Motorcycle("Honda", false)));
    }
}
```

---

### Example 5 — Guarded Patterns with `when`

```java
public class GuardedPatterns {
    static String classify(Object obj) {
        return switch (obj) {
            case Integer i when i < 0    -> "negative int: " + i;
            case Integer i when i == 0   -> "zero";
            case Integer i               -> "positive int: " + i;
            case String s  when s.isEmpty() -> "empty string";
            case String s                -> "string: " + s;
            case null                    -> "null";
            default                      -> "other: " + obj;
        };
    }

    public static void main(String[] args) {
        System.out.println(classify(-5));      // negative int: -5
        System.out.println(classify(0));       // zero
        System.out.println(classify(42));      // positive int: 42
        System.out.println(classify(""));      // empty string
        System.out.println(classify("hi"));    // string: hi
        System.out.println(classify(null));    // null
        System.out.println(classify(3.14));    // other: 3.14
    }
}
```

---

### Example 6 — Pattern Dominance (Compile Error Examples)

```java
public class PatternDominance {
    public static void main(String[] args) {
        Object obj = "test";

        // This would be COMPILE ERROR — Object dominates String:
        /*
        String r = switch (obj) {
            case Object o -> "any";
            case String s -> "string";   // ERROR — unreachable, Object matches first
        };
        */

        // Correct order — specific before general:
        String r = switch (obj) {
            case String s  -> "string: " + s;   // more specific
            case Object o  -> "other: " + o;    // less specific (fallback)
        };
        System.out.println(r);  // string: test

        // With guarded patterns — more specific guard first:
        String r2 = switch (obj) {
            case String s when s.length() > 3 -> "long: " + s;
            case String s -> "short: " + s;       // catches remaining Strings
            default       -> "not string";
        };
        System.out.println(r2);  // long: test (length 4 > 3)
    }
}
```

---

### Example 7 — Null Handling in Pattern Switch

```java
public class NullInPatternSwitch {
    static String handle(Object obj) {
        // Without null case — null throws NPE:
        // return switch (obj) { case String s -> s; default -> "other"; };

        // With null case — safe:
        return switch (obj) {
            case null      -> "null input";
            case String s  -> "string: " + s;
            case Integer i -> "int: " + i;
            default        -> "other: " + obj;
        };
    }

    public static void main(String[] args) {
        System.out.println(handle(null));      // null input
        System.out.println(handle("hello"));   // string: hello
        System.out.println(handle(42));        // int: 42
        System.out.println(handle(3.14));      // other: 3.14
    }
}
```

---

### Example 8 — Pattern Matching in `equals()` Implementation

```java
public class PatternInEquals {
    static class Point {
        final int x, y;

        Point(int x, int y) {
            this.x = x;
            this.y = y;
        }

        @Override
        public boolean equals(Object obj) {
            // Traditional:
            // if (!(obj instanceof Point)) return false;
            // Point other = (Point) obj;

            // Pattern matching:
            return obj instanceof Point other
                && this.x == other.x
                && this.y == other.y;
            // Null-safe: null instanceof Point is false — returns false
        }

        @Override
        public int hashCode() {
            return 31 * x + y;
        }
    }

    public static void main(String[] args) {
        Point p1 = new Point(1, 2);
        Point p2 = new Point(1, 2);
        Point p3 = new Point(3, 4);

        System.out.println(p1.equals(p2));  // true
        System.out.println(p1.equals(p3));  // false
        System.out.println(p1.equals(null)); // false — no NPE
        System.out.println(p1.equals("hi")); // false
    }
}
```

---

## 3. Exam-Style Questions

---

**Q1 — MCQ**
What is the output?
```java
Object obj = "Hello";
if (obj instanceof String s && s.length() > 3) {
    System.out.println(s.toUpperCase());
} else {
    System.out.println("no match");
}
```
- A. `HELLO`
- B. `Hello`
- C. `no match`
- D. Compile error

**Answer: A**
`obj` is `String` → pattern matches, `s = "Hello"`. `s.length() = 5 > 3` → true. Prints `"HELLO"`.

---

**Q2 — MCQ**
What is the output?
```java
Object obj = null;
if (obj instanceof String s) {
    System.out.println("String: " + s);
} else {
    System.out.println("not String");
}
```
- A. `String: null`
- B. `not String`
- C. `NullPointerException`
- D. Compile error

**Answer: B**
`null instanceof String` is always `false` — no NPE. Falls to `else`. Prints `"not String"`.

---

**Q3 — MCQ**
Which line causes a compile error?
```java
Object obj = "test";
if (obj instanceof String s) {
    System.out.println(s);           // Line A
    s = "other";                     // Line B
    String t = s.toUpperCase();      // Line C
}
System.out.println(s);               // Line D
```
- A. Line A
- B. Line B
- C. Line C
- D. Line D

**Answer: B and D**
Line B: pattern variable `s` is effectively final — cannot be reassigned → compile error.
Line D: `s` is out of scope outside the `if` block → compile error.

If only one answer is required by the question, **Line B** is the primary compile error.

---

**Q4 — Select All That Apply**
Which are true about pattern variables in `instanceof` pattern matching?

- A. The pattern variable is in scope inside the `if` body when the pattern matches
- B. The pattern variable is effectively final
- C. The pattern variable can be `null` if the object is `null`
- D. The pattern variable is in scope on the right side of `&&`
- E. The pattern variable is in scope on the right side of `||`
- F. Pattern matching works with primitive types like `int`

**Answer: A, B, D**
C: false — `null` never matches a pattern, so pattern variable is never null. E: false — right side of `||` is not guaranteed to have the pattern match. F: false — only reference types.

---

**Q5 — MCQ**
```java
Object obj = 42;
String result = switch (obj) {
    case Integer i when i > 0 -> "positive";
    case Integer i            -> "non-positive";
    default                   -> "other";
};
System.out.println(result);
```
- A. `positive`
- B. `non-positive`
- C. `other`
- D. Compile error

**Answer: A**
`obj` is `Integer` 42, which is > 0 → first case matches → `"positive"`.

---

**Q6 — MCQ**
```java
Object obj = "hello";
if (obj instanceof String s || s.isEmpty()) {
    System.out.println(s);
}
```
- A. `hello`
- B. `false`
- C. Compile error
- D. NullPointerException

**Answer: C**
`s` is not guaranteed to be bound on the right side of `||` — if `instanceof` is false, `s` is not bound but `s.isEmpty()` would try to use it. Compile error.

---

**Q7 — MCQ**
What is the output?
```java
void process(Object obj) {
    if (!(obj instanceof String s)) {
        System.out.println("not string");
        return;
    }
    System.out.println(s.toUpperCase());
}

process("hello");
process(42);
```
- A. `HELLO` / `not string`
- B. `not string` / `HELLO`
- C. Compile error — `s` not in scope after negated check
- D. NullPointerException

**Answer: A**
After `!(obj instanceof String s)` check with early return, `s` IS in scope for the rest of the method. `process("hello")` → `HELLO`. `process(42)` → `not string`.

---

**Q8 — MCQ**
```java
sealed interface Shape permits Circle, Square {}
record Circle(double r) implements Shape {}
record Square(double side) implements Shape {}

Shape s = new Circle(3.0);
String desc = switch (s) {
    case Circle c  -> "circle r=" + c.r();
    case Square sq -> "square side=" + sq.side();
};
System.out.println(desc);
```
- A. `circle r=3.0`
- B. Compile error — switch expression not exhaustive
- C. Compile error — pattern matching for switch is preview only
- D. NullPointerException

**Answer: A** (assuming preview features enabled) or **C** (if preview not enabled)
For OCP exam context with preview enabled: `Shape` is sealed, all permitted types covered → exhaustive → no `default` needed. `s` is `Circle` → `"circle r=3.0"`.

---

**Q9 — Select All That Apply**
Which are true about pattern matching for `switch` in Java 17?

- A. It is a preview feature requiring `--enable-preview`
- B. Case order matters for type patterns — more specific must come before more general
- C. A `null` case can be handled explicitly without throwing NPE
- D. Guarded patterns use the `when` keyword
- E. Pattern variables in switch cases are effectively final
- F. All constant-based switch behavior is replaced by pattern matching

**Answer: A, B, C, D, E**
F is false — constant switch is still supported and unchanged. A through E are all correct.

---

**Q10 — MCQ**
What is the output?
```java
Object[] items = {1, "two", 3.0, null};
for (Object item : items) {
    String r = switch (item) {
        case null      -> "null";
        case Integer i -> "int:" + i;
        case String s  -> "str:" + s;
        default        -> "other";
    };
    System.out.print(r + " ");
}
```
- A. `int:1 str:two other null `
- B. `int:1 str:two other: null `
- C. NullPointerException on null
- D. `int:1 str:two other null `

**Answer: D**
`1` → `"int:1"`, `"two"` → `"str:two"`, `3.0` → `"other"`, `null` → `"null"`. Output: `int:1 str:two other null `. (Note: `null` case handles the null safely — no NPE.)

---

**Q11 — MCQ**
```java
Object obj = "Hello";
String s = "outer";
if (obj instanceof String s) {
    System.out.println(s);
}
System.out.println(s);
```
- A. `Hello` / `Hello`
- B. `Hello` / `outer`
- C. `outer` / `outer`
- D. Compile error — `s` already declared

**Answer: B**
Pattern variable `s` shadows outer `s` inside the `if` block. Inside: pattern variable → `"Hello"`. Outside: outer `s` → `"outer"`.

---

## 4. Trick Analysis

### Trap 1: `null instanceof T` is always `false` — never NPE
`null instanceof String s` → false, pattern variable not bound, else branch executes. Pattern matching is explicitly null-safe. The pattern variable is never null (because null never matches). Candidates expect NPE from the dereference inside the if-body — but execution never reaches there.

---

### Trap 2: Pattern variable scope with `||` — compile error
```java
if (obj instanceof String s || s.isEmpty()) { }  // COMPILE ERROR
```
On the right side of `||`, the left condition may have been `false` — meaning `s` was not bound. The compiler rejects this. Only `&&` guarantees the pattern matched on the right side.

---

### Trap 3: Pattern variable is effectively final — cannot reassign
```java
if (obj instanceof String s) {
    s = "other";  // COMPILE ERROR
}
```
Unlike regular local variables, pattern variables are effectively final. The exam presents reassignment inside the if-body as a trap. Candidates assume it's just a regular variable declaration.

---

### Trap 4: Pattern variable scope after negated check with early return
```java
if (!(obj instanceof String s)) return;
System.out.println(s.toUpperCase());  // s IS in scope here!
```
After the negated check with early return (or throw), the compiler knows execution only continues if the pattern matched. So `s` IS available after the if block. This is a sophisticated scope rule candidates miss.

---

### Trap 5: Case order matters in pattern switch — more specific first
```java
switch (obj) {
    case Object o -> "any";    // ERROR — dominates String below
    case String s -> "string"; // unreachable
}
```
More general patterns dominate more specific ones. The compiler rejects unreachable cases. Always put specific types before general types.

---

### Trap 6: `null` case in pattern switch prevents NPE
Without `case null ->`, passing null to a pattern switch throws NPE. With `case null ->`, it's handled safely. The exam tests whether candidates know `null` handling changed with pattern matching switch.

---

### Trap 7: Pattern variable shadows outer variable — both are valid
```java
String s = "outer";
if (obj instanceof String s) {  // shadows outer s
    // s here is pattern variable
}
// s here is outer variable — NOT a compile error
```
Shadowing is allowed. Both names are valid in their respective scopes. The exam tests the output — which `s` is used where.

---

### Trap 8: Pattern matching for switch is preview in Java 17
Pattern matching for `instanceof` is **standard** in Java 17 (since Java 16).
Pattern matching for `switch` is **preview** in Java 17 — needs `--enable-preview`.
The exam distinguishes these. Questions about `switch` patterns may note "preview features enabled."

---

### Trap 9: Sealed class switch — no default needed if all types covered
For a sealed interface with known subtypes, a switch expression covering all permitted types is exhaustive — no `default` required. Adding `default` is allowed but not required. The exam tests whether candidates know when exhaustiveness is satisfied without `default`.

---

### Trap 10: Guarded patterns — more specific guard must come before less specific
```java
case Integer i when i > 0 -> "positive";
case Integer i            -> "non-positive";  // catches rest of integers
```
Guarded patterns are checked in order. The unguarded `case Integer i` must come AFTER all guarded versions of the same type. If `case Integer i` came first, it would dominate the guarded version — compile error.

---

## 5. Summary Sheet

### Pattern Matching `instanceof` — Scope Rules

| Situation | Pattern Variable Available? |
|---|---|
| Inside `if` body (non-negated) | ✅ Yes |
| After `if` block (non-negated) | ❌ No |
| Right side of `&&` | ✅ Yes |
| Right side of `\|\|` | ❌ No (compile error) |
| Inside `else` block (non-negated) | ❌ No |
| Inside `else` block (negated with `!`) | ✅ Yes |
| After `if` with early return (negated) | ✅ Yes |
| Shadowing outer variable | ✅ Allowed |

---

### Pattern Variable Properties

| Property | Detail |
|---|---|
| Type | Declared type in pattern |
| Initialization | Automatic when pattern matches |
| Mutability | Effectively final — cannot reassign |
| Null safety | Never null — `null` never matches a pattern |
| Primitive types | ❌ Not supported |
| Scope | Flow-sensitive — where match is guaranteed |

---

### Pattern Matching Switch — Java 17 Preview

| Feature | Detail |
|---|---|
| Standard vs Preview | **Preview** in Java 17 |
| Case ordering | More specific before more general |
| Dominance | Compile error if case unreachable |
| Null handling | `case null ->` prevents NPE |
| Guard syntax | `case T t when condition ->` |
| Exhaustiveness | Required for switch expression |
| Sealed classes | Can omit `default` if all types covered |
| Pattern var scope | Available in case body; effectively final |

---

### Comparison: `instanceof` vs Pattern `instanceof`

| Aspect | Traditional | Pattern Matching |
|---|---|---|
| Code | `if (x instanceof T) { T t = (T)x; }` | `if (x instanceof T t) { }` |
| Null safety | `if (x != null && x instanceof T)` | `if (x instanceof T t)` (null-safe) |
| Cast needed | ✅ Explicit | ❌ Automatic |
| Variable binding | Separate declaration | Combined with test |
| Reassignable | ✅ | ❌ Effectively final |

---

### Key Rules to Memorize

| Rule | Detail |
|---|---|
| `null instanceof T t` | Always `false`, no NPE, `t` not bound |
| Pattern var reassignment | COMPILE ERROR |
| `\|\|` with pattern var | COMPILE ERROR on right side |
| `&&` with pattern var | OK — var available on right side |
| Negated pattern + early return | Var available after the if block |
| Pattern switch case order | Specific before general — or compile error |
| Sealed class switch | No `default` needed if all subtypes covered |
| `case null` in pattern switch | Explicit null handling — no NPE |
| `when` keyword | Guard condition in pattern switch |
| `instanceof` pattern — standard? | ✅ Standard since Java 16 |
| `switch` pattern — standard? | ❌ Preview in Java 17 |

---

### Pattern Matching `switch` Case Order Rules

```
Most specific types / guarded patterns → first
Less specific types → later  
Catch-all pattern (Object o) or default → last
null case → anywhere (usually first)
```

```java
switch (obj) {
    case null                      -> "null";      // null first (optional)
    case Integer i when i > 100    -> "big int";   // guarded — before unguarded
    case Integer i                 -> "int";       // unguarded Integer
    case String s  when s.isEmpty()-> "empty str"; // guarded — before unguarded
    case String s                  -> "string";    // unguarded String
    case Object o                  -> "other";     // most general — last
}
```

---
