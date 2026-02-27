# Topic 1.10 — Object Lifecycle & Garbage Collection Eligibility

---

## 1. Conceptual Explanation

### The Object Lifecycle

Every Java object goes through a well-defined lifecycle:

```
1. Declaration    → reference variable declared (no object yet)
2. Creation       → new allocates memory on heap, constructor runs
3. Usage          → object is referenced and operated on
4. Dereferencing  → all references to the object are gone
5. GC Eligibility → object becomes eligible for garbage collection
6. Finalization   → finalize() called (deprecated, unreliable)
7. Collection     → memory reclaimed by GC
```

The exam focuses heavily on **step 4 and 5** — determining exactly when an object becomes eligible for garbage collection. This requires precise understanding of reference tracking.

---

### What is Garbage Collection?

The **Garbage Collector (GC)** is a JVM subsystem that automatically reclaims memory from objects that are no longer reachable by the running program.

Key characteristics:
- GC runs **automatically** — you cannot force it (though you can request it)
- You cannot predict **when** GC will run
- You cannot predict **which** eligible objects will be collected first
- `System.gc()` is a **hint** to the JVM — not a command. The JVM may ignore it
- Objects are collected only when **no live thread can reach them**
- The exam tests **eligibility** — not when GC actually runs

---

### Reachability — The Core Concept

An object is **reachable** if it can be accessed through any chain of references starting from a **GC root**.

**GC Roots include:**
- Local variables in active method stack frames
- Static variables
- Active thread objects
- JNI references (native code)

An object is **eligible for GC** when it is **no longer reachable** from any GC root through any chain of references.

```
GC Root (static var / local var)
        ↓
    Object A  ──────→  Object B
                              ↓
                          Object C
```

If the GC root reference is removed, A, B, and C all become eligible for GC simultaneously — even though B and C are still referencing each other. The entire unreachable island sinks together.

---

### Ways an Object Becomes Eligible for GC

#### Method 1: Reference goes out of scope

```java
void method() {
    Dog d = new Dog("Rex");   // Dog created
    // ... use d
}   // d goes out of scope — Dog eligible for GC
```

When `method()` returns, the stack frame is popped. `d` is gone. Nothing else references the `Dog` object. It's eligible.

---

#### Method 2: Reference reassigned to another object

```java
Dog d = new Dog("Rex");    // Dog "Rex" created
d = new Dog("Max");        // d now points to Dog "Max"
                           // Dog "Rex" is now eligible for GC
```

After reassignment, nothing references "Rex" anymore.

---

#### Method 3: Reference explicitly set to `null`

```java
Dog d = new Dog("Rex");
d = null;    // reference severed — Dog "Rex" eligible for GC
```

---

#### Method 4: Island of Isolation (Circular References)

This is the most sophisticated GC scenario and a major exam focus.

```java
class Node {
    Node next;
}

void method() {
    Node a = new Node();
    Node b = new Node();
    a.next = b;     // a → b
    b.next = a;     // b → a (circular reference)

    a = null;       // sever external reference to a
    b = null;       // sever external reference to b
    // Both a and b are eligible for GC!
    // Even though they reference each other,
    // no GC root can reach either of them
}
```

Java's GC handles circular references correctly. It uses **reachability from GC roots**, not reference counting. Even if objects reference each other, if no GC root can reach them, they're ALL eligible.

This "island of isolation" is the key insight — a group of objects forming a closed loop, with no path from any GC root, is entirely eligible for GC.

---

#### Method 5: Object Created Within Expression (Never Stored)

```java
int len = new String("hello").length();
// The String object is immediately eligible for GC
// after length() returns — no reference was stored
```

---

### Counting References — The Exam Approach

When the exam presents a code snippet and asks "how many objects are eligible for GC?", you need to:

1. Track every object created (count `new` keywords)
2. Track every reference variable
3. Determine which objects each reference points to at the question point
4. Identify objects with zero reachable references from GC roots

```java
public static void main(String[] args) {
    String s1 = new String("a");   // Object 1: "a"
    String s2 = new String("b");   // Object 2: "b"
    String s3 = s1;                // s3 points to Object 1 — no new object

    s1 = null;      // Object 1 still referenced by s3
    s2 = null;      // Object 2 has no references — ELIGIBLE

    // At this point: Object 1 is reachable via s3, Object 2 is eligible
}
```

