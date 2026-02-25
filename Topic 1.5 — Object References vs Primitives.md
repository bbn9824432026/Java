# Topic 1.5 — Object References vs Primitives

---

## 1. Conceptual Explanation

### The Two Kinds of Variables in Java

Every variable in Java is one of two things:

1. **A primitive** — stores the actual value directly
2. **A reference** — stores a memory address pointing to an object on the heap

This distinction affects assignment behavior, method passing, equality comparison, default values, and memory layout. It is tested throughout the entire OCP exam — not just in this topic.

---

### Primitives

Java has exactly **8 primitive types**. They are not objects. They do not have methods. They are stored directly in the variable's memory location (stack for locals, inline in the object for instance fields).

| Type | Size | Range | Default |
|---|---|---|---|
| `byte` | 8 bits | -128 to 127 | 0 |
| `short` | 16 bits | -32,768 to 32,767 | 0 |
| `int` | 32 bits | -2,147,483,648 to 2,147,483,647 | 0 |
| `long` | 64 bits | -2^63 to 2^63 - 1 | 0L |
| `float` | 32 bits | ~±3.4 × 10^38 (7 decimal digits) | 0.0f |
| `double` | 64 bits | ~±1.8 × 10^308 (15 decimal digits) | 0.0 |
| `char` | 16 bits | 0 to 65,535 (Unicode) | '\u0000' |
| `boolean` | JVM-defined | true / false | false |

> **Exam note:** `boolean` size is not precisely defined by the Java spec — it's JVM-dependent. The exam won't ask you the bit size of `boolean`, but it will test its default value (`false`) and valid literals (`true`/`false` only — not `0`/`1` like C).

---

### References

A reference variable stores the **memory address** of an object on the heap. The variable itself does not contain the object — it points to it.

```java
Dog d = new Dog();
```

What actually happens in memory:

```
Stack                    Heap
┌─────────┐             ┌──────────────┐
│    d    │────────────▶│  Dog object  │
│ (addr)  │             │  name: null  │
└─────────┘             │  age: 0      │
                        └──────────────┘
```

`d` holds a memory address (a reference). The actual `Dog` object lives on the heap. `d` does not "contain" the dog — it points to it.

---

### The `null` Reference

A reference variable that doesn't point to any object holds `null`.

```java
Dog d = null;    // d points to nothing
d.bark();        // NullPointerException at runtime
```

`null` is not an object. It is a literal that represents the absence of a reference. You can assign `null` to any reference type. You cannot assign `null` to a primitive.

```java
int x = null;    // COMPILE ERROR — primitives cannot be null
Dog d = null;    // fine
```

---

### Assignment Behavior — The Critical Difference

#### Primitives: Copy the Value

```java
int a = 5;
int b = a;    // b gets a COPY of the value 5
b = 10;
System.out.println(a);  // 5 — a is unchanged
System.out.println(b);  // 10
```

Changing `b` has no effect on `a`. They are completely independent copies.

#### References: Copy the Reference (Not the Object)

```java
Dog d1 = new Dog("Rex");
Dog d2 = d1;              // d2 gets a COPY of the reference (same address)
d2.name = "Max";
System.out.println(d1.name);  // Max — SAME object was modified
System.out.println(d2.name);  // Max
```

```
Stack                    Heap
┌─────────┐             ┌──────────────┐
│   d1    │────────────▶│  Dog object  │
└─────────┘       ┌────▶│  name: "Max" │
┌─────────┐       │     └──────────────┘
│   d2    │───────┘
└─────────┘
```

Both `d1` and `d2` point to the **same object**. Modifying the object through one reference affects what you see through the other.

---

### Pass-by-Value — Java Always Passes by Value

This is one of the most misunderstood concepts in Java and a favorite OCP topic.

**Java is always pass-by-value.** Always. No exceptions.

The confusion comes from what "the value" is for reference types:
- For primitives: the value IS the data
- For references: the value IS the memory address

