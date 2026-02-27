# Topic 2.9 — `switch` Statements
## (Traditional & Enhanced)

---

## 1. Conceptual Explanation

### What Is a `switch` Statement?

A `switch` statement selects one of several code paths based on the value of a single expression. It is an alternative to chains of `if-else if` blocks when comparing a single variable against multiple constant values.

Java has two distinct forms:
1. **Traditional `switch` statement** — the classic form with `case`, `break`, and fall-through
2. **Enhanced `switch`** — introduced as preview in Java 14, standard in Java 16+ — uses `->` arrows, no fall-through, and can be used as an expression

Both forms are tested on OCP Java SE 17.

---

### Traditional `switch` Statement — Full Syntax

```java
switch (expression) {
    case value1:
        // code
        break;
    case value2:
        // code
        break;
    case value3:
    case value4:    // fall-through grouping
        // code
        break;
    default:
        // code
}
```

---

### What Can Be the `switch` Expression?

The `switch` expression (the value being tested) must be one of these types:

| Allowed Type | Notes |
|---|---|
| `byte`, `short`, `int`, `char` | Primitive integer types |
| `Byte`, `Short`, `Integer`, `Character` | Wrapper types (unboxed) |
| `String` | Since Java 7 |
| `enum` | Since Java 5 |
| `var` | If inferred type is one of the above |

**NOT allowed:**
- `long`, `float`, `double`, `boolean`
- `Long`, `Float`, `Double`, `Boolean`
- Any other object type (before Java 21)

```java
int x = 5;
switch (x) { ... }       // OK

long l = 5L;
switch (l) { ... }       // COMPILE ERROR — long not allowed

boolean b = true;
switch (b) { ... }       // COMPILE ERROR — boolean not allowed

String s = "hello";
switch (s) { ... }       // OK — String allowed since Java 7
```

---

### `case` Values — Must Be Compile-Time Constants

Each `case` value must be a **compile-time constant** of the same type (or compatible with) the switch expression:

```java
int x = 5;
final int A = 1;
int B = 2;           // not final — not a compile-time constant

switch (x) {
    case 1:          // OK — literal
        break;
    case A:          // OK — final variable (compile-time constant)
        break;
    // case B:       // COMPILE ERROR — B is not final
    //     break;
    // case 1 + 1:   // OK — compile-time constant expression
    //     break;
    // case x:       // COMPILE ERROR — x is not a constant
    //     break;
}
```

---

### Fall-Through Behavior — The Most Tested Feature

The single most tested aspect of traditional `switch` is **fall-through**. When there is NO `break` statement at the end of a case, execution **falls through** to the next case's code — regardless of whether that next case's label matches.

```java
int x = 1;
switch (x) {
    case 1:
        System.out.println("One");
        // NO break — falls through!
    case 2:
        System.out.println("Two");
        // NO break — falls through!
    case 3:
        System.out.println("Three");
        break;
    case 4:
        System.out.println("Four");
}
// Output:
// One
// Two
// Three
```

Only `"One"` matches the switch expression, but because there is no `break`, execution continues through `case 2` and `case 3` until the `break` in `case 3`.

---

### `break` in `switch`

`break` exits the `switch` block entirely:

```java
int x = 2;
switch (x) {
    case 1:
        System.out.println("One");
        break;     // exits switch if x==1
    case 2:
        System.out.println("Two");
        break;     // exits switch if x==2 — only "Two" printed
    case 3:
        System.out.println("Three");
}
// Output: Two
```

---

### `default` Clause

`default` matches when no `case` value matches the switch expression:

```java
int x = 99;
switch (x) {
    case 1:
        System.out.println("One");
        break;
    case 2:
        System.out.println("Two");
        break;
    default:
        System.out.println("Other");  // prints this
}
```

**Key rules about `default`:**
- `default` can appear **anywhere** in the switch — not just at the end
- If `default` is in the middle, fall-through still applies
- `default` is optional
- There can be only **one** `default` per switch
- `default` is reached only when no case matches — but if another case falls through to it, it executes

```java
int x = 1;
switch (x) {
    default:                       // default in the MIDDLE
        System.out.println("default");
        // NO break — falls through to case 1!
    case 1:
        System.out.println("one");
        break;
    case 2:
        System.out.println("two");
}
// x==1 matches case 1 → "one" (default NOT reached because case 1 matched)
```

