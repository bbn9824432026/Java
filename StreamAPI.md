# 🎯 **Java 8 Streams – Complete Mnemonics Cheat Sheet** 🚀  

Here’s a **mnemonic-based cheat sheet** to help you remember **ALL** the methods in **Java 8 Streams API** quickly and easily!  

---

## 🛠️ **1️⃣ Getting a Stream**
**Mnemonic:** _"List, Arrays, Of, Lines, Random, Builder, Iterate, Generate"_
| **Method** | **What it does?** | **Example** |
|------------|------------------|------------|
| `Collection.stream()` | Stream from collection | `list.stream()` |
| `Collection.parallelStream()` | Parallel stream from collection | `list.parallelStream()` |
| `Arrays.stream(array)` | Stream from array | `Arrays.stream(arr)` |
| `Stream.of(...)` | Stream from values | `Stream.of(1, 2, 3)` |
| `Stream.builder()` | Custom stream builder | `Stream.<String>builder().add("a").add("b").build()` |
| `Stream.iterate(seed, f)` | Infinite stream (sequence) | `Stream.iterate(0, n -> n + 2)` |
| `Stream.generate(Supplier)` | Infinite stream (random values) | `Stream.generate(Math::random)` |
| `Files.lines(Path)` | Stream of file lines | `Files.lines(Path.of("file.txt"))` |

---

## 🎛️ **2️⃣ Intermediate Operations (Transform Data)**
**Mnemonic:** _"Filter → Map → FlatMap → Peek → Sorted → Distinct → Limit → Skip"_

| **Method**  | **What it does?** | **Example** |
|------------|------------------|------------|
| `filter(Predicate)` | Filters elements | `.filter(n -> n > 10)` |
| `map(Function)` | Transforms elements | `.map(String::toUpperCase)` |
| `flatMap(Function)` | Flattens nested streams | `.flatMap(List::stream)` |
| `peek(Consumer)` | Debugging/logging | `.peek(System.out::println)` |
| `sorted()` | Natural sorting | `.sorted()` |
| `sorted(Comparator)` | Custom sorting | `.sorted(Comparator.reverseOrder())` |
| `distinct()` | Removes duplicates | `.distinct()` |
| `limit(n)` | Limits number of elements | `.limit(5)` |
| `skip(n)` | Skips first `n` elements | `.skip(3)` |

---

## 🏁 **3️⃣ Terminal Operations (End Stream)**
**Mnemonic:** _"ForEach → Collect → Reduce → Count → Min/Max → Match → Find"_

| **Method**  | **What it does?** | **Example** |
|------------|------------------|------------|
| `forEach(Consumer)` | Process each element | `.forEach(System.out::println)` |
| `collect(Collectors)` | Convert to collection | `.collect(Collectors.toList())` |
| `toArray()` | Convert to array | `.toArray(String[]::new)` |
| `reduce(BinaryOperator)` | Combines values | `.reduce(0, Integer::sum)` |
| `count()` | Counts elements | `.count()` |
| `min(Comparator)` | Finds minimum element | `.min(Integer::compareTo)` |
| `max(Comparator)` | Finds maximum element | `.max(Integer::compareTo)` |
| `anyMatch(Predicate)` | Check if **any** match | `.anyMatch(n -> n > 10)` |
| `allMatch(Predicate)` | Check if **all** match | `.allMatch(n -> n > 0)` |
| `noneMatch(Predicate)` | Check if **none** match | `.noneMatch(n -> n < 0)` |
| `findFirst()` | Gets first element | `.findFirst()` |
| `findAny()` | Gets any element | `.findAny()` |

---

## ⚡ **4️⃣ Specialized Streams (IntStream, LongStream, DoubleStream)**
**Mnemonic:** _"Range → RangeClosed → Boxed → Sum → Average → Map → Reduce"_