---

### The `finalize()` Method — Deprecated and Tricky

`Object.finalize()` was a method called by the GC **before** collecting an object. It allowed last-minute cleanup. It is **deprecated since Java 9** and removed in later versions.

Key exam facts about `finalize()`:
- Called at most **once** per object
- **Not guaranteed to be called** at all
- You cannot predict **when** it will be called
- An object can **resurrect itself** inside `finalize()` by assigning `this` to a reachable reference
- If resurrected, `finalize()` will NOT be called again when the object becomes eligible a second time
- Do NOT rely on `finalize()` for resource cleanup — use `try-with-resources` instead

```java
class Resurrection {
    static Resurrection instance;

    @Override
    protected void finalize() {
        System.out.println("finalize called");
        instance = this;    // object resurrects itself!
    }
}
```

The exam may test whether you know `finalize()` is called at most once. After resurrection and second eligibility, it won't run again.

---

### Memory Areas — Where Objects Live

Understanding WHERE things live helps understand GC behavior:

| Area | Contents | GC'd? |
|---|---|---|
| **Heap** | All objects, instance fields | ✅ Yes |
| **Stack** | Local vars, method frames | ❌ No (automatic) |
| **Method Area** | Class metadata, static vars | Rarely |
| **PC Register** | Current instruction pointer | ❌ No |

The GC manages the **heap**. Local variables on the stack are automatically destroyed when a method returns — they don't need GC.

---

### `System.gc()` and `Runtime.gc()`

```java
System.gc();          // request GC — JVM may ignore
Runtime.getRuntime().gc();  // same thing
```

These are **requests**, not commands. The JVM decides whether to honor them. On the exam:
- These calls do **not guarantee** GC runs
- They do **not guarantee** specific objects are collected
- They're not wrong to call — just unreliable

---

### Soft, Weak, and Phantom References

Java has four reference types (in `java.lang.ref`). These are rarely tested deeply on OCP 17 but you should recognize them:

| Reference Type | Class | GC Behavior |
|---|---|---|
| **Strong** | Normal variable | Object never GC'd while reachable |
| **Soft** | `SoftReference<T>` | GC'd only when memory is low |
| **Weak** | `WeakReference<T>` | GC'd at next GC cycle |
| **Phantom** | `PhantomReference<T>` | GC'd, used for cleanup actions |

```java
import java.lang.ref.*;

Dog d = new Dog("Rex");
WeakReference<Dog> weak = new WeakReference<>(d);
d = null;  // strong reference removed
// weak.get() might return null after next GC
```

The OCP exam primarily tests **strong references** and basic eligibility. Weak/soft/phantom are more relevant to professional development.

---

### Object Eligibility — Tricky Scenarios

#### Scenario 1: Object referenced only by another eligible object

```java
class Tree {
    Branch branch;
}
class Branch {
    Leaf leaf;
}
class Leaf { }

void method() {
    Tree t = new Tree();
    t.branch = new Branch();
    t.branch.leaf = new Leaf();

    t = null;   // Tree eligible
                // Branch eligible (only referenced by Tree)
                // Leaf eligible (only referenced by Branch)
}
```

When the root becomes eligible, the entire object graph hanging off it also becomes eligible.

---

#### Scenario 2: Shared reference keeps object alive

```java
void method() {
    StringBuilder sb1 = new StringBuilder("hello");
    StringBuilder sb2 = sb1;   // both point to same object

    sb1 = null;    // object still alive — sb2 references it
    sb2 = null;    // NOW eligible — no more references
}
```

---

#### Scenario 3: Static reference keeps object alive forever

```java
class Cache {
    static List<Object> items = new ArrayList<>();

    static void add(Object obj) {
        items.add(obj);
    }
}
```

Any object added to `Cache.items` will NOT be eligible for GC as long as the `Cache` class is loaded — because `items` is a static field, which is a GC root. This is a common source of **memory leaks** in Java.

---

#### Scenario 4: Object returned from method

```java
StringBuilder create() {
    StringBuilder sb = new StringBuilder("data");
    return sb;    // reference passed to caller — NOT eligible
}

void caller() {
    StringBuilder result = create();  // result holds the reference
    // sb from create() is NOT eligible — result still holds it
}
// result goes out of scope here — NOW eligible
```