So when you pass a reference to a method, you pass a **copy of the address**. The method can use that address to modify the object the address points to — but it cannot change what address the original variable holds.

```java
public class PassByValue {

    static void changePrimitive(int x) {
        x = 99;                    // modifies the local copy only
    }

    static void changeObject(Dog d) {
        d.name = "Buddy";          // modifies the object at the address
    }

    static void changeReference(Dog d) {
        d = new Dog("NewDog");     // only changes the LOCAL copy of the address
    }

    public static void main(String[] args) {
        int num = 5;
        changePrimitive(num);
        System.out.println(num);   // 5 — unchanged

        Dog dog = new Dog("Rex");
        changeObject(dog);
        System.out.println(dog.name);   // Buddy — object was modified

        Dog dog2 = new Dog("Rex");
        changeReference(dog2);
        System.out.println(dog2.name);  // Rex — reference unchanged
    }
}
```

The third case is critical: `changeReference` assigns a new `Dog` to its local copy of `d`. This does not affect `dog2` in `main` — `dog2` still points to the original `Dog("Rex")` object.

---

### Equality: `==` Means Different Things

#### For Primitives: `==` compares values

```java
int a = 5;
int b = 5;
System.out.println(a == b);   // true — same value
```

#### For References: `==` compares addresses

```java
Dog d1 = new Dog("Rex");
Dog d2 = new Dog("Rex");
Dog d3 = d1;

System.out.println(d1 == d2);   // false — different objects, different addresses
System.out.println(d1 == d3);   // true — same address
```

Even though `d1` and `d2` have the same data, they are different objects at different memory locations. `==` compares where they live, not what they contain.

To compare object **content**, use `.equals()`:

```java
System.out.println(d1.equals(d2));  // depends on how equals() is implemented
```

`String` is a special case because it overrides `equals()`:

```java
String s1 = new String("hello");
String s2 = new String("hello");
System.out.println(s1 == s2);       // false — different objects
System.out.println(s1.equals(s2));  // true — same content
```

The String Pool adds another layer (covered in Topic 3.1), but the underlying reference vs primitive distinction is the foundation.

---

### Stack vs Heap Memory

| Location | What Lives There |
|---|---|
| **Stack** | Local variables (primitives + references), method call frames |
| **Heap** | All objects (including arrays, Strings) |

A local primitive variable lives entirely on the stack. A local reference variable lives on the stack, but the object it points to lives on the heap.

Instance fields (whether primitive or reference) live **inside the object on the heap**.

```java
public class Box {
    int size;          // primitive — lives on heap INSIDE the Box object
    Dog dog;           // reference — lives on heap INSIDE Box; Dog object also on heap
}
```

---

### Wrapper Classes — Bridging the Gap

Every primitive has a corresponding wrapper class that IS an object:

| Primitive | Wrapper |
|---|---|
| `byte` | `Byte` |
| `short` | `Short` |
| `int` | `Integer` |
| `long` | `Long` |
| `float` | `Float` |
| `double` | `Double` |
| `char` | `Character` |
| `boolean` | `Boolean` |

Wrapper classes allow primitives to be used where objects are required (e.g., in collections). This involves **autoboxing** and **unboxing** (covered in depth in Topic 3.8), but the connection to this topic is important:

```java
List<int> list;        // COMPILE ERROR — generics require reference types
List<Integer> list;    // OK — Integer is a reference type
```

Primitives cannot be used as generic type parameters. Wrapper classes solve this.

---

### Reference Types Include More Than Just Classes

Any non-primitive type is a reference type:
- Class instances (`Dog`, `String`, `ArrayList`)
- Arrays (even `int[]` is a reference type — the array object lives on the heap)
- Interfaces (as variable types)
- Enums
- Records

```java
int[] arr = new int[5];      // arr is a reference — the array lives on heap
int[] arr2 = arr;            // arr2 points to the same array
arr2[0] = 99;
System.out.println(arr[0]);  // 99 — same array modified
```