| **Method**  | **What it does?** | **Example** |
|------------|------------------|------------|
| `IntStream.range(start, end)` | Numbers from `start` to `end-1` | `IntStream.range(1, 5)` → `1,2,3,4` |
| `IntStream.rangeClosed(start, end)` | Numbers from `start` to `end` | `IntStream.rangeClosed(1, 5)` → `1,2,3,4,5` |
| `IntStream.of(...)` | Stream of numbers | `IntStream.of(1, 2, 3)` |
| `boxed()` | Convert to Stream<Integer> | `IntStream.range(1,5).boxed()` |
| `sum()` | Sum of elements | `IntStream.of(1,2,3).sum()` |
| `average()` | Average of elements | `IntStream.of(1,2,3).average()` |
| `mapToObj(Function)` | Convert to Object stream | `IntStream.range(1,5).mapToObj(Integer::toString)` |

---

## 🎯 **5️⃣ Parallel Streams**
**Mnemonic:** _"Sequential → Parallel → Unordered"_

| **Method**  | **What it does?** | **Example** |
|------------|------------------|------------|
| `parallel()` | Convert to parallel stream | `.parallel()` |
| `sequential()` | Convert back to sequential | `.sequential()` |
| `unordered()` | Removes ordering constraint | `.unordered()` |

---

# 🚀 **Super Simple Mnemonics for ALL Methods**
### ✅ **1. Getting a Stream** → _"List, Arrays, Of, Lines, Random, Builder, Iterate, Generate"_
   → `list.stream()`, `Arrays.stream()`, `Stream.of()`, `Files.lines()`, `Stream.generate()`

### ✅ **2. Intermediate Operations (Transform Data)** → _"Filter → Map → FlatMap → Peek → Sorted → Distinct → Limit → Skip"_
   → `filter()`, `map()`, `flatMap()`, `peek()`, `sorted()`, `distinct()`, `limit()`, `skip()`

### ✅ **3. Terminal Operations (Get Result)** → _"ForEach → Collect → Reduce → Count → Min/Max → Match → Find"_
   → `forEach()`, `collect()`, `reduce()`, `count()`, `min()`, `max()`, `anyMatch()`, `allMatch()`, `findFirst()`

### ✅ **4. Special Streams (Int, Long, Double)** → _"Range → RangeClosed → Boxed → Sum → Average → Map → Reduce"_
   → `range()`, `rangeClosed()`, `boxed()`, `sum()`, `average()`, `mapToObj()`, `reduce()`

### ✅ **5. Parallel Streams** → _"Sequential → Parallel → Unordered"_
   → `parallel()`, `sequential()`, `unordered()`

---

🔥 **Now you have a FULL cheat sheet with mnemonics to remember ALL Java 8 Stream API methods!** 🚀 Let me know if you want more examples! 😊





# 🎯 **Java 8 Functional Interfaces Cheat Sheet** 🚀

Java 8 introduced **functional interfaces**, which contain exactly **one abstract method** but can have **multiple default or static methods**. They are used primarily in **lambda expressions** and **method references**.

---

## 🏆 **1️⃣ Key Functional Interfaces in Java 8**  
👉 **Mnemonic to remember:**  
➡ **"Supplier, Consumer, Predicate, Function, Operator"**  
➡ **"SCaPe FO"** _(Supplier, Consumer, Predicate, Function, Operator)_

| **Interface** | **What it does?** | **Abstract Method** | **Example Usage** |
|--------------|------------------|---------------------|------------------|
| **Supplier<T>** | Returns a value **without input** | `T get()` | `Supplier<Double> random = Math::random;` |
| **Consumer<T>** | Takes an input but **returns nothing** | `void accept(T t)` | `Consumer<String> printer = System.out::println;` |
| **Predicate<T>** | Returns **boolean** based on input | `boolean test(T t)` | `Predicate<Integer> isEven = n -> n % 2 == 0;` |
| **Function<T, R>** | Takes one input and **returns a result** | `R apply(T t)` | `Function<String, Integer> length = String::length;` |
| **UnaryOperator<T>** | Like `Function<T,T>` (same input & output) | `T apply(T t)` | `UnaryOperator<Integer> square = n -> n * n;` |
| **BinaryOperator<T>** | Like `BiFunction<T,T,T>` (2 same-type inputs, 1 output) | `T apply(T t1, T t2)` | `BinaryOperator<Integer> sum = (a, b) -> a + b;` |
| **BiFunction<T, U, R>** | Takes 2 inputs, returns a result | `R apply(T t, U u)` | `BiFunction<Integer, Integer, String> concat = (a, b) -> a + ":" + b;` |
| **BiConsumer<T, U>** | Takes 2 inputs, returns nothing | `void accept(T t, U u)` | `BiConsumer<String, Integer> display = (s, n) -> System.out.println(s + " - " + n);` |
| **BiPredicate<T, U>** | Takes 2 inputs, returns boolean | `boolean test(T t, U u)` | `BiPredicate<Integer, String> check = (n, s) -> s.length() == n;` |