```java
int x = 99;
switch (x) {
    default:                       // default in the MIDDLE
        System.out.println("default");
        // NO break — falls through to case 1!
    case 1:
        System.out.println("one");
        break;
    case 2:
        System.out.println("two");
}
// x==99 → no match → goes to default
// "default" printed, then falls through to case 1: "one"
// Output: default \n one
```

---

### `return` in `switch`

Inside a method, `return` can exit both the switch AND the method:

```java
String getDayType(int day) {
    switch (day) {
        case 1: case 7:
            return "Weekend";
        case 2: case 3: case 4: case 5: case 6:
            return "Weekday";
        default:
            return "Invalid";
    }
}
```

---

### `switch` with `String`

Since Java 7, `String` can be used in switch. The comparison is **case-sensitive** using `.equals()`:

```java
String command = "quit";
switch (command) {
    case "start":
        System.out.println("Starting");
        break;
    case "quit":
        System.out.println("Quitting");  // prints
        break;
    case "QUIT":
        System.out.println("Also quit");  // NOT matched — case-sensitive
        break;
    default:
        System.out.println("Unknown");
}
```

> **Exam trap:** If `command` is `null`, a `NullPointerException` is thrown when the switch evaluates the expression. Never pass null to a String switch.

```java
String s = null;
switch (s) { ... }  // NullPointerException at runtime
```

---

### `switch` with `enum`

```java
enum Day { MON, TUE, WED, THU, FRI, SAT, SUN }

Day day = Day.MON;
switch (day) {
    case MON:
    case TUE:
    case WED:
    case THU:
    case FRI:
        System.out.println("Weekday");
        break;
    case SAT:
    case SUN:
        System.out.println("Weekend");
        break;
}
```

> **Exam note:** In a switch with an `enum` type, the `case` labels must use **unqualified enum constant names** (`MON`, not `Day.MON`):

```java
case Day.MON:  // COMPILE ERROR — must be just MON
case MON:      // correct
```

---

### Enhanced `switch` Statement (Java 14+, Standard Java 16+)

The enhanced `switch` uses **arrow (`->`) case labels** and eliminates fall-through:

```java
int x = 2;
switch (x) {
    case 1 -> System.out.println("One");
    case 2 -> System.out.println("Two");   // only this executes
    case 3 -> System.out.println("Three");
    default -> System.out.println("Other");
}
```

**Key differences from traditional `switch`:**
1. **No fall-through** — each arrow case is independent
2. **No `break` needed** — arrows prevent fall-through automatically
3. **Multiple labels per case** — `case 1, 2, 3 ->` on one line
4. **Block syntax** — can use `{ }` blocks after `->` for multiple statements
5. **Can be used as an expression** — can return a value

---

### Enhanced `switch` — Multiple Labels

```java
int day = 3;
switch (day) {
    case 1, 7 -> System.out.println("Weekend");
    case 2, 3, 4, 5, 6 -> System.out.println("Weekday");
    default -> System.out.println("Invalid");
}
// Output: Weekday
```

---

### Enhanced `switch` — Block Syntax

When a case needs multiple statements, use a block:

```java
int x = 2;
switch (x) {
    case 1 -> System.out.println("One");
    case 2 -> {
        System.out.println("Two - part 1");
        System.out.println("Two - part 2");
    }
    default -> System.out.println("Other");
}
```

---

### `switch` Expression (Java 14+)

The enhanced switch can be used as an **expression** — it produces a value:

```java
int day = 3;
String type = switch (day) {
    case 1, 7 -> "Weekend";
    case 2, 3, 4, 5, 6 -> "Weekday";
    default -> "Invalid";
};
System.out.println(type);  // Weekday
```

**Rules for `switch` expression:**
1. Must be **exhaustive** — every possible value must be covered (either with all cases or with `default`)
2. Each branch must produce a value of the same (or compatible) type
3. Cannot use `break` without a value — use `yield` or arrow labels
4. The semicolon after the closing `}` is required

---

### `yield` in `switch` Expression

When a `switch` **expression** uses traditional colon-style cases (not arrows), you use `yield` to return a value from a block:

```java
int x = 2;
String result = switch (x) {
    case 1:
        yield "One";       // yield returns value from switch expression
    case 2: {
        String s = "Tw";
        yield s + "o";     // yield from block
    }
    default:
        yield "Other";
};
System.out.println(result);  // Two
```

> **`yield` vs `break`:**
> - `yield value` — exits the switch expression AND provides the value
> - `break` — exits the switch statement (not valid for returning a value)
> - `yield` is ONLY valid inside a `switch` expression — not in a switch statement

```java
// In switch STATEMENT — no yield needed/allowed
switch (x) {
    case 1:
        System.out.println("one");
        break;     // OK
        // yield "one";  // COMPILE ERROR in switch statement
}

// In switch EXPRESSION — yield OR arrow
String r = switch (x) {
    case 1 -> "one";          // arrow — no yield needed
    case 2: yield "two";      // colon — yield required
    default: yield "other";
};
```

---

### Mixed Arrow and Colon in Same Switch

You CANNOT mix `->` (arrow) and `:` (colon) style cases in the same switch:

```java
// COMPILE ERROR — cannot mix styles
switch (x) {
    case 1 -> System.out.println("one");
    case 2:                               // COMPILE ERROR
        System.out.println("two");
        break;
}
```

---

### Exhaustiveness in `switch` Expression

A `switch` expression must cover ALL possible values — otherwise it's a compile error:

```java
// int switch expression — MUST have default (infinite possible int values)
int x = 5;
String r = switch (x) {
    case 1 -> "one";
    case 2 -> "two";
    // COMPILE ERROR — not exhaustive, no default
};

// Fix — add default:
String r2 = switch (x) {
    case 1 -> "one";
    case 2 -> "two";
    default -> "other";   // now exhaustive
};

// enum switch expression — can cover all enum values without default:
enum Color { RED, GREEN, BLUE }
Color c = Color.RED;
String name = switch (c) {
    case RED -> "Red";
    case GREEN -> "Green";
    case BLUE -> "Blue";
    // No default needed — all enum values covered
};
```

---

### `switch` Statement vs `switch` Expression — Comparison

| Feature | `switch` Statement | `switch` Expression |
|---|---|---|
| Returns value | ❌ | ✅ |
| `break` | Exits switch | ❌ (use `yield`) |
| `yield` | ❌ | ✅ |
| Must be exhaustive | ❌ | ✅ |
| Arrow `->` style | ✅ | ✅ |
| Colon `:` style | ✅ | ✅ (with `yield`) |
| Fall-through (colon style) | ✅ | ✅ (but rare/dangerous) |
| Fall-through (arrow style) | ❌ | ❌ |
| Semicolon at end | ❌ | ✅ (required) |

---

### Scope in Traditional `switch`

The entire `switch` body is **one scope**. Variables declared in one case are in scope for all subsequent cases — even if those cases don't initialize them:

```java
int x = 2;
switch (x) {
    case 1:
        int y = 10;      // y declared here — in scope for ALL cases below
        break;
    case 2:
        // y is in scope but might not be initialized!
        // System.out.println(y);  // COMPILE ERROR — y might not be initialized
        y = 20;          // OK — assign first
        System.out.println(y);  // 20
        break;
}
```

---

### Null Handling in `switch` (Java 21+ Pattern Matching — Awareness)

In Java SE 17 (the exam version), `null` in switch throws `NullPointerException`. Be aware that later Java versions add null support, but for OCP 17, always assume null → NPE.

---

## 2. Code Examples

### Example 1 — Fall-Through Tracing

```java
public class FallThroughDemo {
    public static void main(String[] args) {
        int x = 2;

        switch (x) {
            case 1:
                System.out.println("case 1");
            case 2:
                System.out.println("case 2");   // matches here
            case 3:
                System.out.println("case 3");   // falls through
            case 4:
                System.out.println("case 4");   // falls through
                break;
            case 5:
                System.out.println("case 5");
        }
        // Output:
        // case 2
        // case 3
        // case 4
    }
}
```

---

### Example 2 — `default` in the Middle