---

#### Scenario 5: Objects in Collections

```java
void method() {
    List<Dog> dogs = new ArrayList<>();
    dogs.add(new Dog("Rex"));    // Dog referenced by list
    dogs.add(new Dog("Max"));    // Dog referenced by list

    dogs = null;    // ArrayList eligible
                    // Both Dog objects eligible (list was only reference)
}
```

But if individual references are kept:

```java
void method() {
    Dog rex = new Dog("Rex");
    List<Dog> dogs = new ArrayList<>();
    dogs.add(rex);

    dogs = null;   // ArrayList eligible
                   // Dog "Rex" NOT eligible — rex variable still references it
}
```

---

### The Exam's GC Question Patterns

The exam typically asks one of:
1. "After line X, how many objects are eligible for GC?"
2. "At the end of the method, which objects are eligible for GC?"
3. "Which statement about GC is true?"

For questions 1 and 2, draw a diagram and trace references carefully.

For question 3, common true statements:
- The GC runs on the heap
- `System.gc()` is only a suggestion
- Objects with circular references can still be GC'd
- `finalize()` is called at most once
- You cannot control when GC runs

Common false statements Oracle presents:
- "Calling `System.gc()` guarantees immediate collection"
- "Objects with circular references are never GC'd"
- "`finalize()` is always called before an object is collected"
- "Setting a reference to null immediately frees memory"

---

## 2. Code Examples

### Example 1 — Basic Eligibility

```java
public class GCBasic {
    public static void main(String[] args) {
        // Create objects
        Object o1 = new Object();    // Object 1
        Object o2 = new Object();    // Object 2
        Object o3 = o1;              // No new object — o3 points to Object 1

        o1 = null;    // Object 1 still referenced by o3 — NOT eligible
        o2 = null;    // Object 2 has no references — ELIGIBLE (1 object)
        o3 = null;    // Object 1 now has no references — ELIGIBLE (2 objects total)
    }
}
```

---

### Example 2 — Island of Isolation

```java
public class IslandDemo {
    Object ref;

    public static void main(String[] args) {
        IslandDemo a = new IslandDemo();   // Object A
        IslandDemo b = new IslandDemo();   // Object B

        a.ref = b;    // A → B
        b.ref = a;    // B → A (circular)

        a = null;     // external reference to A removed
        b = null;     // external reference to B removed

        // Both A and B are eligible for GC
        // Even though A.ref → B and B.ref → A
        // No GC root can reach either
        // How many eligible? 2
    }
}
```

---

### Example 3 — Method Return Keeps Object Alive

```java
public class ReturnDemo {
    static Object createObject() {
        Object obj = new Object();   // created here
        return obj;                  // reference transferred to caller
    }

    public static void main(String[] args) {
        Object result = createObject();
        // obj is NOT eligible — result holds it

        result = null;
        // NOW eligible
    }
}
```

---

### Example 4 — Static Reference Memory Leak

```java
import java.util.*;

public class MemoryLeak {
    static List<byte[]> cache = new ArrayList<>();

    public static void main(String[] args) {
        for (int i = 0; i < 1000; i++) {
            byte[] data = new byte[1024 * 1024];  // 1MB
            cache.add(data);   // static list holds reference
            // data local var goes out of scope, BUT
            // cache (static) still references each byte array
            // NONE of these are eligible for GC!
        }
        // 1000 MB still in memory — potential OutOfMemoryError
    }
}
```

---

### Example 5 — Counting Eligible Objects

```java
public class CountEligible {
    String value;

    public static void main(String[] args) {
        CountEligible a = new CountEligible();  // Object 1
        CountEligible b = new CountEligible();  // Object 2
        CountEligible c = new CountEligible();  // Object 3

        a.value = "x";   // String "x" — Object 4
        b.value = a.value;  // b.value points to same "x" — no new object

        a = b;    // Object 1 loses reference — ELIGIBLE (1 eligible)
                  // a and b both point to Object 2

        b = null; // Object 2 still referenced by (new) a — NOT eligible

        c = null; // Object 3 — ELIGIBLE (2 eligible now)

        // At this point: 2 objects eligible (Object 1, Object 3)
        // Object 2 still referenced by a
        // String "x" still referenced by Object 2's value field
    }
}
```

---