---

## 🚀 **2️⃣ Predefined Functional Interfaces (java.util.function)**  
Java provides these in the `java.util.function` package.

### ✅ **Supplier<T>** – _"Provides a value without input"_  
```java
Supplier<String> supplier = () -> "Hello, World!";
System.out.println(supplier.get()); // Output: Hello, World!
```

### ✅ **Consumer<T>** – _"Consumes an input but returns nothing"_  
```java
Consumer<String> consumer = s -> System.out.println("Hello " + s);
consumer.accept("Alice"); // Output: Hello Alice
```

### ✅ **Predicate<T>** – _"Tests a condition and returns boolean"_  
```java
Predicate<Integer> isEven = n -> n % 2 == 0;
System.out.println(isEven.test(4)); // Output: true
```

### ✅ **Function<T, R>** – _"Takes an input and returns a transformed value"_  
```java
Function<String, Integer> length = String::length;
System.out.println(length.apply("Java")); // Output: 4
```

### ✅ **UnaryOperator<T>** – _"Same input & output type transformation"_  
```java
UnaryOperator<Integer> square = n -> n * n;
System.out.println(square.apply(5)); // Output: 25
```

### ✅ **BinaryOperator<T>** – _"Two inputs of the same type, one output"_  
```java
BinaryOperator<Integer> sum = (a, b) -> a + b;
System.out.println(sum.apply(10, 20)); // Output: 30
```

---

## 🎯 **3️⃣ Default & Static Methods in Functional Interfaces**  
Functional interfaces can have **default and static methods**.

### 🔹 **Default Method Example (Consumer)**
```java
Consumer<String> print = s -> System.out.println(s);
print = print.andThen(s -> System.out.println("Length: " + s.length()));

print.accept("Java 8");  
// Output:
// Java 8
// Length: 6
```

### 🔹 **Static Method Example (Predicate)**
```java
Predicate<String> isNull = Objects::isNull;
Predicate<String> isNotNull = isNull.negate();

System.out.println(isNotNull.test("Hello")); // Output: true
```

---

## 🌟 **4️⃣ Custom Functional Interface**
You can create your own **functional interface** using `@FunctionalInterface`.

```java
@FunctionalInterface
interface MathOperation {
    int operation(int a, int b);
}

public class LambdaExample {
    public static void main(String[] args) {
        MathOperation add = (a, b) -> a + b;
        System.out.println(add.operation(5, 3)); // Output: 8
    }
}
```

---

## 🏆 **5️⃣ Method Reference (Shortcut for Lambdas)**
Instead of writing a lambda expression, you can use **method references**.

### 🔹 **Method Reference Example**
```java
Consumer<String> printer = System.out::println;
printer.accept("Hello, Java 8!"); // Output: Hello, Java 8!
```

### 🔹 **Types of Method References**
| **Type** | **Syntax** | **Example** |
|----------|-----------|------------|
| **Static Method** | `Class::staticMethod` | `Math::sqrt` |
| **Instance Method on Specific Object** | `object::instanceMethod` | `"Hello"::length` |
| **Instance Method on Arbitrary Object** | `Class::instanceMethod` | `String::toUpperCase` |
| **Constructor Reference** | `Class::new` | `ArrayList::new` |

---

## 🚀 **6️⃣ Summary (Easy Mnemonics)**
### **"SCaPe FO" → Supplier, Consumer, Predicate, Function, Operator**
- **Supplier** – Supplies a value (_no input_)
- **Consumer** – Consumes a value (_no output_)
- **Predicate** – Tests a condition (_returns boolean_)
- **Function** – Transforms a value (_T → R_)
- **Operator** – Special function where input and output are same type (_Unary & Binary_)

---

🔥 **Now you have a complete cheat sheet to remember Java 8 Functional Interfaces!** 🚀 Let me know if you need more examples. 😊