```java
public class DefaultMiddle {
    public static void main(String[] args) {
        // Test with x = 5 (no match)
        int x = 5;
        switch (x) {
            case 1:
                System.out.println("one");
                break;
            default:
                System.out.println("default");
                // NO break — falls through to case 2!
            case 2:
                System.out.println("two");
                break;
            case 3:
                System.out.println("three");
        }
        // x=5 → no match → default
        // default: "default" printed, no break → falls through → "two"
        // Output: default \n two

        System.out.println("---");

        // Test with x = 1 (matches case 1)
        x = 1;
        switch (x) {
            case 1:
                System.out.println("one");
                break;
            default:
                System.out.println("default");
            case 2:
                System.out.println("two");
                break;
        }
        // x=1 → matches case 1 → "one" → break
        // Output: one
    }
}
```

---

### Example 3 — Enhanced `switch` Statement

```java
public class EnhancedSwitch {
    public static void main(String[] args) {
        for (int day = 1; day <= 7; day++) {
            switch (day) {
                case 1, 7 -> System.out.println(day + ": Weekend");
                case 2, 3, 4, 5, 6 -> System.out.println(day + ": Weekday");
                default -> System.out.println(day + ": Invalid");
            }
        }
    }
}
```

---

### Example 4 — `switch` Expression with Arrow

```java
public class SwitchExpression {
    public static void main(String[] args) {
        for (int i = 1; i <= 5; i++) {
            String label = switch (i) {
                case 1 -> "one";
                case 2 -> "two";
                case 3 -> "three";
                default -> "many";
            };
            System.out.println(i + " = " + label);
        }
    }
}
```

---

### Example 5 — `switch` Expression with `yield`

```java
public class YieldDemo {
    public static void main(String[] args) {
        int x = 3;

        String result = switch (x) {
            case 1:
                yield "one";
            case 2:
                yield "two";
            case 3: {
                String prefix = "thr";
                yield prefix + "ee";   // yield from block
            }
            default:
                yield "unknown";
        };

        System.out.println(result);  // three
    }
}
```

---

### Example 6 — `switch` Expression Must Be Exhaustive

```java
public class Exhaustive {
    enum Season { SPRING, SUMMER, FALL, WINTER }

    public static void main(String[] args) {
        Season s = Season.SUMMER;

        // Exhaustive — all enum constants covered, no default needed
        String desc = switch (s) {
            case SPRING -> "Warm";
            case SUMMER -> "Hot";
            case FALL   -> "Cool";
            case WINTER -> "Cold";
        };
        System.out.println(desc);  // Hot

        // Non-enum — MUST have default
        int x = 5;
        String r = switch (x) {
            case 1 -> "one";
            case 2 -> "two";
            default -> "other";   // required — int has infinite values
        };
        System.out.println(r);  // other
    }
}
```

---

### Example 7 — `yield` in Expression with Fall-Through

```java
public class YieldFallThrough {
    public static void main(String[] args) {
        int x = 2;

        // Colon-style switch expression CAN have fall-through (but yield required)
        String result = switch (x) {
            case 1:
            case 2:               // fall-through to case 2's yield
                yield "one or two";
            case 3:
                yield "three";
            default:
                yield "other";
        };
        System.out.println(result);  // one or two
    }
}
```

---

### Example 8 — String Switch with null Trap

```java
public class StringSwitch {
    static String getCategory(String fruit) {
        if (fruit == null) return "null input";  // guard against null!
        return switch (fruit) {
            case "apple", "pear" -> "pome";
            case "peach", "plum" -> "drupe";
            case "strawberry"    -> "aggregate";
            default              -> "unknown";
        };
    }

    public static void main(String[] args) {
        System.out.println(getCategory("apple"));       // pome
        System.out.println(getCategory("peach"));       // drupe
        System.out.println(getCategory("banana"));      // unknown
        System.out.println(getCategory(null));          // null input (guarded)
        // Without guard: switch(null) → NullPointerException
    }
}
```

---

### Example 9 — Switch Scope Trap

```java
public class SwitchScope {
    public static void main(String[] args) {
        int x = 2;

        switch (x) {
            case 1:
                int result;               // declared — in scope for ALL cases
            case 2:
                result = 100;             // legal — assigning in different case
                System.out.println(result); // 100
                break;
            case 3:
                // System.out.println(result); // COMPILE ERROR — might not be assigned
                result = 200;
                System.out.println(result);
                break;
        }
    }
}
```

---

### Example 10 — Traditional vs Enhanced Equivalence