### Example 6 — Finalize Behavior

```java
public class FinalizeDemo {
    private String name;

    FinalizeDemo(String name) {
        this.name = name;
    }

    @Override
    protected void finalize() throws Throwable {
        System.out.println(name + " finalized");
        super.finalize();
    }

    public static void main(String[] args) throws InterruptedException {
        FinalizeDemo obj = new FinalizeDemo("MyObject");
        obj = null;             // eligible for GC

        System.gc();            // request GC — may or may not run finalize
        Thread.sleep(100);      // give GC a chance to run

        // Output is not guaranteed — finalize may or may not have run
        System.out.println("Done");
    }
}
```

---

### Example 7 — Object Graph Eligibility

```java
class Engine { }
class Wheel { }
class Car {
    Engine engine = new Engine();
    Wheel[] wheels = new Wheel[4];

    Car() {
        for (int i = 0; i < 4; i++) {
            wheels[i] = new Wheel();
        }
    }
}

public class GraphDemo {
    public static void main(String[] args) {
        Car car = new Car();
        // Objects: 1 Car + 1 Engine + 1 Wheel[] + 4 Wheels = 7 objects

        car = null;
        // All 7 objects become eligible for GC
        // Car → Engine: eligible
        // Car → Wheel[]: eligible
        // Wheel[] → 4 Wheels: all eligible
    }
}
```

---

### Example 8 — Collection and Individual References

```java
import java.util.*;

public class CollectionGC {
    public static void main(String[] args) {
        String s1 = new String("hello");    // Object 1
        String s2 = new String("world");    // Object 2
        List<String> list = new ArrayList<>();  // Object 3
        list.add(s1);
        list.add(s2);

        s1 = null;   // Object 1 NOT eligible — list still holds it
        list = null; // Object 3 (ArrayList) eligible
                     // Object 1 eligible — list was last reference
                     // Object 2 NOT eligible — s2 still holds it

        // Eligible: Object 1, Object 3 (2 objects)
        // Not eligible: Object 2

        s2 = null;   // Now Object 2 also eligible
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
        Object a = new Object();   // Object 1
        Object b = new Object();   // Object 2
        a = b;
        // How many objects are eligible for GC here?
    }
}
```
- A. 0
- B. 1
- C. 2
- D. Cannot be determined

**Answer: B**
`a = b` makes `a` point to Object 2. Object 1 now has no references — 1 object eligible. Object 2 is still referenced by both `a` and `b`.

---

**Q2 — MCQ**
```java
class Node {
    Node next;
}

public class Test {
    public static void main(String[] args) {
        Node n1 = new Node();   // Object 1
        Node n2 = new Node();   // Object 2
        n1.next = n2;
        n2.next = n1;
        n1 = null;
        n2 = null;
        // How many objects are eligible for GC?
    }
}
```
- A. 0
- B. 1
- C. 2
- D. Cannot be determined — circular references prevent GC

**Answer: C**
Both n1 and n2 are null — no GC roots reach either object. Circular references do NOT prevent GC. Both are eligible.

---

**Q3 — Select All That Apply**
Which of the following statements about garbage collection are true? (Choose all that apply)

- A. Calling `System.gc()` guarantees immediate garbage collection
- B. Objects with circular references can be garbage collected
- C. `finalize()` is guaranteed to be called before an object is collected
- D. Setting a reference to `null` makes an object immediately eligible for GC if no other references exist
- E. The garbage collector runs on the heap
- F. Static variables are GC roots

**Answer: B, D, E, F**
A false — `System.gc()` is a hint only. C false — `finalize()` is not guaranteed. B, D, E, F are all correct.

---

**Q4 — MCQ**
```java
public class Test {
    public static void main(String[] args) {
        String s = new String("hello");
        s.concat(" world");   // Line A
        System.out.println(s);
    }
}
```
How many `String` objects are eligible for GC after Line A?

- A. 0
- B. 1
- C. 2
- D. 3

**Answer: B**
`s.concat(" world")` creates a NEW String object but the result is not stored. That new String is immediately eligible. `s` still holds `"hello"`. Also note: `" world"` is a string literal from the pool — not eligible. So 1 String object (`"hello world"` from `concat`) is eligible.

---

**Q5 — MCQ**
Which of the following correctly describes `finalize()`?