---

## 2. Code Examples

### Example 1 — Primitive Assignment is Independent

```java
public class PrimitiveCopy {
    public static void main(String[] args) {
        int x = 100;
        int y = x;      // y gets a copy of 100
        y = 200;
        System.out.println(x);  // 100 — unaffected
        System.out.println(y);  // 200
    }
}
```

---

### Example 2 — Reference Assignment Shares the Object

```java
class Cat {
    String name;
    Cat(String name) { this.name = name; }
}

public class ReferenceCopy {
    public static void main(String[] args) {
        Cat c1 = new Cat("Whiskers");
        Cat c2 = c1;             // c2 points to the same Cat object

        c2.name = "Shadow";
        System.out.println(c1.name);  // Shadow — same object
        System.out.println(c2.name);  // Shadow

        c2 = new Cat("Luna");    // c2 now points to a NEW Cat
        System.out.println(c1.name);  // Shadow — c1 unaffected
        System.out.println(c2.name);  // Luna
    }
}
```

---

### Example 3 — Pass-by-Value with Primitive

```java
public class PrimPass {
    static void doubleIt(int n) {
        n = n * 2;           // only local copy changes
    }

    public static void main(String[] args) {
        int val = 10;
        doubleIt(val);
        System.out.println(val);  // 10 — original unchanged
    }
}
```

---

### Example 4 — Pass-by-Value with Reference (Object Mutation)

```java
class Counter {
    int count = 0;
}

public class RefPass {
    static void increment(Counter c) {
        c.count++;            // modifies the object at the address
    }

    public static void main(String[] args) {
        Counter counter = new Counter();
        increment(counter);
        System.out.println(counter.count);  // 1 — object was mutated
    }
}
```

---

### Example 5 — Pass-by-Value: Reassigning Reference Has No Effect on Caller

```java
class Counter {
    int count = 0;
}

public class RefReassign {
    static void reset(Counter c) {
        c = new Counter();    // local c now points elsewhere
        c.count = 99;         // modifies the new local object only
    }

    public static void main(String[] args) {
        Counter counter = new Counter();
        counter.count = 42;
        reset(counter);
        System.out.println(counter.count);  // 42 — original unchanged
    }
}
```

---

### Example 6 — `==` on References vs Primitives

```java
public class EqualityDemo {
    public static void main(String[] args) {
        // Primitives — == compares values
        int a = 42;
        int b = 42;
        System.out.println(a == b);   // true

        // References — == compares addresses
        String s1 = new String("hello");
        String s2 = new String("hello");
        System.out.println(s1 == s2);      // false — different objects
        System.out.println(s1.equals(s2)); // true — same content

        // Null check
        String s3 = null;
        System.out.println(s3 == null);    // true
        // System.out.println(s3.equals("hi")); // NullPointerException
    }
}
```

---

### Example 7 — Arrays are References

```java
public class ArrayRef {
    public static void main(String[] args) {
        int[] arr1 = {1, 2, 3};
        int[] arr2 = arr1;       // arr2 points to same array

        arr2[0] = 99;
        System.out.println(arr1[0]);  // 99 — same array

        System.out.println(arr1 == arr2);  // true — same reference
    }
}
```

---

### Example 8 — Null Assignment

```java
public class NullDemo {
    public static void main(String[] args) {
        String s = null;
        // int x = null;      // COMPILE ERROR
        System.out.println(s);         // prints: null
        System.out.println(s == null); // true

        s = "hello";
        System.out.println(s == null); // false
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
    public static void main(String[] args) {
        int x = 10;
        int y = x;
        y += 5;
        System.out.println(x + " " + y);
    }
}
```
- A. `10 10`
- B. `15 15`
- C. `10 15`
- D. `15 10`

**Answer: C**
`y` is an independent copy. Modifying `y` doesn't affect `x`.

---

