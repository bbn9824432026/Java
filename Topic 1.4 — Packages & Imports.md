# Topic 1.4 — Packages & Imports
## (Wildcards, Static Imports, Naming Conflicts)

---

## 1. Conceptual Explanation

### What is a Package?

A package is a **namespace** that organizes related classes and interfaces. It serves two purposes:

1. **Organization** — groups related types together (like folders for classes)
2. **Access control** — package-private members are accessible only within the same package

Packages map directly to **directory structures** on the filesystem. A class in package `com.example.animals` must be located in the directory `com/example/animals/`.

---

### Declaring a Package

The `package` statement must be the **first statement** in a `.java` file (before any imports, before the class declaration). Only comments and blank lines may appear before it.

```java
// File: com/example/animals/Dog.java

package com.example.animals;   // MUST be first statement

import java.util.List;         // imports come after

public class Dog {
    // ...
}
```

**Rules:**
- Only **one** `package` statement per file
- Must appear **before** any `import` statements
- Package names are conventionally all **lowercase**
- Uses **dot notation** — dots correspond to directory separators
- If no `package` statement, the class belongs to the **default (unnamed) package**

---

### The Default Package

If you don't declare a package, your class is in the **default package** — an unnamed package.

```java
// No package statement
public class Hello {   // lives in the default package
    // ...
}
```

**Limitations of the default package:**
- Classes in the default package **cannot be imported** by classes in named packages
- Classes in named packages **cannot** access default-package classes
- The default package is fine for small programs but not for production code
- This is tested — the exam shows code trying to import from default package and asks if it compiles

```java
package com.example;
import Hello;          // COMPILE ERROR — cannot import from default package
```

---

### Importing Classes

The `import` statement tells the compiler where to find a class by its **fully qualified name**. Without it, you must use the full name every time.

```java
// Without import — must use fully qualified name
java.util.ArrayList<String> list = new java.util.ArrayList<>();

// With import — can use simple name
import java.util.ArrayList;
ArrayList<String> list = new ArrayList<>();
```

**Import rules:**
- Imports appear **after** `package` and **before** the class declaration
- Imports are **compile-time only** — they don't affect the `.class` file or runtime
- Importing a class you don't use is legal (no compile error, though IDEs warn)
- You never need to import classes in `java.lang` — they are **auto-imported**
- You never need to import classes in the **same package**

---

### Auto-Imported: `java.lang`

The entire `java.lang` package is imported automatically by the compiler. This includes:

- `String`
- `System`
- `Math`
- `Object`
- `Integer`, `Long`, `Double`, `Boolean` (and other wrappers)
- `StringBuilder`
- `Thread`
- `Exception`, `RuntimeException`
- `Enum`
- `Record`

> **Exam trap:** Candidates sometimes think they need to import `String` or `System`. You don't — `java.lang` is always available.

---

### Wildcard Imports

A wildcard import uses `*` to import **all classes** in a package:

```java
import java.util.*;    // imports all classes in java.util
```

**Critical rules about wildcards:**

1. The `*` only imports **classes** in that package — it does **not** import subpackages

```java
import java.util.*;         // imports java.util.List, java.util.ArrayList, etc.
                            // does NOT import java.util.concurrent.ConcurrentHashMap
                            // does NOT import java.util.function.Predicate
```

2. You must import subpackages **separately**:

```java
import java.util.*;
import java.util.concurrent.*;   // separate import needed
```

3. Wildcard imports do **not** cause performance issues at runtime — the compiler resolves which classes are actually used. Only used classes are included.

4. Using `import java.*;` does **not** import all of Java — it only imports classes **directly** in the `java` package (which is empty — all real classes are in subpackages like `java.util`, `java.lang`). This is a rare but devious trap.

---

### Naming Conflicts

When two imported classes have the same simple name, you have a **naming conflict**. The compiler cannot determine which one you mean.