- A. It is called exactly once before every object is collected
- B. It is called at most once per object lifetime
- C. Calling `System.gc()` guarantees `finalize()` will be called
- D. `finalize()` must be implemented to enable garbage collection

**Answer: B**
`finalize()` is called AT MOST once — never guaranteed to be called at all. No implementation required for GC to work.

---

**Q6 — MCQ**
```java
public class Test {
    static Object obj;

    public static void main(String[] args) {
        obj = new Object();   // Object 1
        Object local = obj;   // local points to Object 1

        obj = null;           // Line A
        // Is Object 1 eligible after Line A?
    }
}
```
After Line A, is Object 1 eligible for GC?

- A. Yes — `obj` was set to null
- B. No — `local` still references it
- C. Yes — static variables don't count as references
- D. Cannot be determined

**Answer: B**
`local` is a local variable in `main` that still references Object 1. Even though `obj` (static) is null, `local` keeps it alive. Not eligible.

---

**Q7 — MCQ**
```java
public class Test {
    public static void main(String[] args) {
        Object o1 = new Object();   // Object 1
        Object o2 = new Object();   // Object 2
        Object o3 = new Object();   // Object 3

        o2 = o3;
        o3 = o1;
        o1 = o2;

        // How many objects are eligible for GC?
    }
}
```
- A. 0
- B. 1
- C. 2
- D. 3

**Answer: B**
Trace the references:
- Initial: o1→1, o2→2, o3→3
- `o2 = o3`: o1→1, o2→3, o3→3
- `o3 = o1`: o1→1, o2→3, o3→1
- `o1 = o2`: o1→3, o2→3, o3→1

After all reassignments: Object 1 referenced by o3. Object 3 referenced by o1 and o2. Object 2 has no references — **1 object eligible**.

---

**Q8 — Select All That Apply**
Which of the following make an object eligible for GC? (Choose all that apply)

- A. Setting the only reference to `null`
- B. Reassigning the only reference to point to another object
- C. The reference going out of scope with no other references existing
- D. The object having no instance variables
- E. Calling `System.gc()`
- F. All references to the object forming an island of isolation

**Answer: A, B, C, F**
D is irrelevant — having no fields doesn't affect GC eligibility. E just requests GC — doesn't make objects eligible. A, B, C, F all correctly describe ways objects become eligible.

---

**Q9 — MCQ**
```java
import java.util.*;

public class Test {
    public static void main(String[] args) {
        List<Object> list = new ArrayList<>();
        Object obj = new Object();      // Object 1
        list.add(obj);

        obj = null;    // Line A

        // After Line A, is Object 1 eligible for GC?
    }
}
```
- A. Yes — `obj` was set to null
- B. No — `list` still holds a reference to it
- C. Yes — lists don't prevent GC
- D. Cannot be determined

**Answer: B**
`list.add(obj)` puts a reference to Object 1 inside the ArrayList. Setting `obj = null` removes one reference, but the ArrayList still holds another. Object 1 is NOT eligible.

---

**Q10 — MCQ**
Which is true about the `finalize()` method?

- A. It is called immediately when an object becomes eligible for GC
- B. It allows an object to resurrect itself by storing `this` in a reachable reference
- C. It will be called twice if an object becomes eligible twice
- D. It must call `super.finalize()` or a compile error occurs

**Answer: B**
An object CAN resurrect itself inside `finalize()`. A is false — timing is not guaranteed. C is false — called at most once. D is false — `super.finalize()` is a convention, not a requirement (though good practice).

---

## 4. Trick Analysis

### Trap 1: Circular references are GC'd
A major misconception candidates bring from other languages. Java does NOT use reference counting — it uses reachability from GC roots. Circular references in an island with no GC root path are entirely eligible for GC. The exam presents circular references and asks if they can be collected — they can.

---

### Trap 2: `System.gc()` does not guarantee collection
`System.gc()` is a polite request. The JVM can and often does ignore it, especially in performance-sensitive situations. The exam frequently presents this as "correct" when it's actually just a hint. Any answer saying GC is "guaranteed" after `System.gc()` is wrong.

---

### Trap 3: Object still alive because it's in a collection
```java
list.add(obj);
obj = null;   // NOT eligible — list still holds it
```
Setting your own reference to null doesn't help if other references (like inside a collection) still point to the object. The exam loves this scenario.