```java
public class TraditionalVsEnhanced {
    public static void main(String[] args) {
        int x = 2;

        // Traditional — needs break, has fall-through risk
        switch (x) {
            case 1:
                System.out.println("traditional: one");
                break;
            case 2:
                System.out.println("traditional: two");
                break;
            default:
                System.out.println("traditional: other");
        }

        // Enhanced — no break needed, no fall-through
        switch (x) {
            case 1 -> System.out.println("enhanced: one");
            case 2 -> System.out.println("enhanced: two");
            default -> System.out.println("enhanced: other");
        }

        // Enhanced as expression
        String s = switch (x) {
            case 1 -> "expr: one";
            case 2 -> "expr: two";
            default -> "expr: other";
        };
        System.out.println(s);
    }
}
```

---

## 3. Exam-Style Questions

---

**Q1 — MCQ**
What is the output?
```java
int x = 2;
switch (x) {
    case 1: System.out.println("one");
    case 2: System.out.println("two");
    case 3: System.out.println("three");
    default: System.out.println("default");
}
```
- A. `two`
- B. `two` / `three`
- C. `two` / `three` / `default`
- D. `one` / `two` / `three` / `default`

**Answer: C**
`x=2` matches `case 2`. No `break` → falls through to `case 3`, then `default`. Output: `two`, `three`, `default`.

---

**Q2 — MCQ**
What is the output?
```java
int x = 5;
switch (x) {
    case 1:
        System.out.println("one");
        break;
    default:
        System.out.println("default");
    case 2:
        System.out.println("two");
        break;
}
```
- A. `default`
- B. `default` / `two`
- C. `two`
- D. Nothing

**Answer: B**
`x=5` matches no case → goes to `default`. `default` prints, no `break` → falls through to `case 2` → `two` prints → `break`.

---

**Q3 — MCQ**
Which of the following can be used as a `switch` expression in Java 17?

- A. `long`
- B. `double`
- C. `String`
- D. `boolean`

**Answer: C**
`String` is valid since Java 7. `long`, `double`, and `boolean` are NOT allowed as switch expression types.

---

**Q4 — MCQ**
What is the output?
```java
int x = 1;
switch (x) {
    default:
        System.out.println("default");
    case 1:
        System.out.println("one");
        break;
    case 2:
        System.out.println("two");
}
```
- A. `default` / `one`
- B. `one`
- C. `default`
- D. `one` / `default`

**Answer: B**
`x=1` matches `case 1` directly — `default` is not visited. `"one"` prints, then `break`. Output: `one`.

---

**Q5 — Select All That Apply**
Which statements are true about enhanced `switch` (arrow syntax)?

- A. Fall-through does not occur automatically
- B. `break` is required to exit each case
- C. Multiple case labels can be combined with commas
- D. It can be used as an expression
- E. `yield` is used to return a value from a block
- F. You can mix arrow and colon syntax in the same switch

**Answer: A, C, D, E**
B is false — no `break` needed with arrows. F is false — mixing arrow and colon is a compile error. A, C, D, E are all correct.

---

**Q6 — MCQ**
What is the output?
```java
int x = 3;
String r = switch (x) {
    case 1 -> "one";
    case 2 -> "two";
    case 3 -> {
        String s = "thr";
        yield s + "ee";
    }
    default -> "other";
};
System.out.println(r);
```
- A. Compile error — cannot use `yield` in `switch` expression
- B. `three`
- C. `thr`
- D. `other`

**Answer: B**
`x=3` matches `case 3`. Block executes, `s="thr"`, `yield "three"`. Result: `"three"`.

---

**Q7 — MCQ**
```java
String s = null;
switch (s) {
    case "hello": System.out.println("hello"); break;
    default: System.out.println("other");
}
```
What is the result?
- A. `other`
- B. `hello`
- C. `NullPointerException`
- D. Compile error

**Answer: C**
`switch` on a `null` `String` reference throws `NullPointerException` at runtime.

---

**Q8 — MCQ**
```java
enum Color { RED, GREEN, BLUE }
Color c = Color.GREEN;
switch (c) {
    case RED:   System.out.println("red"); break;
    case GREEN: System.out.println("green"); break;
    case BLUE:  System.out.println("blue"); break;
}
```
Which modification causes a compile error?

