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