---

### Trap 4: `finalize()` is called at most once — not always
Two separate traps:
1. Not guaranteed to be called AT ALL
2. Even if called once, won't be called again after resurrection

The exam presents both: "finalize is guaranteed" (false) and "finalize can be called multiple times" (false).

---

### Trap 5: Method creates object, returns it — not eligible
```java
Object o = createObject();  // o holds the returned reference
```
An object returned from a method is NOT eligible — the caller's variable holds it. Only when the caller's variable also loses the reference does eligibility occur.

---

### Trap 6: Counting `new` keywords for object count
The exam asks "how many objects are created?" Count each `new` keyword. Also remember:
- String literals (`"hello"`) create String pool objects
- Autoboxing creates wrapper objects (`Integer.valueOf()`)
- Concatenation can create StringBuilder objects

---

### Trap 7: Static fields are GC roots
Any object referenced (directly or transitively) by a static field is NOT eligible for GC while the class is loaded. This is a common memory leak pattern — static collections that grow unboundedly.

---

### Trap 8: Reassignment reference trace
```java
Object a = new Object();  // 1
Object b = new Object();  // 2
a = b;  // a→2, 1 is eligible
b = a;  // b→2, still — no change in eligibility
```
Tracing reference reassignments step by step is essential. The exam deliberately chains multiple reassignments to confuse.

---

## 5. Summary Sheet

### Ways to Make an Object Eligible for GC

| Method | Example |
|---|---|
| Set reference to `null` | `obj = null;` |
| Reassign reference | `obj = otherObj;` |
| Reference goes out of scope | Method returns, block ends |
| Island of isolation | Circular refs with no GC root path |
| Object never stored | `new Object().toString();` |

---

### GC Roots (Keep Objects Alive)

| GC Root Type | Example |
|---|---|
| Local variables (active methods) | `Object o` in a running method |
| Static fields | `static Object cache` |
| Active thread objects | Running `Thread` instances |
| JNI references | Native code references |

---

### Key GC Facts

| Statement | True or False |
|---|---|
| `System.gc()` guarantees immediate collection | ❌ False |
| Circular references prevent GC | ❌ False |
| `finalize()` is always called before collection | ❌ False |
| `finalize()` is called at most once per object | ✅ True |
| An object can resurrect itself in `finalize()` | ✅ True |
| GC manages the heap | ✅ True |
| Setting to `null` frees memory immediately | ❌ False |
| Static fields are GC roots | ✅ True |
| Objects in collections can be GC'd | ✅ True (if collection also unreachable) |

---

### Object Eligibility Decision Tree

```
Is there ANY path from a GC root to the object?
        ↓
       YES → NOT eligible for GC
        ↓
       NO  → ELIGIBLE for GC

GC roots: static fields, local vars in active frames,
          active threads, JNI refs
```

---

### `finalize()` Key Points

| Point | Detail |
|---|---|
| Called by | GC — before collection |
| Guaranteed? | No |
| Called how many times? | At most once |
| Can object be resurrected? | Yes — by storing `this` |
| After resurrection, called again? | No |
| Deprecated since | Java 9 |
| Replacement | `try-with-resources`, `Cleaner` API |

---

### Common Exam Scenarios

| Scenario | Eligible Count |
|---|---|
| `obj = null` (only reference) | 1 |
| `a = b` (only reference to original `a`'s object) | 1 |
| Circular ref with no GC root | All in the island |
| Object in collection, external ref nulled | 0 (collection still holds it) |
| Collection nulled, no external refs | All objects in collection |
| Returned from method, stored by caller | 0 |
| Created inline, never stored | 1 (immediately) |

---

## Domain 1 Complete — Congratulations!

You have now covered all 10 topics of **Domain 1: Java Building Blocks**:

- ✅ 1.1 Java Environment (JDK, JRE, JVM)
- ✅ 1.2 Class Structure
- ✅ 1.3 Main Method
- ✅ 1.4 Packages & Imports
- ✅ 1.5 Object References vs Primitives
- ✅ 1.6 Primitive Data Types
- ✅ 1.7 Variable Declarations & Identifiers
- ✅ 1.8 `var` — Type Inference
- ✅ 1.9 Variable Scope & Lifetime
- ✅ 1.10 Object Lifecycle & GC

---