- A. Removing all `break` statements
- B. Changing `case GREEN:` to `case Color.GREEN:`
- C. Adding `default: System.out.println("other");`
- D. Using `Color c = null;` instead

**Answer: B**
In a switch with an `enum` type, `case` labels must use the **unqualified** enum constant name. `case Color.GREEN:` is a compile error.

---

**Q9 — MCQ**
What is the output?
```java
int x = 3;
switch (x) {
    case 1: case 2: case 3:
        System.out.println("1 to 3");
        break;
    case 4: case 5:
        System.out.println("4 to 5");
        break;
    default:
        System.out.println("other");
}
```
- A. `1 to 3` / `4 to 5` / `other`
- B. `1 to 3`
- C. `4 to 5`
- D. Compile error

**Answer: B**
`x=3` matches `case 3` which falls through to the print statement. Prints `"1 to 3"`, then `break`.

---

**Q10 — MCQ**
```java
int x = 2;
int result = switch (x) {
    case 1 -> 10;
    case 2 -> 20;
    case 3 -> 30;
};
System.out.println(result);
```
- A. Compile error — switch expression must have default
- B. `20`
- C. `0`
- D. `30`

**Answer: A**
A `switch` expression on `int` **must** be exhaustive. Without a `default`, not all int values are covered — compile error.

---

**Q11 — Select All That Apply**
Which are valid `case` labels for `switch (x)` where `x` is of type `int`?
```java
final int A = 5;
int B = 6;
final int C;
C = 7;
```
- A. `case 5:`
- B. `case A:`
- C. `case B:`
- D. `case C:`
- E. `case 2 + 3:`
- F. `case (int) 5.0:`

**Answer: A, B, E, F**
C: `B` is not final — ERROR. D: `C` is a blank final assigned later — not a compile-time constant — ERROR. A: literal ✅. B: `A` is `final int` with literal initializer ✅. E: `2+3` = compile-time constant expression ✅. F: `(int)5.0` = 5, a compile-time constant ✅.

---

**Q12 — MCQ**
What is the output?
```java
int x = 2;
String r = switch (x) {
    case 1:
    case 2:
        yield "one or two";
    case 3:
        yield "three";
    default:
        yield "other";
};
System.out.println(r);
```
- A. Compile error — cannot mix colon and yield
- B. `one or two`
- C. `three`
- D. `other`

**Answer: B**
`x=2` → `case 2` → fall-through (colon style in switch expression) → `case 2`'s `yield "one or two"`. Wait — `case 1:` falls through to `case 2:` which yields. Since `x=2` directly hits `case 2` which leads to `yield "one or two"`. Result: `"one or two"`.

---

## 4. Trick Analysis

### Trap 1: Fall-through is silent and unexpected
The single most common switch trap. Without `break`, execution continues into the next case unconditionally. The exam writes code without `break` and asks what prints. Always check every `case` for a `break`, `return`, `throw`, or `yield` that would exit.

---

### Trap 2: `default` anywhere — not just at the end
`default` can be placed first, last, or anywhere in the middle. When placed in the middle with no `break`, it falls through to the next case. The exam puts `default` before other cases and tests whether candidates trace the flow correctly.

---

### Trap 3: `default` only runs when no case matches — unless fallen into
`default` is reached ONLY when no case matches the expression. BUT if another case above it has no `break`, execution can fall INTO `default` from a matching case. When `x=5` hits no case and `default` is in the middle, it falls through to the next case below `default`.

---

### Trap 4: `case Color.GREEN:` is a compile error for enum switch
Inside a switch on an enum type, case labels use UNQUALIFIED names. `case GREEN:` is correct. `case Color.GREEN:` is a compile error. Candidates who come from pattern matching or fully qualified references make this mistake.

---

### Trap 5: `switch(null)` throws NPE — even with `String`
There is no `case null:` in Java 17. Switching on a null `String`, `enum`, or `Integer` throws `NullPointerException`. Always null-check before switching on reference types.

---