**Q2 — MCQ**
What is the output?
```java
class Box {
    int value;
}

public class Test {
    static void modify(Box b) {
        b.value = 100;
    }

    public static void main(String[] args) {
        Box box = new Box();
        box.value = 5;
        modify(box);
        System.out.println(box.value);
    }
}
```
- A. 5
- B. 100
- C. 0
- D. Compile error

**Answer: B**
The method receives a copy of the reference, uses it to access the same object, and modifies `value`. The caller sees the change.

---

**Q3 — MCQ**
What is the output?
```java
class Box {
    int value;
}

public class Test {
    static void replace(Box b) {
        b = new Box();
        b.value = 100;
    }

    public static void main(String[] args) {
        Box box = new Box();
        box.value = 5;
        replace(box);
        System.out.println(box.value);
    }
}
```
- A. 5
- B. 100
- C. 0
- D. Compile error

**Answer: A**
`replace` reassigns its local copy of `b` to a new `Box`. The original `box` in `main` is unaffected. Java is pass-by-value — the reference itself cannot be changed from inside a method.

---

**Q4 — Select All That Apply**
Which of the following can be assigned `null`? (Choose all that apply)

- A. `int`
- B. `String`
- C. `double`
- D. `int[]`
- E. `boolean`
- F. `Object`

**Answer: B, D, F**
Only reference types can hold `null`. `int`, `double`, `boolean` are primitives and cannot be null. `String`, `int[]` (array is a reference type), and `Object` are all reference types.

---

**Q5 — MCQ**
```java
public class Test {
    public static void main(String[] args) {
        String a = new String("Java");
        String b = new String("Java");
        System.out.println(a == b);
        System.out.println(a.equals(b));
    }
}
```
- A. `true` / `true`
- B. `false` / `false`
- C. `true` / `false`
- D. `false` / `true`

**Answer: D**
`==` compares references — two different `new String()` objects → `false`. `.equals()` compares content → `true`.

---

**Q6 — MCQ**
Which statement is true about Java's parameter passing?

- A. Java passes objects by reference
- B. Java passes primitives by value and objects by reference
- C. Java always passes by value
- D. Java passes by reference for arrays only

**Answer: C**
Java is **always** pass-by-value. For objects, the value being passed is the reference (memory address), not the object itself. This is still pass-by-value.

---

**Q7 — MCQ**
What is the output?
```java
public class Test {
    public static void main(String[] args) {
        int[] a = {1, 2, 3};
        int[] b = a;
        b[1] = 99;
        System.out.println(a[1]);
    }
}
```
- A. 2
- B. 99
- C. Compile error
- D. ArrayIndexOutOfBoundsException

**Answer: B**
`b = a` makes `b` point to the same array. Modifying `b[1]` modifies the shared array. `a[1]` reflects the change.

---

**Q8 — Select All That Apply**
Which statements about `==` are correct? (Choose all that apply)

- A. For primitives, `==` compares values
- B. For references, `==` compares object contents
- C. For references, `==` compares memory addresses
- D. `null == null` evaluates to `true`
- E. `==` can be used to compare a primitive and a reference type

**Answer: A, C, D**
B is wrong — `==` on references compares addresses, not contents. D is true — `null == null` is `true`. E: comparing a primitive and reference type may cause unboxing (e.g., `Integer` vs `int`), which works but is not general — the exam treats this carefully (covered in autoboxing topic).

---

**Q9 — MCQ**
```java
public class Test {
    public static void main(String[] args) {
        String s = null;
        System.out.println(s);
    }
}
```
- A. Compile error
- B. Runtime NullPointerException
- C. Prints `null`
- D. Prints empty string

**Answer: C**
`System.out.println(null reference)` prints the string `"null"` — it doesn't throw NPE. `println` handles null references by printing the text "null".

---

**Q10 — Tricky MCQ**
```java
public class Test {
    public static void main(String[] args) {
        String s = null;
        System.out.println(s.toString());
    }
}
```
- A. Prints `null`
- B. Prints empty string
- C. Compile error
- D. NullPointerException