```java
import java.util.Date;
import java.sql.Date;       // COMPILE ERROR — ambiguous

public class Demo {
    Date d;                 // which Date?
}
```

**Resolution strategies:**

**Strategy 1:** Import one explicitly, use fully qualified name for the other:

```java
import java.util.Date;      // import one

public class Demo {
    Date d1;                          // java.util.Date
    java.sql.Date d2 = new java.sql.Date(0L);  // fully qualified for the other
}
```

**Strategy 2:** Don't import either — use fully qualified names for both:

```java
public class Demo {
    java.util.Date d1;
    java.sql.Date d2;
}
```

**The rule:** An **explicit import always wins over a wildcard import** when there's a conflict:

```java
import java.util.Date;      // explicit import
import java.sql.*;          // wildcard

public class Demo {
    Date d;                 // resolves to java.util.Date — explicit wins
}
```

If **both** imports are explicit (not wildcard), it's a compile error:

```java
import java.util.Date;     // explicit
import java.sql.Date;      // explicit — COMPILE ERROR: ambiguous
```

---

### Import Resolution Priority

From highest to lowest priority:

1. **Fully qualified name** in code (always wins — bypasses imports entirely)
2. **Explicit single-class import** (e.g., `import java.util.Date`)
3. **Wildcard import** (e.g., `import java.util.*`)
4. **Same package** (classes in the same package don't need import)

---

### Static Imports

Static imports allow you to use **static members** (fields and methods) of a class without qualifying them with the class name.

```java
// Without static import
System.out.println(Math.PI);
Math.sqrt(16);

// With static import
import static java.lang.Math.PI;
import static java.lang.Math.sqrt;

System.out.println(PI);
sqrt(16);
```

**Syntax:** `import static` (not `static import`)

```java
import static java.util.Collections.sort;     // specific static method
import static java.lang.Math.*;               // all static members of Math
```

**Static import rules:**

- Works for `static` fields and `static` methods only
- Syntax is `import static packageName.ClassName.memberName`
- Wildcard `import static java.lang.Math.*` imports all static members
- Can cause naming conflicts just like regular imports
- If a **local variable or instance member** has the same name as a statically imported member, the local/instance member **wins**

```java
import static java.lang.Math.PI;

public class Demo {
    double PI = 3.0;                // instance field shadows static import

    void method() {
        System.out.println(PI);     // prints 3.0 — local instance field wins
        System.out.println(Math.PI); // prints 3.141592... — explicit wins
    }
}
```

---

### Static Import Naming Conflicts

```java
import static java.lang.Integer.MAX_VALUE;
import static java.lang.Long.MAX_VALUE;    // COMPILE ERROR — ambiguous
```

Resolution: use the fully qualified approach or import only one.

---

### Compiling and Running with Packages

When classes use packages, the directory structure must match. Compilation and execution also change:

```bash
# Compile from project root
javac com/example/animals/Dog.java

# Run — use fully qualified class name, from project root
java com.example.animals.Dog
```

With classpath:
```bash
javac -cp . com/example/animals/Dog.java
java -cp . com.example.animals.Dog
```

---

### Package Naming Conventions

| Convention | Example |
|---|---|
| All lowercase | `com.example.animals` |
| Reverse domain name | `com.oracle.certification` |
| Avoid Java keywords | `com.example.class` ← illegal |
| Hierarchy with dots | `com.company.project.module` |

---

## 2. Code Examples

### Example 1 — Package and Import Order

```java
package com.example;           // 1. Package statement — FIRST

import java.util.ArrayList;    // 2. Imports — AFTER package
import java.util.List;

public class MyClass {         // 3. Class — AFTER imports
    List<String> items = new ArrayList<>();
}
```

Reversing order causes compile errors:

```java
import java.util.List;
package com.example;           // COMPILE ERROR — package must be first
public class MyClass { }
```

---

### Example 2 — Wildcard Does Not Include Subpackages

```java
import java.util.*;

public class Demo {
    java.util.ArrayList<String> list = new java.util.ArrayList<>();  // OK via wildcard
    java.util.concurrent.CopyOnWriteArrayList<String> cow;           // Need separate import

    // This would fail without explicit import:
    // CopyOnWriteArrayList<String> cow2;  // COMPILE ERROR
}
```

Fix:
```java
import java.util.*;
import java.util.concurrent.*;
```

---

### Example 3 — Explicit Import Wins Over Wildcard

```java
import java.util.Date;     // explicit
import java.sql.*;         // wildcard — includes java.sql.Date

public class Demo {
    Date d = new Date();   // resolves to java.util.Date — explicit wins
}
```

---

### Example 4 — Both Explicit: Compile Error

```java
import java.util.Date;
import java.sql.Date;      // COMPILE ERROR

public class Demo {
    Date d;                // compiler doesn't know which Date
}
```

Fix — use one explicit import and one fully qualified:

```java
import java.util.Date;

public class Demo {
    Date d1 = new Date();
    java.sql.Date d2 = new java.sql.Date(System.currentTimeMillis());
}
```

---

### Example 5 — Static Import

```java
import static java.lang.Math.PI;
import static java.lang.Math.pow;
import static java.lang.Math.sqrt;

public class Circle {
    double circumference(double r) {
        return 2 * PI * r;         // no Math. prefix needed
    }

    double hypotenuse(double a, double b) {
        return sqrt(pow(a, 2) + pow(b, 2));
    }
}
```

---

### Example 6 — Static Import with Wildcard

```java
import static java.lang.Math.*;   // imports ALL static members of Math

public class Demo {
    public static void main(String[] args) {
        System.out.println(PI);          // 3.141592...
        System.out.println(sqrt(144));   // 12.0
        System.out.println(abs(-5));     // 5
        System.out.println(max(10, 20)); // 20
    }
}
```

---

### Example 7 — Cannot Import from Default Package

```java
// File: Helper.java (no package statement — default package)
public class Helper {
    public static void greet() {
        System.out.println("Hello");
    }
}
```

```java
// File: com/example/App.java
package com.example;
import Helper;             // COMPILE ERROR — cannot import from default package

public class App {
    public static void main(String[] args) {
        Helper.greet();    // COMPILE ERROR — cannot access default package class
    }
}
```

---

### Example 8 — `java.lang` is Auto-Imported

```java
// No import needed for String, System, Math, Object, StringBuilder
public class Demo {
    public static void main(String[] args) {
        String s = "hello";               // java.lang.String
        StringBuilder sb = new StringBuilder();  // java.lang.StringBuilder
        System.out.println(Math.PI);      // java.lang.System, java.lang.Math
        Object o = new Object();          // java.lang.Object
    }
}
```

---

### Example 9 — Redundant Import (Legal)

```java
import java.lang.String;    // redundant — java.lang is auto-imported
import java.util.ArrayList; // this one is actually needed
import java.util.List;      // this one is also needed

public class Demo {
    List<String> list = new ArrayList<>();  // String doesn't need import
}
```

Redundant imports are **not** a compile error. They're just unnecessary.

---

### Example 10 — Static Import Conflict with Local Member

```java
import static java.lang.Math.random;

public class Demo {
    public double random() {        // instance method with same name
        return 42.0;
    }

    public void test() {
        System.out.println(random());   // calls THIS class's random() method
                                        // NOT Math.random()
        System.out.println(Math.random()); // explicit to get Math.random()
    }
}
```

Local member always shadows static import.

---

## 3. Exam-Style Questions

---

**Q1 — MCQ**
Which statement about `import java.util.*;` is correct?

- A. It imports all classes in `java.util` and all its subpackages
- B. It imports all classes in `java.util` only
- C. It causes a performance penalty at runtime
- D. It imports all static members of classes in `java.util`

**Answer: B**
Wildcard imports only apply to the specific package, not subpackages. No runtime penalty — compiler resolves at compile time. Static members require `import static`.

---

**Q2 — Select All That Apply**
Which of the following imports are **automatically** available without any import statement? (Choose all that apply)

- A. `java.util.List`
- B. `java.lang.String`
- C. `java.lang.Math`
- D. `java.io.File`
- E. `java.lang.Integer`
- F. `java.util.ArrayList`

**Answer: B, C, E**
Only `java.lang.*` is auto-imported. `java.util` and `java.io` require explicit imports.

---

**Q3 — MCQ**
```java
package com.test;
import java.util.Date;
import java.sql.Date;

public class Demo {
    public static void main(String[] args) {
        Date d = new Date();
    }
}
```
What is the result?

- A. Compiles and runs — uses `java.util.Date`
- B. Compiles and runs — uses `java.sql.Date`
- C. Compile error — ambiguous import
- D. Runtime error — ambiguous class

**Answer: C**
Two explicit imports of classes with the same simple name cause a compile error.

---

**Q4 — MCQ**
```java
import java.util.Date;
import java.sql.*;

public class Demo {
    Date d = new Date();
}
```
What type is `d`?

- A. `java.util.Date`
- B. `java.sql.Date`
- C. Compile error — ambiguous
- D. Runtime error

**Answer: A**
Explicit import (`java.util.Date`) takes priority over wildcard import (`java.sql.*`). No ambiguity — explicit wins.

---

**Q5 — MCQ**
Which is the correct syntax for a static import?

- A. `static import java.lang.Math.PI;`
- B. `import static java.lang.Math.PI;`
- C. `import java.lang.Math.PI;`
- D. `import static PI from java.lang.Math;`

**Answer: B**
The correct syntax is `import static` (keyword `import` first, then `static`). Option A reverses the keywords.

---

**Q6 — Code Output**
```java
import static java.lang.Math.max;
import static java.lang.Math.min;

public class Demo {
    public static void main(String[] args) {
        System.out.println(max(3, 7));
        System.out.println(min(3, 7));
    }
}
```

- A. Compile error
- B. `7` then `3`
- C. `3` then `7`
- D. `7` then `7`

**Answer: B**
`max(3,7)` returns 7, `min(3,7)` returns 3.

---

**Q7 — Select All That Apply**
Which of the following are true about the `package` statement? (Choose all that apply)

- A. A file can have multiple `package` statements
- B. The `package` statement must appear before any `import` statements
- C. The `package` statement is optional
- D. Classes without a package statement belong to the default package
- E. Classes in the default package can be imported by named packages

**Answer: B, C, D**
A is false — only one package statement allowed. B is true. C is true — package is optional. D is true. E is false — default package classes cannot be imported.

---

**Q8 — MCQ**
```java
package com.example;

public class App {
    public static void main(String[] args) {
        System.out.println("Running");
    }
}
```
Which command correctly runs this program from the project root directory?

- A. `java App`
- B. `java com/example/App`
- C. `java com.example.App`
- D. `java com.example/App`

**Answer: C**
Fully qualified class name with dots. No slashes, no `.class` extension.

---

**Q9 — MCQ**
```java
import java.*;
import java.util.ArrayList;

public class Demo {
    ArrayList<String> list = new ArrayList<>();
}
```
What is the result?

- A. Compile error — `import java.*` is illegal
- B. Compile error — cannot use two imports for same class
- C. Compiles fine
- D. Runtime error

**Answer: C**
`import java.*` is legal syntax but imports nothing useful (the `java` package itself has no classes — they're all in subpackages). `import java.util.ArrayList` provides the needed class. No conflict. Compiles fine.

---

**Q10 — Tricky Scenario**
```java
package com.example;
import java.util.*;
import java.util.ArrayList;

public class Demo {
    ArrayList<String> list = new ArrayList<>();
}
```
What is the result?

- A. Compile error — duplicate import
- B. Compile error — cannot have both wildcard and explicit import for same class
- C. Compiles fine
- D. Runtime warning

**Answer: C**
Having both a wildcard and an explicit import for the same class is **legal** and not an error. Redundant imports are permitted.

---

## 4. Trick Analysis

### Trap 1: Wildcard imports subpackages
`import java.util.*` does **not** import `java.util.concurrent.*` or `java.util.function.*`. Candidates assume the `*` is recursive. It is not — it only covers classes directly in that one package.

---

### Trap 2: `import java.*` imports everything
This is false. `java.*` imports only classes in the `java` package itself — which is essentially nothing, since all useful classes are in subpackages like `java.util`, `java.lang`. This is a rare but memorable trick.

---

### Trap 3: Explicit + wildcard = ambiguity
Only **explicit + explicit** causes ambiguity. **Explicit + wildcard** is fine — the explicit import wins. Candidates often think any combination of two `Date` imports causes a compile error, but that's only when both are explicit.

---

### Trap 4: `import static` vs `static import`
The correct syntax is `import static`. Writing `static import` is a compile error. Oracle will present both in options. The keyword order matches the sentence "import [this] static [member]."

---

### Trap 5: `java.lang` needs to be imported
It doesn't. `String`, `System`, `Math`, `Object`, `Integer`, `Long`, `Double`, `Boolean`, `StringBuilder`, `Thread` — none of these need an import. The exam will include code with `import java.lang.String;` and ask if it compiles — it does, but the import is redundant.

---

### Trap 6: Package statement position
Even one line out of order causes a compile error. If an `import` appears before `package`, or a class declaration appears before `import`, it fails. The exam presents scrambled order and asks what happens.

---

### Trap 7: Default package classes are inaccessible from named packages
A class in the default package cannot be imported into or used by a class in a named package. The exam shows this scenario and asks if it compiles — it doesn't.

---

### Trap 8: Unused imports don't cause compile errors
The exam sometimes shows code with imports for classes that aren't used. This is **not** a compile error. Unused imports are a warning in IDEs but the compiler accepts them.

---

## 5. Summary Sheet

### Order of Statements in a `.java` File

```
1. package statement      (optional, at most one, must be first)
2. import statements      (optional, after package, before class)
3. class/interface/enum   (the actual type declaration)
```

---

### Import Types Compared

| Import Type | Syntax | What It Does |
|---|---|---|
| Single-class | `import java.util.List;` | Imports one specific class |
| Wildcard | `import java.util.*;` | Imports all classes in that package |
| Static single | `import static java.lang.Math.PI;` | Imports one static member |
| Static wildcard | `import static java.lang.Math.*;` | Imports all static members of a class |

---

### Naming Conflict Resolution

| Situation | Result |
|---|---|
| Explicit + Explicit (same name) | **Compile error** |
| Explicit + Wildcard (same name) | **Explicit wins** — no error |
| Wildcard + Wildcard (same name) | **Compile error** |
| Fully qualified name in code | **Always wins** — overrides any import |

---

### Key Rules to Memorize

| Rule | Detail |
|---|---|
| `java.lang` | Auto-imported — never needs explicit import |
| Same package | Never needs import |
| Default package | Cannot be imported by named packages |
| Wildcard `*` | Only covers classes directly in that package — not subpackages |
| `import static` | For static fields and methods only — keyword order: `import` then `static` |
| Unused imports | Legal — no compile error |
| Multiple package statements | **Illegal** — compile error |
| Package statement position | Must be before any import or class declaration |
| Redundant imports | Legal (e.g., explicit + wildcard for same class) |

---

### Packages and Directory Structure

| Package | Directory |
|---|---|
| `com.example.animals` | `com/example/animals/` |
| `java.util` | `java/util/` |
| (default) | Project root directory |

---