### Trap 6: `long`, `float`, `double`, `boolean` are NOT valid switch types
The exam presents these types as switch expressions and asks if it compiles. `long` is the most common trap — it seems like it should work (it's an integer type) but it doesn't. Only `byte`, `short`, `int`, `char`, their wrappers, `String`, and `enum` are allowed.

---

### Trap 7: Missing `default` in `switch` expression → compile error
A switch STATEMENT without `default` is fine. A switch EXPRESSION without `default` is only fine if all possible values are explicitly covered (e.g., all enum constants). For `int`, `String`, etc., `default` is required in a switch expression.

---

### Trap 8: `yield` only in switch expression, `break` only in switch statement
`yield` cannot appear in a `switch` statement — compile error. `break` without a value cannot appear in a `switch` expression — use `yield`. The exam swaps these to test whether candidates know which belongs where.

---

### Trap 9: Cannot mix `->` and `:` in same switch
```java
switch (x) {
    case 1 -> "one";
    case 2: yield "two";   // COMPILE ERROR — mixed styles
}
```
Pick one style per switch. The exam presents mixed-style switches to test this.

---

### Trap 10: Switch scope — variable declared in one case is in scope for all
Unlike `if-else`, the entire switch body is one block. A variable declared in `case 1:` is technically in scope in `case 2:`, `case 3:`, etc. But it might not be initialized there — leading to "might not have been initialized" compile errors. The exam tests both the scope accessibility and the initialization requirement.

---

### Trap 11: `case 2 + 3:` is valid — compile-time constant expression
Arithmetic expressions of compile-time constants are valid `case` labels. `case 2 + 3:` is the same as `case 5:`. Candidates don't expect expressions here.

---

### Trap 12: Fall-through in `switch` expression with colon style
```java
String r = switch (x) {
    case 1:
    case 2:
        yield "one or two";  // case 1 falls through to case 2's yield
    default:
        yield "other";
};
```
Colon-style switch expressions CAN have fall-through. The `case 1:` falls through to the `yield` in `case 2`. The exam tests whether candidates know this is possible.

---

## 5. Summary Sheet

### Valid `switch` Expression Types

| Type | Valid? |
|---|---|
| `byte`, `short`, `int`, `char` | ✅ |
| `Byte`, `Short`, `Integer`, `Character` | ✅ |
| `String` | ✅ (Java 7+) |
| `enum` | ✅ |
| `var` (if inferred as above) | ✅ |
| `long`, `float`, `double` | ❌ |
| `boolean`, `Boolean` | ❌ |
| `Long`, `Float`, `Double` | ❌ |

---

### Traditional vs Enhanced Switch

| Feature | Traditional (`:`) | Enhanced (`->`) |
|---|---|---|
| Fall-through | ✅ Yes | ❌ No |
| `break` needed | ✅ Yes | ❌ No |
| Multiple labels | `case 1: case 2:` | `case 1, 2 ->` |
| As expression | With `yield` | Directly |
| `yield` keyword | ✅ (in expression) | In `{ }` blocks |
| Mix both styles | ❌ Compile error | ❌ Compile error |

---

### `switch` Statement vs Expression

| Aspect | Statement | Expression |
|---|---|---|
| Returns value | ❌ | ✅ |
| `default` required | ❌ | ✅ (unless exhaustive) |
| `break` to exit | ✅ | ❌ |
| `yield` to return | ❌ | ✅ |
| Semicolon at end | ❌ | ✅ |

---

### Fall-Through Rules

| Scenario | Fall-Through? |
|---|---|
| Colon `:` style, no `break`/`return`/`yield` | ✅ Yes |
| Arrow `->` style | ❌ Never |
| After `break` | ❌ Exits switch |
| After `return` | ❌ Exits method |
| After `throw` | ❌ Throws exception |
| `default` in middle, no `break` | ✅ Falls through to next case |

---

### Key Rules to Memorize

| Rule | Detail |
|---|---|
| `null` in switch | NPE — no `case null:` in Java 17 |
| `case Color.GREEN:` | COMPILE ERROR — use `case GREEN:` |
| `long` in switch | COMPILE ERROR |
| `case` value must be | Compile-time constant |
| `default` position | Anywhere — not just end |
| Switch scope | One block — variable scope spans all cases |
| `yield` vs `break` | `yield` for expression value, `break` for statement exit |
| Missing `default` in expression | Compile error for non-exhaustive types |
| Mixed `->` and `:` | COMPILE ERROR |
| Enum switch — no `default` | OK if all constants covered |

---