**Answer: D**
Calling `.toString()` on a `null` reference throws `NullPointerException`. Contrast with Q9 where `println` receives the reference directly and handles null internally.

---

## 4. Trick Analysis

### Trap 1: Java passes objects by reference
This is the most common misconception in all of Java. Java **never** passes by reference. It passes the value of the reference (the memory address). The distinction matters: you can mutate the object a reference points to, but you cannot change what object the caller's variable points to. Oracle frames questions specifically to test this — showing a method that reassigns a parameter and asking if the caller's variable changes. It doesn't.

---

### Trap 2: `==` on Strings
`new String("hello") == new String("hello")` is `false`. This surprises candidates who think string comparison with `==` "just works." It doesn't — `==` on references is always address comparison. `equals()` is for content. The String pool (topic 3.1) adds nuance, but the fundamental rule here is address comparison.

---

### Trap 3: Arrays are reference types
`int[]` is a reference type even though `int` is a primitive. The array object lives on the heap. Assigning one array variable to another copies the reference, not the array contents. This means modifying elements through one variable affects what the other sees.

---

### Trap 4: `println(null reference)` doesn't throw NPE
`System.out.println(someNullReference)` prints `"null"` — it does not throw `NullPointerException`. But `someNullReference.anyMethod()` does throw NPE. The exam uses this to test whether candidates understand that `println` receives the reference value (null) and handles it gracefully internally.

---

### Trap 5: `null` cannot be assigned to primitives
`int x = null` is a **compile error**. Only reference types can hold null. This seems obvious but the exam embeds it in longer code snippets where you might miss it.

---

### Trap 6: Instance primitive fields live on the heap
Candidates assume "primitives live on the stack." This is only true for **local variables**. Instance fields — even primitive ones — live inside the object on the **heap**. The stack/heap distinction is about where the variable's storage is, and instance fields are stored within heap-allocated objects.

---

## 5. Summary Sheet

### Primitives vs References

| Aspect | Primitive | Reference |
|---|---|---|
| Stores | Actual value | Memory address (pointer) |
| Default (field) | 0, false, '\u0000' | `null` |
| Default (local) | None — compile error if uninitialized | None — compile error if uninitialized |
| Can be `null` | ❌ | ✅ |
| `==` compares | Values | Memory addresses |
| Assignment copies | The value | The address |
| Method passing | Copy of value | Copy of address |
| Lives (local var) | Stack | Reference on stack, object on heap |
| Lives (instance field) | Inside object on heap | Inside object on heap |

---

### The 8 Primitives

| Type | Size | Key Range/Values |
|---|---|---|
| `byte` | 8-bit | -128 to 127 |
| `short` | 16-bit | -32,768 to 32,767 |
| `int` | 32-bit | ~-2.1B to 2.1B |
| `long` | 64-bit | Very large — needs `L` suffix |
| `float` | 32-bit | Decimal — needs `f` suffix |
| `double` | 64-bit | Default decimal type |
| `char` | 16-bit | 0 to 65,535 (Unicode) |
| `boolean` | JVM-defined | `true` / `false` only |

---

### Pass-by-Value Rules

| What You Pass | Method Can... | Method Cannot... |
|---|---|---|
| Primitive | Read the copy | Affect the original |
| Reference | Mutate the object it points to | Change what object the caller's variable points to |

---

### Key Rules to Memorize

| Rule | Detail |
|---|---|
| Java is always pass-by-value | Even for objects — the reference (address) is copied |
| `==` on references | Compares addresses, not content |
| `null` | Only for reference types — primitives cannot be null |
| Calling method on null | `NullPointerException` at runtime |
| `println(null ref)` | Prints `"null"` — no NPE |
| Arrays | Always reference types — even `int[]` |
| Instance primitives | Live on the heap (inside the object) |
| Local primitives | Live on the stack |

---

