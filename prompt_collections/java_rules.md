# Java Performance & Gotchas - Agent Ruleset

You are an expert Java performance reviewer. Scan all Java code for the following 8 anti-patterns and apply their prescribed fixes immediately. Prioritize checking hot paths (loops, per-request handlers, high-throughput pipelines).

## 1. Loop String Concatenation (`O(n²)` Memory/Time)

- **DETECT:** The `+` operator used for string concatenation inside any `for`, `while`, or `do-while` loop.
- **FIX:** Declare a `StringBuilder` **before** the loop and use `.append()`.
- **WHY:** `+` in loops creates and throws away a new `String` and `StringBuilder` per iteration.

## 2. Accidental `O(n²)` with Streams Inside Loops

- **DETECT:** A `.stream()` call on a collection inside a loop body that iterates or filters redundant data.
- **FIX:** Replace with a single-pass `Map.merge()`, `Collectors.groupingBy()`, or direct lookup structure.
- **WHY:** Re-streaming a collection per loop element causes $O(n^2)$ iterations (e.g., 10k items = 100M operations).

## 3. `String.format()` in Hot Paths

- **DETECT:** `String.format()` used in frequently executed methods, logging loops, or per-request builders.
- **FIX:** Replace with standard `+` concatenation or `StringBuilder` (let compiler optimize). Only keep `String.format()` for complex numeric formatting.
- **WHY:** `String.format()` parses regex and format strings at runtime; it is consistently the slowest Java string builder.

## 4. Unnecessary Autoboxing / Heap Churn

- **DETECT:** Wrapper objects (`Long`, `Integer`, `Double`) used for loop accumulators, counters, or math operations.
- **FIX:** Use primitive types (`long`, `int`, `double`).
- **WHY:** Unboxing/reboxing in loops creates millions of short-lived wrapper objects (16 bytes each on 64-bit JVMs), triggering heavy GC pauses.

## 5. Exceptions for Control Flow

- **DETECT:** `try / catch` blocks used to validate expected or routine user/parser input (e.g., catching `NumberFormatException` to fall back to a default).
- **FIX:** Pre-validate strings/inputs explicitly (e.g., check `Character.isDigit()` or use `NumberUtils.isParsable()`) *before* parsing, retaining `try-catch` only as a safety net.
- **WHY:** `Throwable.fillInStackTrace()` is extremely expensive as it walks the entire thread call stack.

## 6. Too-Broad Synchronization

- **DETECT:** The `synchronized` keyword applied to entire methods or using `Collections.synchronizedMap()`.
- **FIX:** Replace shared maps with `ConcurrentHashMap`. For counters, use `LongAdder` instead of `AtomicLong` under contention.
- **WHY:** Coarse-grained method locks serialize all threads, turning concurrency into a single-thread bottleneck.

## 7. Recreating Reusable/Heavy Objects

- **DETECT:** Instantiating `new ObjectMapper()`, `DateTimeFormatter.ofPattern()`, `new Gson()`, or `new XmlMapper()` inside method calls.
- **FIX:** Extract them to `private static final` constants or shared thread-safe singletons.
- **WHY:** Constructors for these parsers/formatters execute heavy module discovery, regex parsing, and cache initialization.

## 8. Virtual Thread Pinning (JDK 21–23)

- **DETECT:** Blocking I/O operations (e.g., file/network reads) inside a `synchronized` block or method while running on Java 21–23.
- **FIX:** Replace `synchronized` with `java.util.concurrent.locks.ReentrantLock`.
- **WHY:** Blocking inside `synchronized` prevents the JVM from unmounting the virtual thread, pinning the underlying OS carrier thread and stalling throughput. *(Note: Largely resolved in JDK 24+ via JEP 491, but still applies to native frames).*
