# Java Fundamentals — Senior Developer Interview Study Notes

---

## 1. Topic Overview

Java is a **statically typed, object-oriented, platform-independent** language designed around the principle of *Write Once, Run Anywhere (WORA)*. It underpins enterprise backends (Spring), Android, distributed systems (Kafka, Hadoop), and financial platforms.

**Why it matters for senior interviews:**
- Deep JVM knowledge separates senior from mid-level engineers
- Memory model, type system, and language semantics have subtle gotchas that appear in production bugs
- Foundational to understanding frameworks like Spring, Hibernate, and reactive systems

---

## 2. Java Platform Overview

### The Three Pillars

```
┌─────────────────────────────────────┐
│              JDK                    │
│  ┌───────────────────────────────┐  │
│  │            JRE                │  │
│  │  ┌─────────────────────────┐  │  │
│  │  │          JVM            │  │  │
│  │  └─────────────────────────┘  │  │
│  │  + Java Class Libraries       │  │
│  └───────────────────────────────┘  │
│  + Compiler (javac), javadoc, etc.  │
└─────────────────────────────────────┘
```

| Component | Full Name | Purpose |
|-----------|-----------|---------|
| **JVM** | Java Virtual Machine | Executes bytecode; platform-specific |
| **JRE** | Java Runtime Environment | JVM + standard class libraries; run Java apps |
| **JDK** | Java Development Kit | JRE + compiler + dev tools; build Java apps |

**Key distinction:** You need the JDK to *develop*, the JRE to *run*, the JVM is the actual engine doing the work. Since Java 11, the standalone JRE is no longer distributed separately — JDK is the standard distribution.

---

## 3. Java Compilation Process & Bytecode

### Compilation Pipeline

```
Source Code (.java)
        │
        ▼
   javac (compiler)
        │
        ▼
  Bytecode (.class)   ◄── Platform-independent intermediate representation
        │
        ▼
  Class Loader (JVM)
        │
        ▼
  Bytecode Verifier
        │
        ▼
  JIT Compiler / Interpreter
        │
        ▼
  Native Machine Code  ◄── Platform-specific
```

### Key Points
- **Bytecode** is not machine code — it's an intermediate format for the JVM
- The **JIT (Just-In-Time) compiler** identifies "hot" code paths and compiles them to native code at runtime for performance
- **AOT (Ahead-of-Time)** compilation (GraalVM) compiles to native binary at build time — faster startup, lower memory
- `javap -c MyClass.class` disassembles bytecode — useful for debugging and interview discussions

```bash
# Compile
javac HelloWorld.java

# Run
java HelloWorld

# Inspect bytecode
javap -c HelloWorld.class
```

---

## 4. Java Program Structure

```java
// Package declaration (optional but best practice)
package com.example.demo;

// Imports
import java.util.List;
import java.util.ArrayList;

// Class declaration
public class HelloWorld {             // filename must match public class name

    // Static variables (class-level)
    private static final String GREETING = "Hello";

    // Instance variables
    private String name;

    // Constructor
    public HelloWorld(String name) {
        this.name = name;
    }

    // Main method — JVM entry point
    public static void main(String[] args) {
        HelloWorld hw = new HelloWorld("World");
        System.out.println(hw.greet());
    }

    // Instance method
    public String greet() {
        return GREETING + ", " + name + "!";
    }
}
```

**Rules:**
- One public class per `.java` file; filename must match
- `main` signature: `public static void main(String[] args)` — exact signature required
- Execution order: static initializers → instance initializers → constructor

---

## 5. Data Types

### Primitive Types (8 total)

| Type | Size | Default | Range |
|------|------|---------|-------|
| `byte` | 8-bit | 0 | -128 to 127 |
| `short` | 16-bit | 0 | -32,768 to 32,767 |
| `int` | 32-bit | 0 | ~±2.1 billion |
| `long` | 64-bit | 0L | ~±9.2 × 10^18 |
| `float` | 32-bit | 0.0f | ~6-7 decimal digits |
| `double` | 64-bit | 0.0d | ~15 decimal digits |
| `char` | 16-bit | '\u0000' | 0 to 65,535 (Unicode) |
| `boolean` | ~1-bit* | false | true / false |

> *`boolean` size is JVM-implementation-dependent; often stored as 4 bytes in practice*

### Primitive vs Reference Types

| Aspect | Primitive | Reference |
|--------|-----------|-----------|
| Storage | Stack (value directly) | Stack holds reference → Heap holds object |
| Default | Type-specific (0, false) | `null` |
| Comparison | `==` compares value | `==` compares reference address |
| Nullability | Cannot be null | Can be null |
| Performance | Faster (no heap allocation) | Slower (GC overhead) |

```java
int a = 5;
int b = 5;
System.out.println(a == b);    // true — value comparison

String s1 = new String("hello");
String s2 = new String("hello");
System.out.println(s1 == s2);      // false — different references
System.out.println(s1.equals(s2)); // true — content comparison
```

---

## 6. Variables and Scope

### Variable Types

```java
public class ScopeDemo {

    static int classVar = 10;        // Class/static variable — shared across instances
    int instanceVar = 20;            // Instance variable — per object, lives on heap

    public void method() {
        int localVar = 30;           // Local variable — stack, must be initialized before use

        for (int i = 0; i < 3; i++) {
            int blockVar = i * 2;    // Block variable — only exists inside loop
        }
        // blockVar not accessible here
    }
}
```

### Scope Rules
- **Local variables** have no default values — compiler error if used uninitialized
- **Instance/static** variables get default values (0, null, false)
- `this` refers to current instance — used to disambiguate instance vs local
- `static` variables belong to the class, not any instance

---

## 7. Operators

### Key Operators and Gotchas

```java
// Integer division truncates
int result = 7 / 2;          // 3, NOT 3.5
double result2 = 7.0 / 2;    // 3.5

// Modulo with negatives
System.out.println(-7 % 3);   // -1 (sign follows dividend in Java)

// Bitwise operators
int x = 5 & 3;   // AND: 0101 & 0011 = 0001 = 1
int y = 5 | 3;   // OR:  0101 | 0011 = 0111 = 7
int z = 5 ^ 3;   // XOR: 0101 ^ 0011 = 0110 = 6
int s = 5 << 1;  // Left shift = 10 (multiply by 2)
int r = 5 >> 1;  // Right shift = 2  (divide by 2, sign-preserving)
int u = -1 >>> 1; // Unsigned right shift — fills with 0s

// Short-circuit evaluation
boolean val = (x != null) && x.isValid(); // safe — stops if x is null
boolean val2 = (a != 0) & (b / a > 1);   // NOT safe — evaluates both sides
```

### Pre vs Post Increment

```java
int a = 5;
int b = a++;   // b = 5, a = 6 (post: use then increment)
int c = ++a;   // c = 7, a = 7 (pre: increment then use)
```

---

## 8. Control Flow Statements

```java
// Switch expression (Java 14+ — preferred modern form)
String day = "MONDAY";
String type = switch (day) {
    case "MONDAY", "TUESDAY", "WEDNESDAY", "THURSDAY", "FRIDAY" -> "Weekday";
    case "SATURDAY", "SUNDAY" -> "Weekend";
    default -> throw new IllegalArgumentException("Unknown: " + day);
};

// Traditional switch — fall-through pitfall
switch (day) {
    case "MONDAY":
        System.out.println("Start of week");
        break;           // Without break, falls through to next case!
    case "FRIDAY":
        System.out.println("End of week");
        break;
    default:
        System.out.println("Other");
}

// Ternary
int max = (a > b) ? a : b;
```

**Senior note:** Switch expressions (Java 14+) are exhaustive and prevent fall-through bugs. Prefer them. Know the difference between `switch` statement (can fall through) vs `switch` expression (must be exhaustive, no fall-through by default).

---

## 9. Loops

```java
// Standard for loop
for (int i = 0; i < 10; i++) { }

// Enhanced for (for-each) — read only, no index access
int[] arr = {1, 2, 3};
for (int n : arr) {
    System.out.println(n);
}

// While vs Do-While
int i = 0;
while (i < 5) { i++; }        // may not execute at all

int j = 0;
do { j++; } while (j < 5);    // always executes at least once

// Labeled break — exit outer loop
outer:
for (int x = 0; x < 3; x++) {
    for (int y = 0; y < 3; y++) {
        if (x == 1 && y == 1) break outer; // exits both loops
    }
}

// continue — skip current iteration
for (int n = 0; n < 10; n++) {
    if (n % 2 == 0) continue;
    System.out.print(n + " ");  // prints 1 3 5 7 9
}
```

---

## 10. Arrays

```java
// Declaration and initialization
int[] arr1 = new int[5];              // Default values: 0
int[] arr2 = {1, 2, 3, 4, 5};        // Array literal
int[] arr3 = new int[]{1, 2, 3};     // Explicit new with literal

// Multi-dimensional (arrays of arrays — jagged is possible!)
int[][] matrix = new int[3][4];
int[][] jagged = new int[3][];        // rows have different lengths
jagged[0] = new int[2];
jagged[1] = new int[5];

// Key properties
System.out.println(arr2.length);      // 5 — field, not method!
// Arrays are fixed-size — use ArrayList for dynamic sizing

// Useful utilities
import java.util.Arrays;
Arrays.sort(arr2);                    // O(n log n), modifies in-place
int idx = Arrays.binarySearch(arr2, 3);  // requires sorted array
int[] copy = Arrays.copyOf(arr2, 3);     // [1, 2, 3]
System.out.println(Arrays.toString(arr2)); // "[1, 2, 3, 4, 5]"

// Common pitfall: array covariance
Object[] objs = new String[3];
objs[0] = 42;  // Compiles! But throws ArrayStoreException at runtime
```

**Key Distinction:** `array.length` is a **field** (no parentheses). `String.length()` is a **method** (with parentheses). `Collection.size()` is also a method.

---

## 11. Strings and Immutability

### String Immutability

```java
String s = "hello";
s.toUpperCase();              // Creates NEW string — original unchanged!
System.out.println(s);       // Still "hello"

String upper = s.toUpperCase();
System.out.println(upper);   // "HELLO"
```

**Why immutable?**
- Thread safety — no synchronization needed
- Security — cannot modify class names, file paths after validation
- Caching — hashCode can be cached (computed once)
- String pool efficiency

### String Pool

```java
String s1 = "hello";           // Goes into String Pool (heap — PermGen/Metaspace)
String s2 = "hello";           // Reuses same pool object
String s3 = new String("hello"); // Forces new heap object — bypasses pool

System.out.println(s1 == s2);   // true  — same pool reference
System.out.println(s1 == s3);   // false — different heap objects
System.out.println(s1.equals(s3)); // true — same content

// intern() — manually add to / retrieve from pool
String s4 = s3.intern();
System.out.println(s1 == s4);   // true — s4 now points to pool
```

### String vs StringBuilder vs StringBuffer

| | String | StringBuilder | StringBuffer |
|---|---|---|---|
| Mutable | No | Yes | Yes |
| Thread-safe | Yes (immutable) | No | Yes (synchronized) |
| Performance | Slow for concatenation | Fast | Slower than StringBuilder |
| Use case | Fixed strings | Single-thread mutation | Multi-thread (rare) |

```java
// BAD — creates N intermediate String objects
String result = "";
for (int i = 0; i < 1000; i++) {
    result += i;  // O(n²) complexity
}

// GOOD — O(n) complexity
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000; i++) {
    sb.append(i);
}
String result = sb.toString();
```

### Key String Methods

```java
String s = "Hello, World!";
s.length()                     // 13
s.charAt(0)                    // 'H'
s.substring(7)                 // "World!"
s.substring(7, 12)             // "World"
s.indexOf("World")             // 7
s.contains("World")            // true
s.startsWith("Hello")          // true
s.toLowerCase()                // "hello, world!"
s.trim()                       // removes leading/trailing whitespace
s.strip()                      // Java 11+, Unicode-aware trim
s.replace("World", "Java")     // "Hello, Java!"
s.split(", ")                  // ["Hello", "World!"]
s.isEmpty()                    // false
s.isBlank()                    // false (Java 11+, checks whitespace too)
String.format("Hi %s", "Bob")  // "Hi Bob"
String.join(", ", "a","b","c") // "a, b, c"
```

---

## 12. Wrapper Classes

Each primitive has a corresponding wrapper class in `java.lang`:

| Primitive | Wrapper |
|-----------|---------|
| `int` | `Integer` |
| `long` | `Long` |
| `double` | `Double` |
| `float` | `Float` |
| `boolean` | `Boolean` |
| `char` | `Character` |
| `byte` | `Byte` |
| `short` | `Short` |

```java
// Parsing
int i = Integer.parseInt("42");
double d = Double.parseDouble("3.14");

// Conversion
String s = Integer.toString(42);
String s2 = String.valueOf(42);      // preferred — handles null

// Useful constants
Integer.MAX_VALUE    // 2147483647
Integer.MIN_VALUE    // -2147483648
Integer.toBinaryString(10)  // "1010"
Integer.toHexString(255)    // "ff"

// Comparison — use compareTo, never == for wrappers!
Integer a = 1000;
Integer b = 1000;
System.out.println(a == b);       // false! Different objects (beyond cache range)
System.out.println(a.equals(b));  // true
System.out.println(Integer.compare(a, b)); // 0
```

---

## 13. Autoboxing and Unboxing

```java
// Autoboxing — primitive → wrapper (done by compiler)
Integer x = 42;           // Compiler: Integer x = Integer.valueOf(42);

// Unboxing — wrapper → primitive
int y = x;                // Compiler: int y = x.intValue();

// Integer Cache: -128 to 127 are cached
Integer a = 127;
Integer b = 127;
System.out.println(a == b);   // true  — cached, same object

Integer c = 128;
Integer d = 128;
System.out.println(c == d);   // false — beyond cache, new objects

// NullPointerException trap
Integer val = null;
int primitive = val;   // NullPointerException at unboxing!

// Performance trap in loops
Long sum = 0L;
for (long i = 0; i < 1_000_000; i++) {
    sum += i;   // Unbox sum, add i, autobox result — 1M object creations!
}
// Fix: use primitive long sum = 0L;
```

---

## 14. Type Casting

### Widening vs Narrowing

```java
// Widening (implicit — no data loss)
// byte → short → int → long → float → double
int i = 100;
long l = i;       // implicit widening
double d = i;     // implicit widening

// Narrowing (explicit cast — potential data loss)
double pi = 3.14159;
int truncated = (int) pi;    // 3 — decimal truncated, not rounded
long big = 1_000_000_000_000L;
int overflow = (int) big;    // data loss — undefined-looking result

// char and numeric interplay
char c = 'A';
int ascii = c;               // widening: 65
char back = (char) 66;       // narrowing: 'B'

// Object casting
Object obj = "Hello";
String s = (String) obj;              // OK — actual type is String
Integer n = (Integer) obj;            // ClassCastException at runtime!

// Safe casting with instanceof
if (obj instanceof String str) {      // Java 16+ pattern matching
    System.out.println(str.length()); // no explicit cast needed
}
```

### Numeric Promotion Rules
When mixing types in expressions, smaller types are promoted:
- `byte + byte` → `int` (not `byte`!)
- `int + long` → `long`
- `int + float` → `float`
- `long + double` → `double`

```java
byte a = 10, b = 20;
byte c = a + b;    // COMPILE ERROR — a+b is int!
byte d = (byte)(a + b);  // requires explicit cast
```

---

## 15. Pass by Value

**Java is ALWAYS pass-by-value. Always.**

```java
// Primitive — copy of value passed
public static void increment(int x) {
    x++;               // modifies local copy only
}
int a = 5;
increment(a);
System.out.println(a); // Still 5

// Reference type — copy of REFERENCE passed
public static void addElement(List<String> list) {
    list.add("new");   // modifies the OBJECT the reference points to
}
List<String> myList = new ArrayList<>();
addElement(myList);
System.out.println(myList.size()); // 1 — object was modified

// But reassigning the reference doesn't affect caller
public static void reassign(List<String> list) {
    list = new ArrayList<>();  // only local reference changes
    list.add("ignored");
}
reassign(myList);
System.out.println(myList.size()); // Still 1 — caller's reference unchanged
```

**Mental model:** Think of it as "pass by copy of the pointer." The object on the heap is shared, but you can't make the caller's variable point to something else.

---

## 16. Java Keywords (Essential Ones)

```java
// Access modifiers
public      // accessible everywhere
protected   // accessible in same package + subclasses
private     // accessible only in same class
// (default) // accessible in same package only (no keyword)

// Class-related
class, interface, enum, record     // type declarations
extends     // class inheritance (single)
implements  // interface implementation (multiple)
abstract    // class cannot be instantiated / method has no body
final       // class: no subclassing; method: no override; var: no reassign
static      // belongs to class, not instance
new         // creates a new object on the heap

// Type/value
null        // absence of reference (NOT a type)
void        // no return value
this        // reference to current instance
super       // reference to parent class
instanceof  // type check (use with pattern matching in Java 16+)

// Control
if, else, switch, case, default
for, while, do, break, continue, return
throw, throws, try, catch, finally

// Memory/synchronization
synchronized  // acquire monitor lock on object/method
volatile      // read/write directly to main memory (visibility guarantee)
transient     // exclude field from serialization

// Rarely used but asked about
native        // method implemented in native code (C/C++)
strictfp      // strict floating-point (deprecated Java 17)
assert        // debug assertions (disabled by default at runtime)
goto, const   // reserved but UNUSED in Java
```

---

## 17. Interview Questions with Expert Answers

**Q1: What is the difference between JVM, JRE, and JDK?**
> JVM is the runtime engine that executes bytecode — it's platform-specific. JRE bundles the JVM with the standard class libraries needed to run Java applications. JDK includes the JRE plus development tools (compiler `javac`, `javadoc`, `jshell`, etc.) needed to build Java programs. Since Java 11, Oracle no longer distributes a standalone JRE — developers use the full JDK.

---

**Q2: Why is Java "platform-independent" but the JVM is platform-dependent?**
> Java source compiles to bytecode — a platform-neutral intermediate format. The JVM is a spec that different vendors implement for each OS/architecture (Windows x64, Linux ARM, etc.). Each JVM implementation knows how to translate bytecode to that platform's native instructions. So the Java code is portable; the JVM itself is not.

---

**Q3: What is the String pool and why does it exist?**
> The String pool (interned string table) is a special region in heap memory (Metaspace in Java 8+) where the JVM stores unique string literals. When you write `String s = "hello"`, the JVM checks the pool first. If "hello" exists, it reuses the reference; otherwise it creates a new entry. This saves memory since strings are immutable and widely reused. `new String("hello")` deliberately bypasses the pool.

---

**Q4: Explain autoboxing cache. Why does `Integer a = 127; Integer b = 127; a == b` return true but `Integer a = 128; Integer b = 128; a == b` return false?**
> `Integer.valueOf()` (used during autoboxing) maintains a cache of `Integer` objects for values -128 to 127 (per JLS spec). Values in this range return the same cached object, so `==` compares the same reference. Outside this range, `valueOf()` creates a new heap object each time, so `==` compares two different references. **Always use `.equals()` to compare wrapper types** — never `==`.

---

**Q5: Java is pass-by-value or pass-by-reference?**
> Always pass-by-value. For primitives, a copy of the value is passed. For objects, a copy of the *reference* (memory address) is passed. This means the method can mutate the object the reference points to, but cannot make the caller's variable point to a different object. Many people confuse this with pass-by-reference, but true pass-by-reference (as in C++) would allow changing the caller's variable itself.

---

**Q6: Why are Strings immutable in Java?**
> Four reasons: (1) **Security** — class names, file paths, network URLs are Strings; mutability would allow code to change them after security validation. (2) **Thread safety** — immutable objects are inherently thread-safe. (3) **String pool** — pooling requires objects to never change; a shared mutable string would be catastrophic. (4) **HashCode caching** — String's hashCode is computed once and cached, which is safe only if the string never changes.

---

**Q7: What's the difference between `String`, `StringBuilder`, and `StringBuffer`?**
> `String` is immutable — every "modification" creates a new object. `StringBuilder` is mutable and not thread-safe but fast — use it for single-threaded string manipulation. `StringBuffer` is mutable and thread-safe (all methods synchronized) but slower. In practice, `StringBuffer` is rarely needed — prefer `StringBuilder` in single-threaded code and higher-level coordination (`ConcurrentLinkedQueue` of messages, etc.) in multi-threaded contexts.

---

**Q8: What happens when you concatenate strings using `+` in a loop?**
> Each `+=` creates a new `String` object (since String is immutable), copies the old content, and appends. For N iterations, you create N intermediate objects and copy O(1+2+...+N) = O(n²) characters total. The JVM's JIT may optimize some cases (Java 9+ uses `StringConcatFactory` with `invokedynamic`), but for explicit loops with `+=`, it's still O(n²). Use `StringBuilder.append()` for O(n) performance.

---

**Q9: What is widening vs narrowing conversion? When do you need an explicit cast?**
> Widening converts to a larger type (e.g., `int` → `long`) — done implicitly, no data loss. Narrowing converts to a smaller type (e.g., `double` → `int`) — requires explicit cast, risks data loss (decimal truncation, integer overflow). Noteworthy: `byte + byte` produces `int` due to numeric promotion, so you need `(byte)(a + b)` to store it back in a byte.

---

**Q10: What is the difference between `==` and `.equals()`?**
> `==` compares references for objects (are they the same object in memory?) and values for primitives. `.equals()` compares logical equality — content. For `String`, `Integer`, and most well-designed classes, `.equals()` compares content. For custom classes, you should override `equals()` (and always `hashCode()` together with it). Common trap: `Integer a = 1000; Integer b = 1000; a == b` is false even though they're "equal."

---

**Q11: Explain the `final` keyword in different contexts.**
> (1) **`final` variable** — value cannot be reassigned after initialization (but object's state can still mutate). (2) **`final` method** — cannot be overridden in subclasses. (3) **`final` class** — cannot be subclassed (e.g., `String`, `Integer`). `final` ≠ immutable for objects — `final List<String> list` cannot be reassigned, but `list.add("x")` still works.

---

**Q12: What is `static` and when should you use it?**
> `static` makes a member belong to the class rather than any instance. Static variables are shared across all instances — one copy regardless of how many objects. Static methods can be called without an instance but cannot access instance variables (`this` doesn't exist). Use for utility methods (`Math.abs()`), constants (`static final`), factory methods, and the singleton pattern. Pitfall: over-using static introduces hidden global state, making code hard to test and reason about.

---

**Q13: What is the difference between `null`, an empty String, and a blank String?**
> `null` means the reference points to no object — operations on it throw `NullPointerException`. `""` (empty) is a valid String object with length 0. `" "` (blank) is a String with only whitespace. `isEmpty()` returns true for `""`. `isBlank()` (Java 11+) returns true for `""` and `"   "`. Senior tip: always null-check before calling string methods; use `Objects.isNull()` or `Optional` for cleaner null handling.

---

**Q14: How does the JIT compiler work?**
> The JVM initially interprets bytecode. The JIT compiler monitors execution and identifies "hot" code (methods called frequently, loops iterated many times). It then compiles those hot paths to optimized native machine code and caches it. Subsequent calls use the native code directly. HotSpot uses two JIT compilers: C1 (fast, light optimization — client) and C2 (heavy optimization — server). `-XX:+PrintCompilation` shows what's being JIT-compiled.

---

**Q15: What are `volatile` and `transient`?**
> `volatile` ensures that reads and writes to a variable go directly to main memory — guarantees visibility across threads but NOT atomicity (e.g., `volatile int++` is still not atomic). Use for flags, not counters. `transient` marks a field to be excluded from Java serialization — the field won't be written to the ObjectOutputStream and will be initialized to its default value when deserialized. Common use: password fields, cached computed values.

---

## 18. Advanced / Senior-Level Considerations

### JVM Internals
- **Class loading:** Bootstrap → Extension → Application classloader (delegation model)
- **Memory areas:** Heap (objects), Stack (frames/locals), Metaspace (class metadata), PC Register, Native Method Stack
- **GC relevance:** Primitives on stack don't need GC; objects on heap do. String pool is on heap (Metaspace area)

### Performance Traps to Know
- Autoboxing in hot loops — use primitive arrays/streams
- `String +` in loops — use `StringBuilder`
- `new Integer(x)` — deprecated; use `Integer.valueOf(x)` for cache benefits
- `StringBuffer` instead of `StringBuilder` in single-threaded code — unnecessary locking

### Modern Java (Java 16-21) Additions
- **Records** — immutable data carriers, auto-generates constructor/equals/hashCode
- **Pattern matching instanceof** — `if (obj instanceof String s)` — no explicit cast
- **Switch expressions** — exhaustive, expression form, arrow syntax
- **Text blocks** — multiline string literals with `"""`
- **Sealed classes** — restrict which classes can extend/implement

```java
// Record (Java 16+)
record Point(int x, int y) { }    // immutable, auto-equals/hashCode/toString

// Text block (Java 15+)
String json = """
    {
        "name": "Java",
        "version": 21
    }
    """;

// Pattern matching (Java 16+)
if (shape instanceof Circle c) {
    return Math.PI * c.radius() * c.radius();
}
```

### Type System Design Decisions
- Choose `int` over `Integer` unless nullability or collection storage requires it
- Use `long` for IDs, timestamps — `int` overflows in ~2 billion
- Use `BigDecimal` for financial calculations — `double` has floating-point precision errors
- `char` should not be used for text processing in modern Java — use `String`; Unicode beyond BMP requires codepoints

---

## 19. Summary / Cheat Sheet

```
JDK ⊃ JRE ⊃ JVM
.java → javac → .class (bytecode) → JVM → JIT → native code

Primitives: byte, short, int, long, float, double, char, boolean
Everything else: reference type (lives on heap, can be null)

String pool: "literal" → pool; new String() → heap bypass
String is immutable: always use StringBuilder for mutation in loops

Autoboxing cache: Integer -128 to 127 → same object; use .equals() always
Unboxing null → NullPointerException

Pass-by-value ALWAYS:
  - primitive: copy of value
  - reference: copy of reference (object still shared)

Widening: implicit (no data loss)  byte → short → int → long → float → double
Narrowing: explicit cast required (data loss possible)
byte + byte = int (numeric promotion!)

final: variable = no reassign, method = no override, class = no subclass
static: belongs to class, not instance
volatile: visibility guarantee (not atomicity)
transient: exclude from serialization

== for primitives: value comparison
== for references: same object? Use .equals() for content comparison
```

**Key senior differentiators:**
- Know *why* String is immutable, not just *that* it is
- Understand Integer cache bounds and consequences
- Explain pass-by-value with a reference type example convincingly
- Know when `StringBuilder` vs `StringBuffer` matters (and when it doesn't)
- Understand autoboxing performance implications in hot paths
- Connect JIT compilation to why Java "warms up" and gets faster over time


# Object-Oriented Programming in Java — Senior Developer Interview Study Notes

---

## 1. Topic Overview

OOP is a programming paradigm that models software around **objects** — entities that bundle state (fields) and behavior (methods). Java is built on OOP from the ground up; every non-primitive value is an object.

**Why it matters at the senior level:**
- Misused OOP (deep inheritance, broken encapsulation, violated contracts) is the root cause of unmaintainable enterprise codebases
- Senior engineers must know *when not to use* inheritance, *when composition beats inheritance*, and *how broken `equals`/`hashCode`* causes silent bugs in collections
- Framework internals (Spring, Hibernate, JPA) rely heavily on OOP contracts — understanding them explains framework behavior

---

## 2. The Four OOP Principles

```
┌─────────────────────────────────────────────────────┐
│                  OOP PILLARS                        │
│                                                     │
│  Encapsulation  │  Abstraction                      │
│  ─────────────  │  ───────────                      │
│  Hide state     │  Hide complexity                  │
│  Expose API     │  Expose interface                 │
│                 │                                   │
│  Inheritance    │  Polymorphism                     │
│  ────────────   │  ─────────────                    │
│  Reuse behavior │  Many forms, one interface        │
│  IS-A relation  │  Runtime/Compile-time dispatch    │
└─────────────────────────────────────────────────────┘
```

---

## 3. Encapsulation

**Definition:** Bundle data and the methods that operate on it into a single unit (class), and restrict direct access to the data.

```java
public class BankAccount {

    // Private state — no direct access from outside
    private String accountNumber;
    private double balance;
    private List<String> transactionHistory;

    public BankAccount(String accountNumber, double initialBalance) {
        if (initialBalance < 0) throw new IllegalArgumentException("Balance cannot be negative");
        this.accountNumber = accountNumber;
        this.balance = initialBalance;
        this.transactionHistory = new ArrayList<>();
    }

    // Controlled access — validation and invariants enforced
    public void deposit(double amount) {
        if (amount <= 0) throw new IllegalArgumentException("Deposit must be positive");
        balance += amount;
        transactionHistory.add("Deposit: " + amount);
    }

    public void withdraw(double amount) {
        if (amount <= 0) throw new IllegalArgumentException("Must be positive");
        if (amount > balance) throw new IllegalStateException("Insufficient funds");
        balance -= amount;
        transactionHistory.add("Withdrawal: " + amount);
    }

    // Read-only access — defensive copy to protect internal list
    public double getBalance() { return balance; }
    public List<String> getTransactionHistory() {
        return Collections.unmodifiableList(transactionHistory); // NOT a direct reference!
    }
}
```

### Key Details
- `private` fields + `public` getters/setters is the minimum — but don't blindly generate setters for everything
- **Defensive copying:** returning `new ArrayList<>(internalList)` or `Collections.unmodifiableList()` prevents external mutation of internal state
- **Invariant protection:** constructors and setters should validate and enforce class invariants (balance ≥ 0)
- **Tell, don't ask:** instead of `if (account.getBalance() > 0) account.deduct(x)`, expose `account.withdraw(x)` — keep logic inside the class

### Common Pitfalls
- Generating getters/setters for every field — this is just syntactic sugar, not real encapsulation
- Returning mutable internal collections directly — callers can silently corrupt state
- `public` fields for "simplicity" — breaks all future ability to add validation

---

## 4. Inheritance

**Definition:** A class (subclass) acquires the properties and behaviors of another class (superclass). Models an **IS-A** relationship.

```java
// Superclass
public abstract class Shape {
    private String color;

    public Shape(String color) {
        this.color = color;
    }

    public String getColor() { return color; }

    // Contract: every Shape must know its area
    public abstract double area();

    // Concrete behavior shared by all shapes
    public String describe() {
        return String.format("%s shape with area %.2f", color, area());
    }
}

// Subclass
public class Circle extends Shape {
    private double radius;

    public Circle(String color, double radius) {
        super(color);          // Must call superclass constructor first
        this.radius = radius;
    }

    @Override
    public double area() {
        return Math.PI * radius * radius;
    }
}

public class Rectangle extends Shape {
    private double width, height;

    public Rectangle(String color, double width, double height) {
        super(color);
        this.width = width;
        this.height = height;
    }

    @Override
    public double area() {
        return width * height;
    }
}
```

### Inheritance Chain
```
Object
  └── Shape
        ├── Circle
        ├── Rectangle
        └── Triangle
```

### Key Details
- Java supports **single inheritance** for classes — a class can only `extend` one class
- Java supports **multiple inheritance of type** via interfaces — a class can `implement` multiple interfaces
- All classes implicitly extend `java.lang.Object`
- `super()` must be the first statement in a constructor if calling parent constructor explicitly
- If no `super()` call, compiler inserts `super()` (no-arg) automatically — compile error if parent has no no-arg constructor

### When NOT to Use Inheritance
- When the relationship is HAS-A, not IS-A (use composition)
- When you need to inherit from multiple classes (use interfaces + composition)
- When the subclass needs to override most of the parent's behavior (Liskov violation signal)
- When parent and child are in different libraries/teams — tight coupling across boundaries

### Common Pitfalls
- **Fragile base class problem:** changing superclass breaks subclasses unexpectedly
- **Deep inheritance hierarchies:** 5+ levels deep becomes unmaintainable
- Calling overridable methods from constructors — subclass override runs before subclass constructor completes

```java
public class Parent {
    public Parent() {
        init();   // DANGEROUS — calls overridden method in subclass
    }
    public void init() { System.out.println("Parent init"); }
}

public class Child extends Parent {
    private String name = "child";
    @Override
    public void init() {
        System.out.println(name.toUpperCase()); // NullPointerException! name not yet set
    }
}
```

---

## 5. Polymorphism

**Definition:** The ability of a single interface to represent different underlying forms. "Many shapes, one interface."

### Two Types

**1. Compile-time Polymorphism (Static Dispatch) — Method Overloading**

```java
public class Printer {
    public void print(int value)    { System.out.println("int: " + value); }
    public void print(String value) { System.out.println("String: " + value); }
    public void print(double value) { System.out.println("double: " + value); }

    // Resolved at compile time based on parameter types
}

Printer p = new Printer();
p.print(42);      // "int: 42"
p.print("hello"); // "String: hello"
```

**2. Runtime Polymorphism (Dynamic Dispatch) — Method Overriding**

```java
Shape s1 = new Circle("red", 5.0);
Shape s2 = new Rectangle("blue", 4.0, 6.0);

List<Shape> shapes = List.of(s1, s2);
for (Shape s : shapes) {
    System.out.println(s.area());  // Correct area() called based on ACTUAL type at runtime
}
// 78.539...
// 24.0
```

### How Dynamic Dispatch Works (vtable)
```
At runtime, each object carries a reference to its class's virtual method table (vtable).
When s.area() is called on a Shape reference:
  1. JVM looks up actual type (Circle, Rectangle)
  2. Finds that type's vtable
  3. Calls the correct area() implementation
This happens at runtime — NOT at compile time
```

### Covariant Return Types
```java
public class Animal {
    public Animal create() { return new Animal(); }
}
public class Dog extends Animal {
    @Override
    public Dog create() { return new Dog(); }  // Covariant: return type can be subtype
}
```

---

## 6. Abstraction

**Definition:** Expose *what* something does, hide *how* it does it. Reduce complexity by modeling only relevant attributes.

### Abstract Classes vs Interfaces

```java
// Abstract class — partial implementation, shared state
public abstract class Vehicle {
    protected String brand;     // shared state
    protected int year;

    public Vehicle(String brand, int year) {
        this.brand = brand;
        this.year = year;
    }

    // Concrete method — shared behavior
    public String getInfo() { return brand + " (" + year + ")"; }

    // Abstract method — must be implemented by subclasses
    public abstract void startEngine();

    // Template Method pattern — defines algorithm skeleton
    public final void start() {
        checkFuel();       // concrete
        startEngine();     // abstract — customized by subclass
        engage();          // concrete
    }

    private void checkFuel() { System.out.println("Checking fuel..."); }
    private void engage()    { System.out.println("Vehicle engaged."); }
}

// Interface — pure contract, no state (except constants)
public interface Chargeable {
    int getChargeLevel();                  // abstract by default
    default boolean isFullyCharged() {    // default method (Java 8+)
        return getChargeLevel() == 100;
    }
    static Chargeable noop() {            // static method (Java 8+)
        return () -> 100;
    }
}

// Class can extend one abstract class AND implement multiple interfaces
public class ElectricCar extends Vehicle implements Chargeable, Serializable {
    private int chargeLevel;

    public ElectricCar(String brand, int year, int chargeLevel) {
        super(brand, year);
        this.chargeLevel = chargeLevel;
    }

    @Override public void startEngine() { System.out.println("Silent electric start"); }
    @Override public int getChargeLevel() { return chargeLevel; }
}
```

### Abstract Class vs Interface Decision Table

| Aspect | Abstract Class | Interface |
|--------|----------------|-----------|
| State | Yes (fields) | No (constants only) |
| Constructor | Yes | No |
| Access modifiers | All | public (default) |
| Multiple inheritance | No | Yes |
| When to use | Shared base with state | Contract / capability |
| Java 8+ additions | — | default + static methods |
| Java 9+ additions | — | private methods |

---

## 7. Method Overloading

**Rules for valid overloading:**
- Must differ in **parameter type**, **number**, or **order**
- Return type alone is NOT sufficient — compile error
- Access modifier can differ — but that's independent, not what distinguishes overloads

```java
public class Calculator {
    public int add(int a, int b)          { return a + b; }
    public double add(double a, double b) { return a + b; }
    public int add(int a, int b, int c)   { return a + b + c; }

    // INVALID — return type only, same signature
    // public double add(int a, int b)    { return a + b; } // COMPILE ERROR

    // Varargs overloading
    public int add(int... numbers) {
        return Arrays.stream(numbers).sum();
    }
}
```

### Overloading Resolution Order
When multiple overloads could match, Java picks in this order:
1. Exact match
2. Widening conversion (int → long)
3. Autoboxing (int → Integer)
4. Varargs

```java
void test(int x)    { System.out.println("int"); }
void test(long x)   { System.out.println("long"); }
void test(Integer x){ System.out.println("Integer"); }

test(5);  // "int" — exact match wins over widening, wins over autoboxing
```

### Common Pitfalls
- `null` passed to overloaded methods with reference types → ambiguity → compile error
- Varargs + overloading → easy to create ambiguous calls
- Overloading is NOT polymorphism — it's resolved at compile time based on declared type, not runtime type

```java
// TRAP: overloading resolved on DECLARED type
Shape s = new Circle("red", 5);
process(s);      // calls process(Shape) — not process(Circle)!

void process(Shape s)  { System.out.println("Shape"); }
void process(Circle c) { System.out.println("Circle"); }
```

---

## 8. Method Overriding

**Rules (enforced by compiler + `@Override`):**
- Same method name, same parameter list (exact), same return type (or covariant subtype)
- Access modifier must be **same or broader** (cannot restrict access)
- Cannot throw broader checked exceptions than the overridden method
- `static` methods cannot be overridden (only hidden)
- `private` methods cannot be overridden (they're invisible to subclass)
- `final` methods cannot be overridden

```java
public class Animal {
    protected String sound() { return "..."; }
    public Animal clone() throws CloneNotSupportedException { return (Animal) super.clone(); }
}

public class Dog extends Animal {
    @Override                         // Compiler validates override — always use this annotation!
    public String sound() {           // Broadened access: protected → public ✓
        return "Woof";
    }

    @Override
    public Dog clone() throws CloneNotSupportedException {  // Covariant return type ✓
        return (Dog) super.clone();
    }
}
```

### Static Methods — Hiding, Not Overriding

```java
public class Parent {
    public static void staticMethod() { System.out.println("Parent static"); }
    public void instanceMethod()      { System.out.println("Parent instance"); }
}

public class Child extends Parent {
    public static void staticMethod() { System.out.println("Child static"); }  // HIDING
    @Override
    public void instanceMethod()      { System.out.println("Child instance"); }  // OVERRIDING
}

Parent p = new Child();
p.staticMethod();   // "Parent static" — resolved at compile time (hiding)
p.instanceMethod(); // "Child instance" — resolved at runtime (overriding)
```

---

## 9. Constructors

```java
public class Person {
    private final String name;
    private final int age;
    private final String email;

    // No-arg constructor
    public Person() {
        this("Unknown", 0);  // Constructor chaining with this()
    }

    // Two-arg constructor
    public Person(String name, int age) {
        this(name, age, "");  // this() must be first statement
    }

    // Full constructor — actual initialization
    public Person(String name, int age, String email) {
        if (name == null || name.isBlank()) throw new IllegalArgumentException("Name required");
        if (age < 0) throw new IllegalArgumentException("Age must be non-negative");
        this.name = name;
        this.age = age;
        this.email = email;
    }

    // Copy constructor
    public Person(Person other) {
        this(other.name, other.age, other.email);
    }
}
```

### Constructor Rules
- Same name as class, no return type (not even void)
- If no constructor defined → compiler inserts `public ClassName() {}` (no-arg)
- If ANY constructor is defined → compiler does NOT insert no-arg constructor
- `this()` and `super()` calls must be the **first statement** — can't have both
- Constructors are NOT inherited
- Cannot be `abstract`, `static`, `final`, or `synchronized`

### Initialization Order

```java
public class InitOrder {
    // 1. Static fields (in order of declaration)
    static int staticVar = initStatic();
    static { System.out.println("2. Static initializer block"); }  // 2. Static blocks

    // 3. Instance fields (in order)
    int instanceVar = initInstance();
    { System.out.println("4. Instance initializer block"); }       // 4. Instance blocks

    // 5. Constructor body
    public InitOrder() {
        System.out.println("5. Constructor body");
    }
}
// Full order: static fields → static blocks → instance fields → instance blocks → constructor
```

---

## 10. Object Lifecycle

```
┌──────────────────────────────────────────────────────────┐
│                  OBJECT LIFECYCLE                        │
│                                                          │
│  1. CLASS LOADING                                        │
│     ClassLoader loads .class → static fields init       │
│     → static initializer blocks run                     │
│                                                          │
│  2. OBJECT CREATION                                      │
│     new Keyword → allocate heap memory                   │
│     → instance fields default init → instance blocks    │
│     → constructor runs → reference returned             │
│                                                          │
│  3. OBJECT IN USE                                        │
│     Referenced from stack/other objects → GC root       │
│                                                          │
│  4. OBJECT UNREACHABLE                                   │
│     No more strong references → eligible for GC         │
│                                                          │
│  5. GARBAGE COLLECTION                                   │
│     GC reclaims memory → finalize() (deprecated)        │
└──────────────────────────────────────────────────────────┘
```

### Reference Types and GC
```java
// Strong reference — prevents GC
Object obj = new Object();

// Soft reference — GC'd only when memory pressure
SoftReference<Object> soft = new SoftReference<>(new Object());

// Weak reference — GC'd at next collection (used in WeakHashMap)
WeakReference<Object> weak = new WeakReference<>(new Object());

// Phantom reference — queued after GC, used for cleanup
PhantomReference<Object> phantom = new PhantomReference<>(new Object(), referenceQueue);
```

---

## 11. `this` vs `super`

```java
public class Animal {
    protected String name;
    protected int age;

    public Animal(String name, int age) {
        this.name = name;   // this — disambiguates field from param
        this.age = age;
    }

    public String describe() {
        return name + ", age " + age;
    }

    public void makeSound() {
        System.out.println("...");
    }
}

public class Dog extends Animal {
    private String breed;

    public Dog(String name, int age, String breed) {
        super(name, age);       // super() — call parent constructor
        this.breed = breed;     // this — reference to current instance
    }

    public Dog(String name) {
        this(name, 0, "Mixed"); // this() — chain to sibling constructor
    }

    @Override
    public String describe() {
        return super.describe() + ", breed: " + breed;  // super.method() — call parent impl
    }

    @Override
    public void makeSound() {
        super.makeSound();  // Call parent version first if needed
        System.out.println("Woof!");
    }

    public void printSelf() {
        System.out.println(this);  // this — reference to current object
    }
}
```

### Rules Summary

| | `this` | `super` |
|---|---|---|
| Refers to | Current instance | Parent class portion |
| `this()` | Call sibling constructor | — |
| `super()` | — | Call parent constructor |
| Method call | `this.method()` (usually redundant) | `super.method()` (call parent override) |
| Position in constructor | First statement | First statement |
| Can both be in same constructor? | **No** — only one |

---

## 12. Static Members

```java
public class Counter {
    // Static variable — ONE copy shared across all instances
    private static int count = 0;
    private static final int MAX = 1000;  // Constant — naming: UPPER_SNAKE_CASE

    // Instance variable — each object has its own
    private final int id;

    public Counter() {
        count++;
        this.id = count;
    }

    // Static method — no 'this', no instance access
    public static int getCount() { return count; }
    public static void reset()   { count = 0; }

    // Instance method — can access both static and instance
    public int getId() { return id; }

    // Static initializer block — runs once when class is loaded
    static {
        System.out.println("Counter class loaded. MAX=" + MAX);
    }
}

// Usage
Counter.getCount();        // No instance needed
new Counter().getCount();  // Works but bad style — misleads reader
```

### Static Inner Classes vs Non-Static

```java
public class Outer {
    private int value = 10;

    // Non-static inner class — holds implicit reference to Outer instance
    class Inner {
        void show() { System.out.println(value); }  // can access outer's fields
    }

    // Static nested class — no reference to Outer instance
    static class StaticNested {
        void show() { System.out.println("Static nested"); }
        // Cannot access 'value' — no Outer instance
    }
}

// Non-static inner class requires outer instance
Outer.Inner inner = new Outer().new Inner();

// Static nested class — no outer instance needed
Outer.StaticNested nested = new Outer.StaticNested();
```

**Senior note:** Non-static inner classes holding outer references are a **classic memory leak** — if the inner object outlives the outer, the outer can't be GC'd. Prefer `static` nested classes unless you specifically need the outer reference.

---

## 13. `final` Keyword — Complete Coverage

```java
// 1. final VARIABLE — value/reference cannot be reassigned
final int x = 10;
x = 20;  // COMPILE ERROR

final List<String> list = new ArrayList<>();
list.add("hello");   // OK — object is mutable, only reference is final
list = new ArrayList<>();  // COMPILE ERROR — reference is final

// 2. final METHOD — cannot be overridden
public class Base {
    public final void secureMethod() { /* cannot override */ }
}

// 3. final CLASS — cannot be subclassed
public final class ImmutablePoint {
    private final int x, y;
    public ImmutablePoint(int x, int y) { this.x = x; this.y = y; }
    public int getX() { return x; }
    public int getY() { return y; }
}
// class ExtendedPoint extends ImmutablePoint {} // COMPILE ERROR

// 4. Effectively final (Java 8+) — local vars used in lambdas
int multiplier = 3;
// multiplier = 4;   // uncommenting this breaks lambda below
Runnable r = () -> System.out.println(multiplier * 10);  // effectively final
```

### `final` for Immutability Pattern

```java
// Immutable class checklist:
// 1. final class (no subclassing)
// 2. All fields private + final
// 3. No setters
// 4. Defensive copies in constructor and getters
public final class Money {
    private final BigDecimal amount;
    private final Currency currency;

    public Money(BigDecimal amount, Currency currency) {
        this.amount = Objects.requireNonNull(amount);
        this.currency = Objects.requireNonNull(currency);
    }

    public Money add(Money other) {
        if (!this.currency.equals(other.currency)) throw new IllegalArgumentException("Currency mismatch");
        return new Money(this.amount.add(other.amount), this.currency);  // returns new instance
    }

    public BigDecimal getAmount() { return amount; }  // BigDecimal is immutable — no defensive copy needed
}
```

---

## 14. Access Modifiers

```
┌─────────────────────────────────────────────────────┐
│           ACCESS MODIFIER SCOPE                     │
│                                                     │
│  Modifier    │ Class │ Package │ Subclass │ World   │
│  ────────────┼───────┼─────────┼──────────┼──────── │
│  private     │  YES  │   NO    │    NO    │  NO     │
│  (default)   │  YES  │   YES   │    NO    │  NO     │
│  protected   │  YES  │   YES   │   YES    │  NO     │
│  public      │  YES  │   YES   │   YES    │  YES    │
└─────────────────────────────────────────────────────┘
```

```java
package com.example.animals;

public class Animal {
    public String publicName = "Animal";        // accessible everywhere
    protected int protectedAge = 5;             // package + subclasses
    String packageField = "default";            // package only
    private String privateDNA = "ATCG";         // this class only

    // protected method accessible to subclasses in OTHER packages
    protected void breathe() { System.out.println("Breathing"); }
}

// In different package
package com.example.dogs;

public class Dog extends Animal {
    void test() {
        System.out.println(publicName);     // OK
        System.out.println(protectedAge);   // OK — subclass
        // System.out.println(packageField); // ERROR — different package
        // System.out.println(privateDNA);   // ERROR — private
        breathe();                          // OK — protected, subclass
    }
}
```

**Best practice:** Use the most restrictive access that works. Default to `private`; promote to `protected` for subclass needs; `public` only for intended API.

---

## 15. Composition vs Inheritance

**Composition:** A class HAS-A reference to another class.
**Inheritance:** A class IS-A more specific version of another class.

```java
// INHERITANCE approach — often overused
class Stack extends ArrayList<Integer> {
    // Inherits 28+ methods from ArrayList — most are inappropriate for a stack
    // Users can call add(0, element), remove(0), etc. — violates stack contract
    // This is a famous mistake in the Java standard library itself (java.util.Stack)
}

// COMPOSITION approach — correct
class Stack<T> {
    private final Deque<T> storage = new ArrayDeque<>();  // HAS-A, not IS-A

    public void push(T item) { storage.push(item); }
    public T pop()           { return storage.pop(); }
    public T peek()          { return storage.peek(); }
    public boolean isEmpty() { return storage.isEmpty(); }
    public int size()        { return storage.size(); }
    // Only exposes stack-appropriate operations — clean API
}
```

### Rule of Thumb: Liskov Substitution Test
Before using inheritance, ask: *Can I always substitute a subclass where the superclass is expected without breaking behavior?*

```java
// Violation example — the classic Rectangle/Square problem
class Rectangle {
    protected int width, height;
    public void setWidth(int w)  { this.width = w; }
    public void setHeight(int h) { this.height = h; }
    public int area() { return width * height; }
}

class Square extends Rectangle {
    @Override public void setWidth(int w)  { this.width = this.height = w; }
    @Override public void setHeight(int h) { this.width = this.height = h; }
}

// This code breaks with Square — violates LSP
void resize(Rectangle r) {
    r.setWidth(5);
    r.setHeight(3);
    assert r.area() == 15; // Fails for Square! area = 9
}
```

### Composition Benefits
- Flexibility — swap implementation at runtime (Strategy pattern)
- No fragile base class problem
- Easier to test — inject mocks
- No unwanted inherited methods leaking into public API

---

## 16. Association, Aggregation, Composition (UML Relations)

```
Association:   A uses B — weakest relationship
               Teacher ──── Student (Teacher references Student)

Aggregation:   A has B — B can exist without A (weak ownership)
               Department ◇──── Employee (Employee can outlive Department)

Composition:   A owns B — B cannot exist without A (strong ownership)
               House ◆──── Room (Room destroyed when House is destroyed)
```

```java
// Association — A has a reference to B, no ownership
public class Teacher {
    private List<Student> students;   // association — students exist independently
}

// Aggregation — owns collection but members exist independently
public class Department {
    private List<Employee> employees;

    public void addEmployee(Employee e) { employees.add(e); }
    public void removeEmployee(Employee e) { employees.remove(e); }
    // Employee objects not created or destroyed here
}

// Composition — creates and owns; parts destroyed with parent
public class House {
    private final List<Room> rooms;  // Composition

    public House(int numRooms) {
        rooms = new ArrayList<>();
        for (int i = 0; i < numRooms; i++) {
            rooms.add(new Room(i + 1));  // House creates rooms
        }
        // No way to get a Room reference out — fully owned
    }
}
```

---

## 17. Object Equality — `equals` and `hashCode`

This is one of the most critical and most broken contracts in Java.

### The Contract

```
equals() contract (reflexive, symmetric, transitive, consistent, null-safe):
  - x.equals(x) == true                          (reflexive)
  - x.equals(y) == y.equals(x)                   (symmetric)
  - if x.equals(y) && y.equals(z) → x.equals(z)  (transitive)
  - multiple calls → same result                  (consistent)
  - x.equals(null) == false                       (null-safe)

hashCode() contract:
  - If x.equals(y) → x.hashCode() == y.hashCode()  (MANDATORY)
  - If !x.equals(y) → hashCodes MAY differ          (not required but good for performance)
```

### Correct Implementation

```java
public class Employee {
    private final String id;          // unique business key
    private final String name;
    private final String department;

    public Employee(String id, String name, String department) {
        this.id = Objects.requireNonNull(id);
        this.name = Objects.requireNonNull(name);
        this.department = Objects.requireNonNull(department);
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;                    // Same reference shortcut
        if (o == null || getClass() != o.getClass()) return false;  // null + type check
        Employee employee = (Employee) o;
        return Objects.equals(id, employee.id);        // Business key equality — often just ID
        // Note: only including fields that define identity, not all fields
    }

    @Override
    public int hashCode() {
        return Objects.hash(id);   // Must include SAME fields as equals
    }

    @Override
    public String toString() {
        return String.format("Employee{id='%s', name='%s', department='%s'}", id, name, department);
    }
}
```

### The Breaking Consequences

```java
// What happens when you break hashCode contract
Set<Employee> set = new HashSet<>();
Employee e1 = new Employee("001", "Alice", "Engineering");
set.add(e1);

Employee e2 = new Employee("001", "Alice", "Engineering");  // Same business key
System.out.println(e1.equals(e2));  // true (if equals correct)
System.out.println(set.contains(e2)); // FALSE if hashCode not overridden!
// HashSet uses hashCode to find bucket → wrong bucket → not found

// HashMap same issue
Map<Employee, String> map = new HashMap<>();
map.put(e1, "Senior Dev");
System.out.println(map.get(e2));  // null! Can't find the entry
```

### `equals` with Inheritance — Symmetry Trap

```java
// BROKEN — symmetry violation
class Point {
    int x, y;
    @Override public boolean equals(Object o) {
        if (!(o instanceof Point)) return false;
        Point p = (Point) o;
        return x == p.x && y == p.y;
    }
}

class ColorPoint extends Point {
    String color;
    @Override public boolean equals(Object o) {
        if (!(o instanceof ColorPoint)) return false;
        ColorPoint cp = (ColorPoint) o;
        return super.equals(o) && color.equals(cp.color);
    }
}

Point p   = new Point(1, 2);
ColorPoint cp = new ColorPoint(1, 2, "red");

p.equals(cp);   // true  — Point's equals ignores color
cp.equals(p);   // false — ColorPoint's equals rejects non-ColorPoint
// SYMMETRY BROKEN → bugs in collections

// FIX: use getClass() check OR use composition instead of inheritance
```

---

## 18. `toString` Contract

```java
// Contract: return a human-readable string describing the object's state
// Default from Object: ClassName@hexHashCode — useless for debugging!

public class Order {
    private String orderId;
    private List<String> items;
    private double total;

    @Override
    public String toString() {
        return "Order{" +
               "orderId='" + orderId + '\'' +
               ", items=" + items +
               ", total=" + total +
               '}';
    }

    // Modern approach with String.format
    @Override
    public String toString() {
        return String.format("Order{id='%s', items=%s, total=%.2f}", orderId, items, total);
    }
}

// toString is called implicitly by:
System.out.println(order);          // implicit toString()
"Order: " + order                   // implicit toString()
logger.info("Processing {}", order) // logging frameworks call toString()
```

**Senior note:** `toString` should include enough state for debugging but omit sensitive data (passwords, PII, tokens). Never include mutable large collections inline — it can produce enormous log lines.

---

## 19. Interview Questions with Expert Answers

**Q1: What are the four pillars of OOP and how does Java implement each?**
> Encapsulation — access modifiers + getters/setters + validation logic in methods. Inheritance — `extends` keyword for single class inheritance, `implements` for multiple interfaces. Polymorphism — method overriding (runtime dispatch via vtable), method overloading (compile-time dispatch). Abstraction — `abstract` classes and `interface` declarations hide implementation details behind contracts.

---

**Q2: What is the difference between method overloading and method overriding?**
> Overloading is compile-time (static) polymorphism — same method name, different parameter signatures, resolved by the compiler based on declared types. Overriding is runtime (dynamic) polymorphism — same signature in subclass, resolved at runtime based on actual object type. Key trap: overloading is NOT affected by the runtime type of an object, only by the declared type at the call site.

---

**Q3: Why should you favor composition over inheritance?**
> Inheritance creates tight coupling — changes to superclass can break all subclasses (fragile base class). It also exposes you to the Liskov Substitution Problem when the IS-A relationship breaks down (Square extending Rectangle). Composition keeps classes loosely coupled, allows swapping implementations at runtime, avoids inheriting unwanted methods, and is easier to test with mocks. The Java standard library's `java.util.Stack extends Vector` is a famous example of inappropriate inheritance.

---

**Q4: Why must you override `hashCode` whenever you override `equals`?**
> The Java contract requires: if `a.equals(b)` then `a.hashCode() == b.hashCode()`. Hash-based collections (`HashMap`, `HashSet`, `Hashtable`) use `hashCode` to find the bucket, then `equals` to confirm the match. If you override `equals` without `hashCode`, two logically equal objects can land in different buckets, causing `contains()` to return false and `get()` to return null — silent data corruption bugs that are very hard to trace.

---

**Q5: What is the Liskov Substitution Principle?**
> LSP states that objects of a subclass should be substitutable for objects of the superclass without altering correctness. A subclass should strengthen postconditions (do at least as much), weaken preconditions (accept at least as broad input), and not add new exceptions. The classic violation is Square extending Rectangle — setting width on a Square also changes height, breaking the Rectangle contract that width and height are independent.

---

**Q6: What is the difference between `abstract` class and `interface`?**
> Abstract class: can have state (fields), constructors, concrete methods, any access modifiers — models a "is-a" base with partial implementation. Interface: contract-only (until Java 8 default methods), no instance state, no constructors, implicitly public — models a "can-do" capability. A class can implement multiple interfaces but extend only one abstract class. Use abstract class for shared base implementation with state; use interface for defining capabilities/contracts across unrelated class hierarchies.

---

**Q7: What happens if a parent constructor calls an overridable method?**
> This is a dangerous pattern. When a subclass object is created, the parent constructor runs first. If it calls a method overridden in the subclass, the subclass override executes before the subclass constructor has initialized its fields. Instance fields are at their default values (null, 0, false), so the override can see uninitialized state and throw NullPointerException or behave incorrectly. Fix: don't call overridable methods from constructors; use `private` or `final` methods in constructors instead.

---

**Q8: Explain `protected` access and its inheritance behavior.**
> `protected` members are accessible within the same package AND in subclasses (even in different packages). However, a subclass in a different package can only access the protected member through its own type, not through a reference to the parent type. This is subtle: `super.method()` works, but `((Animal) this).method()` does not give you access to protected members of Animal from a different-package subclass through a parent reference.

---

**Q9: What is the difference between composition, aggregation, and association?**
> Association is the loosest — one class knows about another (has a reference). Aggregation adds weak ownership — a Department has Employees, but Employees have an independent lifecycle. Composition is strong ownership — a House has Rooms; when the House is destroyed, the Rooms cease to exist too. In code, composition is expressed by creating objects inside the constructor and not exposing them; aggregation accepts objects from outside and may expose them.

---

**Q10: Can you override a `static` method?**
> No. Static methods can be re-declared in a subclass — this is called method hiding, not overriding. The key difference: with overriding, the method called depends on the runtime type of the object. With hiding, it depends on the compile-time (declared) type. `Parent p = new Child(); p.staticMethod()` calls Parent's static method even if Child defines one with the same signature. The `@Override` annotation on a static method causes a compile error.

---

**Q11: What is the difference between `==` and `equals()` for objects?**
> `==` compares references — are both variables pointing to the exact same object in memory? `equals()` compares logical content — are the two objects considered equal by the class's definition? For value objects (String, Integer, your domain objects), always use `equals()`. Only use `==` when you deliberately want to check reference identity, or for comparing enums (safe because enum constants are singletons) and null checks (`obj == null`).

---

**Q12: What is the `toString` contract and when is it called implicitly?**
> `toString` should return a string that clearly represents the object's state, useful for debugging and logging. It is called implicitly in string concatenation (`"value: " + obj`), by `System.out.println(obj)`, by most logging frameworks, and by debugger tools. The default `Object.toString()` returns `ClassName@hexHashCode` — useless for debugging. Always override it in value/domain objects. Avoid including sensitive fields (passwords, tokens) in `toString` output.

---

**Q13: What is encapsulation and how is it different from data hiding?**
> Data hiding is just making fields private. Encapsulation is broader — it means bundling state and behavior together so the class manages its own invariants, and external code interacts only through a controlled interface. A class with all-private fields but public getters/setters for everything is doing data hiding without real encapsulation — any invariant can still be violated externally. True encapsulation means the class's behavior guarantees its own correctness regardless of how it's used.

---

**Q14: How does Java achieve multiple inheritance?**
> Java doesn't allow multiple class inheritance (to avoid the diamond problem). It achieves multiple inheritance of type via interfaces — a class can implement many interfaces. Since Java 8, interfaces can have `default` methods with implementations, which introduces a form of multiple inheritance of behavior. If two interfaces provide conflicting default methods, the implementing class must override the method to resolve the ambiguity. The diamond problem with abstract state (fields) is avoided because interfaces cannot have instance fields.

---

**Q15: What distinguishes a well-designed class from a poorly designed one in OOP terms?**
> A well-designed class: (1) has a single, clear responsibility (SRP), (2) encapsulates its invariants — no external code can put it in an invalid state, (3) has a minimal, intentional public API, (4) correctly implements `equals`/`hashCode`/`toString` if it's a value object, (5) prefers composition over deep inheritance, (6) is immutable where possible, (7) fails fast with clear exceptions on invalid input. Poor classes: public fields, no validation, god objects with 50 methods, deep inheritance trees that violate LSP, broken equals/hashCode contracts.

---

## 20. Advanced / Senior-Level Considerations

### Design Patterns Rooted in OOP
- **Template Method** — abstract class defines algorithm skeleton; subclasses fill in steps
- **Strategy** — composition replaces inheritance; swap behavior at runtime via interface
- **Decorator** — wrap objects to add behavior without subclassing (used in Java I/O streams)
- **Factory Method** — subclasses decide which class to instantiate
- **Proxy** — same interface, different implementation (Spring AOP, Hibernate lazy loading)

### SOLID Principles (senior expected to know)
```
S — Single Responsibility: one class, one reason to change
O — Open/Closed: open for extension, closed for modification (use interfaces)
L — Liskov Substitution: subclasses must honor parent contracts
I — Interface Segregation: many small interfaces > one large one
D — Dependency Inversion: depend on abstractions, not concretions
```

### OOP and Performance
- Virtual method dispatch (overriding) has a small overhead vs static dispatch (final methods)
- JIT's inline caching and devirtualization can optimize away the overhead for monomorphic call sites
- Deep inheritance + overriding = harder for JIT to inline → prefer `final` for performance-critical methods
- Non-static inner class memory leak: holds implicit outer reference — prefer `static` nested

### Java Records (Java 16+) — OOP Simplified
```java
// Immutable value object — compiler generates constructor, equals, hashCode, toString
public record Point(int x, int y) {
    // Compact constructor — for validation
    public Point {
        if (x < 0 || y < 0) throw new IllegalArgumentException("Coordinates must be positive");
    }

    // Can add instance methods
    public double distanceTo(Point other) {
        return Math.sqrt(Math.pow(x - other.x, 2) + Math.pow(y - other.y, 2));
    }
}
// equals/hashCode compare ALL fields — appropriate for value objects
```

---

## 21. Summary / Cheat Sheet

```
FOUR PILLARS:
  Encapsulation  → private fields + controlled access + invariant protection
  Abstraction    → interface/abstract class → hide HOW, expose WHAT
  Inheritance    → extends (IS-A) → single class, multiple interfaces
  Polymorphism   → overriding (runtime dispatch) + overloading (compile-time)

OVERLOADING: same name, different params → compile-time resolution on DECLARED type
OVERRIDING:  same signature, subclass    → runtime resolution on ACTUAL type
  Rules: same/wider access, same/covariant return, no broader checked exceptions

CONSTRUCTORS:
  this() or super() must be FIRST statement
  super() auto-inserted if you don't call it (needs parent no-arg!)
  Init order: static fields → static blocks → instance fields → instance blocks → constructor

EQUALS/HASHCODE CONTRACT:
  a.equals(b) → a.hashCode() == b.hashCode()   (MANDATORY)
  Override BOTH or neither
  Use Objects.equals() + Objects.hash() to avoid NPEs

COMPOSITION > INHERITANCE when:
  - Relationship is HAS-A not IS-A
  - Multiple sources of behavior needed
  - Liskov Substitution would be violated
  - You want to swap implementations

ACCESS: private < (default) < protected < public
  Use most restrictive access that works

FINAL:
  variable  → no reassign (object still mutable!)
  method    → no override
  class     → no subclass (e.g., String, Integer)

THIS vs SUPER:
  this   → current instance / sibling constructor
  super  → parent class / parent constructor

STATIC:
  belongs to CLASS not instance
  no 'this', no instance field access
  static nested class preferred over non-static (avoids memory leak)
```

# Java Classes & Core APIs — Senior Developer Interview Study Notes

---

## 1. Topic Overview

Java's core APIs form the foundation every Java application is built on. Mastery of these APIs separates developers who fight the language from those who leverage it. For senior interviews, the depth expected goes well beyond basic usage — examiners want to see understanding of **design contracts, performance implications, thread safety, and modern idiomatic Java**.

**Real-world relevance:**
- `Optional` misuse is one of the most common code review issues in modern Java codebases
- Broken `equals`/`hashCode` causes silent bugs in every HashMap/HashSet usage
- `Date`/`Calendar` are legacy minefields still found in most enterprise codebases
- Enum and annotation misunderstanding leads to framework misconfiguration (Spring, JPA, Jackson)

---

## 2. Object Class Methods

Every Java class implicitly extends `java.lang.Object`. Its methods form the universal contract for all Java objects.

```
java.lang.Object
├── equals(Object obj)
├── hashCode()
├── toString()
├── getClass()
├── clone()
├── finalize()          ← deprecated Java 9+
├── wait()
├── wait(long timeout)
├── wait(long timeout, int nanos)
├── notify()
└── notifyAll()
```

### `equals` and `hashCode` — The Critical Contract

```java
public final class ProductId {
    private final String value;

    public ProductId(String value) {
        this.value = Objects.requireNonNull(value, "value must not be null");
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;                         // 1. Reference shortcut
        if (o == null || getClass() != o.getClass())        // 2. Null + type check
            return false;
        ProductId productId = (ProductId) o;
        return Objects.equals(value, productId.value);      // 3. Field comparison
    }

    @Override
    public int hashCode() {
        return Objects.hash(value);   // Must use SAME fields as equals
    }

    @Override
    public String toString() {
        return "ProductId{value='" + value + "'}";
    }
}
```

### `clone()` — Deep vs Shallow

```java
public class Order implements Cloneable {
    private String id;
    private List<String> items;   // mutable reference — needs deep copy

    @Override
    public Order clone() {
        try {
            Order cloned = (Order) super.clone();   // shallow copy of all fields
            cloned.items = new ArrayList<>(this.items);  // deep copy mutable fields
            return cloned;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError("Should never happen", e);
        }
    }
}
```

**Senior note:** `clone()` is widely considered broken by design (Josh Bloch, *Effective Java*). Prefer copy constructors or factory methods. The `Cloneable` interface is a marker with no method — the actual `clone()` is in `Object` with `protected` access, making the design incoherent.

### `wait`, `notify`, `notifyAll` — Monitor-Based Concurrency

```java
// These operate on the intrinsic monitor lock of the object
// Must be called from a synchronized block/method — else IllegalMonitorStateException

class MessageQueue {
    private final Queue<String> queue = new LinkedList<>();

    public synchronized void produce(String msg) throws InterruptedException {
        while (queue.size() >= 10) {
            wait();             // Releases lock and waits
        }
        queue.add(msg);
        notifyAll();            // Wake up all waiting threads
    }

    public synchronized String consume() throws InterruptedException {
        while (queue.isEmpty()) {
            wait();             // Must wait in loop — guard against spurious wakeups
        }
        String msg = queue.poll();
        notifyAll();
        return msg;
    }
}
```

**Key rules:**
- Always call `wait()` in a `while` loop, not `if` — spurious wakeups are real
- `notify()` wakes one arbitrary thread; `notifyAll()` wakes all — prefer `notifyAll()` for safety
- `wait()` atomically releases the lock — `Thread.sleep()` does NOT release the lock

### `getClass()` vs `instanceof`

```java
Object obj = "hello";

// getClass — exact type, no inheritance
System.out.println(obj.getClass());         // class java.lang.String
System.out.println(obj.getClass() == String.class);  // true
System.out.println(obj.getClass() == Object.class);  // false

// instanceof — respects inheritance hierarchy
System.out.println(obj instanceof String);  // true
System.out.println(obj instanceof Object);  // true
System.out.println(obj instanceof CharSequence); // true

// Pattern matching instanceof (Java 16+) — eliminates cast
if (obj instanceof String s && s.length() > 3) {
    System.out.println(s.toUpperCase());    // s is already String — no cast
}
```

---

## 3. String vs StringBuilder vs StringBuffer

Covered deeply in Java Fundamentals notes. Senior-level additions:

### String Interning and Memory

```java
String s1 = "hello";                    // Pool
String s2 = "hello";                    // Same pool object
String s3 = new String("hello");        // New heap object
String s4 = s3.intern();                // Returns pool reference

System.out.println(s1 == s2);   // true
System.out.println(s1 == s3);   // false
System.out.println(s1 == s4);   // true

// Java 9+: Compact Strings — String internally uses byte[] not char[]
// Latin-1 strings use 1 byte per char instead of 2 — ~50% memory saving for ASCII
// Transparent to developer — no API change
```

### StringBuilder Internals

```java
StringBuilder sb = new StringBuilder();    // default capacity: 16 chars
StringBuilder sb2 = new StringBuilder(256); // pre-sized — avoids resizing

// Internal array doubles when capacity exceeded — amortized O(1) append
// Pre-size when you know approximate length — avoids array copy overhead

// Chaining — returns 'this' so calls can be chained
String result = new StringBuilder()
    .append("Hello")
    .append(", ")
    .append("World")
    .append("!")
    .toString();

// Java 9+ string concat uses StringConcatFactory (invokedynamic)
// Simple "a" + "b" + "c" is optimized at bytecode level — but loops still need StringBuilder
```

### Performance Benchmark Perspective

```java
// O(n²) — avoid
String result = "";
for (int i = 0; i < 100_000; i++) result += i;

// O(n) — use this
StringBuilder sb = new StringBuilder(700_000); // pre-sized estimate
for (int i = 0; i < 100_000; i++) sb.append(i);
String result = sb.toString();

// StringBuffer — only when shared across threads (rare)
// Prefer higher-level sync (ConcurrentLinkedQueue of chunks) over StringBuffer in practice
```

---

## 4. Math API

```java
// Basic operations
Math.abs(-5)           // 5
Math.max(3, 7)         // 7
Math.min(3, 7)         // 3
Math.pow(2, 10)        // 1024.0
Math.sqrt(144)         // 12.0
Math.cbrt(27)          // 3.0

// Rounding
Math.floor(3.9)        // 3.0 — round down
Math.ceil(3.1)         // 4.0 — round up
Math.round(3.5)        // 4   — round half-up (returns long/int)
Math.rint(3.5)         // 4.0 — round to nearest even (banker's rounding)

// Logarithms and trigonometry
Math.log(Math.E)       // 1.0  — natural log
Math.log10(1000)       // 3.0
Math.sin(Math.PI / 2)  // 1.0
Math.cos(0)            // 1.0

// Constants
Math.PI    // 3.141592653589793
Math.E     // 2.718281828459045

// Safe arithmetic — throws ArithmeticException on overflow (Java 8+)
Math.addExact(Integer.MAX_VALUE, 1)      // ArithmeticException: integer overflow
Math.multiplyExact(100_000, 100_000)     // ArithmeticException: overflow
// Use these in financial/critical code instead of silent overflow

// Random — NOT cryptographically secure
Math.random()          // [0.0, 1.0) — uses Random internally
// For production randomness: use SecureRandom
// For range: ThreadLocalRandom.current().nextInt(min, max)
```

---

## 5. Date and Time API

### Legacy API (Know to Recognize and Migrate)

```java
// java.util.Date — avoid in new code
Date date = new Date();             // current time
date.getYear()                      // returns year - 1900 — absurd!
date.getMonth()                     // 0-indexed — January = 0!

// java.util.Calendar — slightly better but still terrible
Calendar cal = Calendar.getInstance();
cal.set(Calendar.MONTH, Calendar.JANUARY);  // 0-indexed months
cal.add(Calendar.DAY_OF_MONTH, 5);

// SimpleDateFormat — NOT thread-safe! Classic concurrency bug
SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
// Sharing sdf across threads causes data corruption — use ThreadLocal or java.time
```

### Modern API: `java.time` (Java 8+) — Always Use This

```
java.time hierarchy:
├── LocalDate          — date only (2024-03-15)
├── LocalTime          — time only (14:30:00)
├── LocalDateTime      — date + time, no timezone
├── ZonedDateTime      — date + time + timezone
├── Instant            — machine time (epoch seconds + nanos) — use for timestamps
├── Duration           — amount of time (seconds/nanos based)
├── Period             — date-based amount (years/months/days)
├── ZoneId             — timezone identifier (e.g., "Europe/London")
├── DateTimeFormatter  — thread-safe formatting/parsing
└── OffsetDateTime     — date + time + UTC offset (good for APIs)
```

```java
import java.time.*;
import java.time.format.*;
import java.time.temporal.*;

// --- LocalDate ---
LocalDate today = LocalDate.now();
LocalDate birthday = LocalDate.of(1990, Month.JUNE, 15);
LocalDate nextWeek = today.plusWeeks(1);
LocalDate lastMonth = today.minusMonths(1);

boolean isLeap = today.isLeapYear();
DayOfWeek dow = today.getDayOfWeek();              // MONDAY, TUESDAY, etc.
int dayOfYear = today.getDayOfYear();

// --- LocalTime ---
LocalTime now = LocalTime.now();
LocalTime meeting = LocalTime.of(14, 30, 0);
LocalTime later = meeting.plusHours(2);
boolean before = meeting.isBefore(later);          // true

// --- LocalDateTime ---
LocalDateTime ldt = LocalDateTime.of(today, meeting);
LocalDateTime parsed = LocalDateTime.parse("2024-03-15T14:30:00");

// --- ZonedDateTime ---
ZonedDateTime zdt = ZonedDateTime.now(ZoneId.of("America/New_York"));
ZonedDateTime utc = zdt.withZoneSameInstant(ZoneId.of("UTC"));
ZonedDateTime london = zdt.withZoneSameInstant(ZoneId.of("Europe/London"));

// --- Instant — best for storing timestamps ---
Instant now2 = Instant.now();
long epochMillis = now2.toEpochMilli();
Instant fromMillis = Instant.ofEpochMilli(System.currentTimeMillis());

// Convert between Instant and ZonedDateTime
ZonedDateTime fromInstant = instant.atZone(ZoneId.systemDefault());
Instant backToInstant = fromInstant.toInstant();

// --- Duration and Period ---
Duration duration = Duration.between(LocalTime.of(9, 0), LocalTime.of(17, 30));
long hours = duration.toHours();       // 8
long minutes = duration.toMinutes();   // 510

Period period = Period.between(birthday, today);
int years = period.getYears();

// --- DateTimeFormatter — THREAD-SAFE unlike SimpleDateFormat ---
DateTimeFormatter fmt = DateTimeFormatter.ofPattern("dd/MM/yyyy HH:mm");
String formatted = ldt.format(fmt);
LocalDateTime parsed2 = LocalDateTime.parse("15/03/2024 14:30", fmt);

// Built-in formatters
String iso = today.format(DateTimeFormatter.ISO_LOCAL_DATE);  // "2024-03-15"

// --- Practical: business days calculation ---
public static long businessDaysBetween(LocalDate start, LocalDate end) {
    return start.datesUntil(end)
        .filter(d -> d.getDayOfWeek() != DayOfWeek.SATURDAY
                  && d.getDayOfWeek() != DayOfWeek.SUNDAY)
        .count();
}
```

### Legacy ↔ Modern Conversion

```java
// Date → Instant → ZonedDateTime
Date legacyDate = new Date();
Instant instant = legacyDate.toInstant();
ZonedDateTime modern = instant.atZone(ZoneId.systemDefault());

// ZonedDateTime → Date (for legacy APIs)
Date backToLegacy = Date.from(modern.toInstant());

// Calendar → ZonedDateTime
Calendar cal = Calendar.getInstance();
ZonedDateTime fromCal = cal.toInstant().atZone(ZoneId.systemDefault());

// java.sql.Timestamp (JDBC)
Timestamp ts = Timestamp.from(Instant.now());
Instant fromTs = ts.toInstant();
```

---

## 6. Optional Class

`Optional<T>` is a container that may or may not contain a value. It is a tool for expressing the absence of a value in return types — **not a universal null replacement**.

### Correct Usage Patterns

```java
// CREATION
Optional<String> present = Optional.of("hello");        // throws NPE if null
Optional<String> nullable = Optional.ofNullable(null);  // safe with null
Optional<String> empty = Optional.empty();

// CHECKING AND EXTRACTING
optional.isPresent()           // true if value exists
optional.isEmpty()             // true if empty (Java 11+)
optional.get()                 // returns value OR throws NoSuchElementException!

// SAFE EXTRACTION — prefer these over get()
optional.orElse("default")                   // value or default (always evaluated!)
optional.orElseGet(() -> computeDefault())   // lazy — supplier only called if empty
optional.orElseThrow()                       // throw NoSuchElementException if empty (Java 10+)
optional.orElseThrow(() -> new EntityNotFoundException("Not found"))

// TRANSFORMATION
optional.map(String::toUpperCase)             // Optional<String>
optional.flatMap(s -> findById(s))            // prevents Optional<Optional<T>>
optional.filter(s -> s.startsWith("A"))       // Optional<String> or empty

// CONSUMING
optional.ifPresent(System.out::println)
optional.ifPresentOrElse(                     // Java 9+
    val -> System.out.println("Found: " + val),
    () -> System.out.println("Not found")
);

// CHAINING — the power of Optional
public Optional<String> findEmailByUserId(String userId) {
    return userRepository.findById(userId)          // Optional<User>
        .filter(User::isActive)                     // Optional<User>
        .map(User::getProfile)                      // Optional<Profile>
        .map(Profile::getEmail);                    // Optional<String>
}
```

### What Optional is NOT for

```java
// BAD — Optional as method parameter (forces callers into Optional)
public void process(Optional<String> name) { ... }  // wrong
public void process(String name) { ... }            // correct

// BAD — Optional as field (not serializable, heap overhead)
private Optional<String> middleName;    // wrong
private String middleName;              // nullable field, documented

// BAD — Optional in collections (pointless — use the collection's absence)
List<Optional<String>> list;  // wrong
List<String> list;            // correct

// BAD — calling get() without checking
String val = optional.get();  // throws if empty — defeats the purpose

// BAD — orElse with expensive computation
optional.orElse(expensiveDbCall());   // DB call happens EVEN IF optional has value!
optional.orElseGet(() -> expensiveDbCall());  // lazy — only called when needed

// GOOD — Optional as method return type to signal nullable result
public Optional<User> findUserByEmail(String email) { ... }
```

---

## 7. UUID

```java
import java.util.UUID;

// Generate random UUID (Version 4) — 122 bits of randomness
UUID id = UUID.randomUUID();
System.out.println(id);          // e.g., "550e8400-e29b-41d4-a716-446655440000"

// Parse from string
UUID parsed = UUID.fromString("550e8400-e29b-41d4-a716-446655440000");

// Version 3/5 — name-based (deterministic)
UUID nameUUID = UUID.nameUUIDFromBytes("user@example.com".getBytes());
// Same input → same UUID every time

// Properties
UUID uuid = UUID.randomUUID();
uuid.version()   // 4 for random
uuid.variant()   // 2 for RFC 4122
uuid.toString()  // "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
uuid.getMostSignificantBits()   // long
uuid.getLeastSignificantBits()  // long

// String representation: 36 chars with dashes, 32 without
String withDashes    = uuid.toString();              // "a3b5c7d9-..."
String withoutDashes = uuid.toString().replace("-", ""); // "a3b5c7d9..."

// As primary key — common in distributed systems
// Pros: globally unique, no coordination needed, embeds version/time
// Cons: not sequential → index fragmentation in B-tree databases
// Alternative: ULID (lexicographically sortable), Snowflake IDs
```

---

## 8. Enums

Enums in Java are full-fledged classes — far more powerful than in other languages.

### Basic to Advanced Enum Usage

```java
// Simple enum
public enum Direction { NORTH, SOUTH, EAST, WEST }

// Enum with fields, constructor, methods
public enum Planet {
    MERCURY(3.303e+23, 2.4397e6),
    VENUS  (4.869e+24, 6.0518e6),
    EARTH  (5.976e+24, 6.37814e6),
    MARS   (6.421e+23, 3.3972e6);

    private final double mass;    // kg
    private final double radius;  // meters

    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
    }

    static final double G = 6.67300E-11;

    public double surfaceGravity() {
        return G * mass / (radius * radius);
    }

    public double surfaceWeight(double otherMass) {
        return otherMass * surfaceGravity();
    }
}

double earthWeight = 75.0;
double mass = earthWeight / Planet.EARTH.surfaceGravity();
for (Planet p : Planet.values()) {
    System.out.printf("Weight on %s is %6.2f%n", p, p.surfaceWeight(mass));
}
```

### Abstract Methods in Enums

```java
public enum Operation {
    PLUS("+") {
        @Override public double apply(double x, double y) { return x + y; }
    },
    MINUS("-") {
        @Override public double apply(double x, double y) { return x - y; }
    },
    TIMES("*") {
        @Override public double apply(double x, double y) { return x * y; }
    },
    DIVIDE("/") {
        @Override public double apply(double x, double y) { return x / y; }
    };

    private final String symbol;

    Operation(String symbol) { this.symbol = symbol; }

    public abstract double apply(double x, double y);  // Each constant implements this

    @Override public String toString() { return symbol; }
}

// Usage — no if/switch needed, polymorphic dispatch on enum constant
double result = Operation.PLUS.apply(3, 4);  // 7.0
```

### Enum as Singleton / Strategy

```java
// Thread-safe singleton via enum (Josh Bloch's recommendation)
public enum AppConfig {
    INSTANCE;

    private final Properties props = loadProperties();

    public String getProperty(String key) { return props.getProperty(key); }
    private Properties loadProperties() { /* load from file */ return new Properties(); }
}
AppConfig.INSTANCE.getProperty("db.url");
```

### Enum Utilities

```java
// Built-in methods all enums inherit
Direction d = Direction.NORTH;
d.name()            // "NORTH" — always the declared name
d.ordinal()         // 0 — position (0-based) — fragile! don't use for persistence
d.toString()        // "NORTH" by default, overridable

Direction.valueOf("SOUTH")    // Direction.SOUTH — throws IllegalArgumentException if not found
Direction.values()            // Direction[] — all constants, in declaration order

// EnumSet — highly efficient (bit vector internally)
EnumSet<Direction> horizontal = EnumSet.of(Direction.EAST, Direction.WEST);
EnumSet<Direction> all = EnumSet.allOf(Direction.class);
EnumSet<Direction> none = EnumSet.noneOf(Direction.class);

// EnumMap — array-backed, very fast
EnumMap<Direction, String> labels = new EnumMap<>(Direction.class);
labels.put(Direction.NORTH, "Up");
labels.put(Direction.SOUTH, "Down");
```

### Common Pitfalls

```java
// NEVER use ordinal() for persistence or logic — order can change!
// BAD
int stored = direction.ordinal();

// GOOD — use name() or a dedicated field
String stored = direction.name();
// OR better: a custom code field
public enum Status {
    ACTIVE("A"), INACTIVE("I");
    private final String code;
    Status(String code) { this.code = code; }
    public String getCode() { return code; }
}
```

---

## 9. Records

Records (Java 16+) are immutable data carriers. The compiler auto-generates constructor, `equals`, `hashCode`, `toString`, and accessors.

```java
// Declaration — all fields are final, private
public record Point(int x, int y) { }

// Generated by compiler:
// - Canonical constructor: public Point(int x, int y)
// - Accessors: x(), y()  (NOT getX/getY)
// - equals/hashCode using ALL components
// - toString: "Point[x=1, y=2]"

Point p1 = new Point(1, 2);
Point p2 = new Point(1, 2);
System.out.println(p1.x());        // 1 — accessor, not getter
System.out.println(p1.equals(p2)); // true
System.out.println(p1);            // Point[x=1, y=2]
```

### Custom Constructors and Methods

```java
public record Range(int min, int max) {

    // Compact canonical constructor — validation, normalization (no parameter list)
    public Range {
        if (min > max) throw new IllegalArgumentException("min must be <= max");
        // Assignment to fields happens automatically after this block
    }

    // Custom constructor
    public Range(int value) {
        this(value, value);   // delegates to canonical
    }

    // Instance methods allowed
    public int length() { return max - min; }
    public boolean contains(int value) { return value >= min && value <= max; }

    // Static factory
    public static Range of(int min, int max) { return new Range(min, max); }
}
```

### Records with Interfaces and Generics

```java
// Records can implement interfaces
public record NamedValue<T>(String name, T value) implements Comparable<NamedValue<T>>
        where T extends Comparable<T> {

    @Override
    public int compareTo(NamedValue<T> other) {
        return this.value.compareTo(other.value);
    }
}

// Records in sealed hierarchies (Java 17+)
public sealed interface Shape permits Circle, Rectangle, Triangle { }
public record Circle(double radius) implements Shape { }
public record Rectangle(double width, double height) implements Shape { }
public record Triangle(double base, double height) implements Shape { }

// Pattern matching with sealed + records (Java 21)
double area = switch (shape) {
    case Circle c    -> Math.PI * c.radius() * c.radius();
    case Rectangle r -> r.width() * r.height();
    case Triangle t  -> 0.5 * t.base() * t.height();
};
```

### Records Limitations

```java
// Cannot:
// - extend any class (implicitly extends Record)
// - have non-static, non-final fields
// - be abstract
// - declare mutable state

// Workaround for Optional fields
public record User(String name, String email, Optional<String> phone) { }
// Better: use @Nullable annotation or overloaded factory methods

// Mutable collection in record — still mutable!
public record Playlist(String name, List<String> tracks) {
    public Playlist {
        tracks = List.copyOf(tracks);  // defensive copy in compact constructor
    }
}
```

---

## 10. Annotations

Annotations are metadata attached to code elements — processed at compile time or runtime via reflection.

### Built-in Annotations

```java
@Override           // Compiler verifies you're actually overriding — always use!
@Deprecated         // Marks API as deprecated — use @deprecated in Javadoc for reason
@SuppressWarnings("unchecked")  // Suppress compiler warnings
@FunctionalInterface // Compiler verifies interface has exactly one abstract method
@SafeVarargs        // Suppresses heap pollution warnings on varargs methods

// Java 9+
@Deprecated(since = "3.0", forRemoval = true)  // Enriched deprecation
```

### Custom Annotation Definition

```java
import java.lang.annotation.*;

// Meta-annotations define annotation behavior
@Target({ElementType.METHOD, ElementType.TYPE})   // Where it can be applied
@Retention(RetentionPolicy.RUNTIME)               // How long it survives
@Documented                                        // Included in Javadoc
@Inherited                                         // Subclasses inherit it
public @interface Audited {
    String author() default "unknown";
    String date();
    AuditLevel level() default AuditLevel.INFO;
}

public enum AuditLevel { INFO, WARN, CRITICAL }

// Usage
@Audited(author = "alice", date = "2024-01-15", level = AuditLevel.CRITICAL)
public void transferFunds(Account from, Account to, BigDecimal amount) { ... }
```

### Retention Policies

```
RetentionPolicy.SOURCE   — exists in source only; discarded by compiler
                           Examples: @Override, @SuppressWarnings

RetentionPolicy.CLASS    — stored in .class but not loaded into JVM (default)
                           Examples: bytecode manipulation tools (ASM, ByteBuddy)

RetentionPolicy.RUNTIME  — available via reflection at runtime
                           Examples: Spring @Component, JPA @Entity, JUnit @Test
```

### Runtime Annotation Processing via Reflection

```java
@Audited(author = "bob", date = "2024-03-01")
public class PaymentService {

    @Audited(author = "alice", date = "2024-03-15", level = AuditLevel.CRITICAL)
    public void processPayment() { }
}

// Reading annotations at runtime
Class<?> clazz = PaymentService.class;

// Class-level annotation
Audited classAnnotation = clazz.getAnnotation(Audited.class);
System.out.println(classAnnotation.author());  // "bob"

// Method-level annotation
Method method = clazz.getMethod("processPayment");
Audited methodAnnotation = method.getAnnotation(Audited.class);
System.out.println(methodAnnotation.level());  // CRITICAL

// Discover all annotated methods
Arrays.stream(clazz.getDeclaredMethods())
    .filter(m -> m.isAnnotationPresent(Audited.class))
    .forEach(m -> System.out.println("Audited: " + m.getName()));
```

### Compile-Time Annotation Processing

```java
// javax.annotation.processing.AbstractProcessor
// Used by: Lombok, MapStruct, Dagger — generate code at compile time
// Registered via META-INF/services/javax.annotation.processing.Processor
// Key benefit: zero runtime overhead — all processing at build time
```

---

## 11. Packages

```java
// Package declaration — must be first statement in file
package com.company.product.module;

// Single import
import java.util.List;

// Static import — import static members
import static java.lang.Math.PI;
import static java.util.Collections.emptyList;

// Wildcard import — imports all public types (not sub-packages)
import java.util.*;
// Note: wildcard does NOT import sub-packages — java.util.* doesn't include java.util.concurrent.*

// Fully qualified name — no import needed
java.util.List<String> list = new java.util.ArrayList<>();
```

### Package Conventions

```
com.company.product          — root
com.company.product.domain   — domain entities, value objects
com.company.product.service  — business logic
com.company.product.repo     — data access
com.company.product.api      — REST controllers
com.company.product.config   — Spring/framework config
com.company.product.util     — utilities
com.company.product.exception— custom exceptions
```

### Java Modules (Java 9+) — Module System

```java
// module-info.java — module descriptor (at src/main/java root)
module com.company.product {
    requires java.sql;                          // module dependency
    requires transitive java.logging;           // transitive: consumers also get this
    requires static java.annotation;            // optional at runtime

    exports com.company.product.api;            // public API
    exports com.company.product.domain to       // restricted export
        com.company.product.service;

    opens com.company.product.config to         // reflection access (e.g., Spring)
        spring.core;

    provides com.company.product.spi.Plugin     // service provider
        with com.company.product.impl.DefaultPlugin;

    uses com.company.product.spi.Plugin;        // service consumer
}
```

**Senior note:** The module system enforces strong encapsulation — `public` is no longer universally accessible; a class must be in an `exports` clause. This breaks many pre-Java 9 reflection-heavy frameworks. Understanding `--add-opens` and `--add-exports` JVM flags is important for migration.

---

## 12. Class Loading Basics

### Class Loader Hierarchy

```
Bootstrap ClassLoader       — loads rt.jar/JDK core classes (java.lang, java.util)
        │                     Written in native C/C++, not Java
        ▼
Platform ClassLoader        — loads JDK extension modules (Java 9+)
(Extension in Java 8)         Previously loaded from $JAVA_HOME/lib/ext
        │
        ▼
Application ClassLoader     — loads classpath (your application classes)
        │
        ▼
Custom ClassLoaders         — frameworks (OSGi, Spring Boot, web containers)
                              Load from jars-in-jars, databases, network
```

### Delegation Model

```java
// Parent Delegation: when loading class X
// 1. Application ClassLoader asks Platform ClassLoader
// 2. Platform asks Bootstrap
// 3. Bootstrap tries to load — if found, done
// 4. If not, Platform tries — if found, done
// 5. If not, Application ClassLoader loads it
// 6. If not found, ClassNotFoundException

// This prevents malicious override of core classes
// You can't create java.lang.String that replaces the real one

// Programmatic class loading
Class<?> clazz = Class.forName("com.example.MyClass");          // uses current context loader
Class<?> clazz2 = Class.forName("com.example.MyClass", true,   // initialize?
    Thread.currentThread().getContextClassLoader());             // explicit loader

// Instantiate loaded class
Object instance = clazz.getDeclaredConstructor().newInstance();
```

### Class Loading Phases

```
LOADING
  → Find the .class file (bytecode)
  → Create Class object in method area (Metaspace)

LINKING
  ├── Verification  — verify bytecode is valid JVM bytecode
  ├── Preparation   — allocate memory for static fields, set to defaults
  └── Resolution    — resolve symbolic references to actual references

INITIALIZATION
  → Run static initializers in order
  → Happens when class is first actively used
  → Thread-safe — guaranteed by JVM spec (used in initialization-on-demand holder pattern)
```

```java
// Initialization-on-demand holder pattern — thread-safe lazy singleton
public class Singleton {
    private Singleton() { }

    // Inner class not loaded until getInstance() is called
    // JVM guarantees class initialization is thread-safe
    private static class Holder {
        private static final Singleton INSTANCE = new Singleton();
    }

    public static Singleton getInstance() {
        return Holder.INSTANCE;
    }
}
```

### Class Loading in Frameworks

```java
// Spring uses custom ClassLoader for:
// - Hot reload in dev mode (Spring DevTools)
// - Loading beans from child application contexts

// Thread context ClassLoader — important for library code
ClassLoader contextLoader = Thread.currentThread().getContextClassLoader();
Thread.currentThread().setContextClassLoader(customLoader);
// Restore after use!

// Class unloading — happens when ClassLoader becomes eligible for GC
// Metaspace can fill up in dynamic environments (app servers, OSGi)
// -XX:MaxMetaspaceSize — limit Metaspace to detect leaks early
```

---

## 13. Interview Questions with Expert Answers

**Q1: What methods does `Object` provide and why are `equals`/`hashCode` so important?**
> `Object` provides `equals`, `hashCode`, `toString`, `getClass`, `clone`, `finalize` (deprecated), and the monitor methods `wait`/`notify`/`notifyAll`. `equals`/`hashCode` are critical because the entire hash-based collections framework (`HashMap`, `HashSet`, `LinkedHashMap`) depends on their contract: if `a.equals(b)`, then `a.hashCode()` must equal `b.hashCode()`. Violating this causes objects to be silently unfindable in collections — `contains()` returns false, `get()` returns null, even though the object "is" in the collection. This is one of the most common sources of subtle production bugs.

---

**Q2: What is the correct way to use `Optional`? What are the anti-patterns?**
> `Optional` should be used exclusively as a return type when a method may return no result, replacing the contract ambiguity of returning `null`. Anti-patterns: using `Optional` as a method parameter (forces callers to wrap values), as a class field (not serializable, heap overhead), in collections (pointless), and calling `.get()` without checking `.isPresent()` (defeats the purpose). The `orElseGet()` vs `orElse()` distinction is important — `orElse(expr)` always evaluates `expr` even if the value is present; `orElseGet(() -> expr)` is lazy and only evaluates when needed.

---

**Q3: Why should you avoid `ordinal()` in enums?**
> `ordinal()` returns the position of a constant in the declaration order (0-based). If you persist or use this value for logic and later add a constant in the middle of the declaration, all ordinals shift — silently corrupting stored data and breaking switch logic. Instead, use a dedicated field with an explicit, stable value. For persistence in JPA/Hibernate, use `@Enumerated(EnumType.STRING)` rather than `EnumType.ORDINAL` (the default) for the same reason.

---

**Q4: What are records, and how do they differ from regular classes?**
> Records are immutable data carriers introduced in Java 16. The compiler automatically generates the canonical constructor, accessors (named after the component, not `getX`), and correct `equals`/`hashCode`/`toString` using all components. Records implicitly extend `java.lang.Record`, so they cannot extend any other class. All fields are final. They are ideal for DTOs, value objects, and command/event objects. Key distinction: records generate `equals`/`hashCode` over all fields — if you need partial equality (by ID only), use a regular class with manual implementation.

---

**Q5: Explain `java.time` vs the legacy date API. Why was the legacy API problematic?**
> The legacy API (`java.util.Date`, `Calendar`) had multiple design flaws: months were 0-indexed in `Calendar` but 1-indexed in some `Date` methods, `Date` despite its name represented a point in time not a calendar date, `SimpleDateFormat` was not thread-safe causing data corruption when shared, and `Date` was mutable making it unsafe to return from APIs. `java.time` (inspired by Joda-Time) is immutable, thread-safe, clearly separated (`LocalDate` for dates, `Instant` for machine timestamps, `ZonedDateTime` for timezone-aware values), and has a fluent, consistent API. `DateTimeFormatter` is thread-safe and can be stored as a constant.

---

**Q6: What is the difference between `@Retention(SOURCE)`, `CLASS`, and `RUNTIME`?**
> `SOURCE` annotations are discarded by the compiler — used for compile-time checks like `@Override` and `@SuppressWarnings`. `CLASS` annotations are stored in the `.class` file but not loaded into the JVM — used by bytecode manipulation tools like ASM or ByteBuddy. `RUNTIME` annotations survive into the running JVM and can be read via reflection — this is what Spring (`@Component`, `@Autowired`), JPA (`@Entity`), and JUnit (`@Test`) use. Processing `RUNTIME` annotations via reflection has overhead; compile-time annotation processors (Lombok, MapStruct) use `SOURCE`/`CLASS` to do their work at build time with zero runtime cost.

---

**Q7: How does Java's class loading delegation model work and why does it exist?**
> When a ClassLoader is asked to load a class, it first delegates to its parent. This continues up to the Bootstrap ClassLoader. Only if the parent cannot find the class does the child attempt to load it. This model exists for security and consistency: it ensures that core Java classes (`java.lang.String`, `java.util.List`) are always loaded by the Bootstrap ClassLoader — you cannot inject a malicious replacement. It also ensures that a class loaded by different loaders is not accidentally treated as the same type (though this can still happen in custom loader scenarios, causing `ClassCastException` even between seemingly identical classes).

---

**Q8: What is the `Math.addExact` family of methods and when should you use them?**
> `Math.addExact`, `subtractExact`, `multiplyExact`, `incrementExact`, `decrementExact`, and `negateExact` (Java 8+) perform arithmetic and throw `ArithmeticException` on integer overflow rather than silently wrapping. Standard Java integer arithmetic silently overflows — `Integer.MAX_VALUE + 1 == Integer.MIN_VALUE`. In financial calculations, order processing, or any domain where overflow is a defect rather than intended behavior, these methods catch the bug immediately. For large numbers, use `BigDecimal` (financial) or `BigInteger` (arbitrary precision integers).

---

**Q9: What is the difference between `UUID.randomUUID()` and `UUID.nameUUIDFromBytes()`?**
> `randomUUID()` generates a Version 4 UUID — 122 bits of cryptographically strong random data from `SecureRandom`. Different call → different UUID. `nameUUIDFromBytes()` generates a Version 3 UUID using MD5 hashing — same input always produces the same UUID. Version 3 is useful for deterministic IDs: generating a stable UUID for a known entity (e.g., converting a legacy string ID to UUID format reproducibly). Version 5 is similar but uses SHA-1 — prefer Version 5 over Version 3 for new code. For distributed systems primary keys, random UUIDs avoid coordination but cause B-tree index fragmentation — consider ULID or time-ordered UUIDs (Version 7, Java doesn't have built-in support yet) for database performance.

---

**Q10: How do annotations work in frameworks like Spring?**
> Spring uses `RUNTIME` retention annotations and reads them via reflection during application context startup. For `@Component`-annotated classes, Spring's `ClassPathScanningCandidateComponentProvider` scans the classpath, finds classes with the annotation using reflection, instantiates them, and registers them as beans. For `@Autowired`, it uses reflection to find fields/constructors/methods needing injection and sets them (including private fields via `field.setAccessible(true)`). Spring AOP creates dynamic proxies (JDK proxy or CGLIB) around annotated beans. Since Java 9 modules restrict reflective access, Spring requires `opens` clauses in `module-info.java` to maintain this behavior.

---

**Q11: When would you use `EnumSet` and `EnumMap` over regular `HashSet`/`HashMap`?**
> Always prefer them when the keys are enum values. `EnumSet` is implemented as a bit vector — a single `long` can represent all constants of an enum with up to 64 values, making operations like `contains`, `add`, `remove` O(1) with no hashing and no object allocation. `EnumMap` uses an array indexed by ordinal — same O(1) performance with minimal memory overhead and no hash collision handling. Both are also faster to iterate than their hash counterparts. The only cost is slightly more restricted type — the key type must be a specific enum.

---

**Q12: What is the `wait`/`notify` contract, and what is a spurious wakeup?**
> `wait()` must be called inside a `synchronized` block on the object being waited on — it atomically releases the lock and suspends the thread. `notify()` wakes one waiting thread; `notifyAll()` wakes all. A spurious wakeup is when a thread wakes from `wait()` without being notified — this is allowed by the JVM spec (and happens on some OS/hardware implementations). Therefore `wait()` must always be called inside a `while` loop checking the condition, not an `if` statement. Today, `java.util.concurrent` (`ReentrantLock`, `Condition`, `BlockingQueue`) is preferred over raw `wait`/`notify` for clearer, safer concurrency.

---

**Q13: Can records have custom `equals`/`hashCode`? What are the implications?**
> Yes — you can override `equals` and `hashCode` in a record. The default implementation compares all components. If you override to compare only a subset, you break the record's value semantics — callers expect records to be equal when all fields are equal. If you need identity-based or partial equality, a regular class with explicit `equals`/`hashCode` is more appropriate. One legitimate use case: overriding `equals` when a component type has a broken `equals` (rare), or adding tolerance for floating-point component comparison.

---

## 14. Advanced / Senior-Level Considerations

### Custom ClassLoader Use Cases

```java
// Implement custom ClassLoader for:
// - Loading classes from database or network
// - Hot-reloading without restarting JVM (dev tools, OSGi)
// - Bytecode transformation (AOP weaving, instrumentation agents)
// - Sandboxing (load untrusted code in isolated loader)

public class HotReloadClassLoader extends ClassLoader {
    private final Path classesDir;

    public HotReloadClassLoader(Path classesDir, ClassLoader parent) {
        super(parent);
        this.classesDir = classesDir;
    }

    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        Path classFile = classesDir.resolve(name.replace('.', '/') + ".class");
        try {
            byte[] bytes = Files.readAllBytes(classFile);
            return defineClass(name, bytes, 0, bytes.length);
        } catch (IOException e) {
            throw new ClassNotFoundException(name, e);
        }
    }
}
```

### Java Time in Distributed Systems

```java
// Always store timestamps as Instant or epoch millis — timezone-neutral
// Convert to ZonedDateTime at the API boundary for display
// Never store LocalDateTime as a timestamp — no timezone info = ambiguous at DST changes

// Comparing across timezones
Instant i1 = ZonedDateTime.of(2024, 3, 15, 14, 0, 0, 0,
                               ZoneId.of("America/New_York")).toInstant();
Instant i2 = ZonedDateTime.of(2024, 3, 15, 19, 0, 0, 0,
                               ZoneId.of("UTC")).toInstant();
System.out.println(i1.equals(i2));   // true — same moment in time

// Clock abstraction for testability
public class OrderService {
    private final Clock clock;

    public OrderService(Clock clock) { this.clock = clock; }

    public Order createOrder() {
        return new Order(Instant.now(clock));  // injectable clock
    }
}
// In tests: Clock.fixed(instant, ZoneId.of("UTC"))
// In production: Clock.systemUTC()
```

### Annotation Processing at Scale

```java
// Compile-time processors (zero runtime cost):
// Lombok: @Data, @Builder, @Value → generates boilerplate
// MapStruct: @Mapper → generates type-safe mapper implementations
// Dagger: @Component → generates DI code
// These use AbstractProcessor and generate source files at compile time
// Prefer compile-time over runtime annotation processing in performance-critical paths

// Java 21 — annotation on patterns (preview)
// Moving toward more expressive compile-time contracts
```

---

## 15. Summary / Cheat Sheet

```
OBJECT METHODS:
  equals + hashCode — ALWAYS override together; broken hashCode = invisible bugs in HashMap/HashSet
  toString          — override for debuggability; exclude sensitive data
  clone()           — avoid; prefer copy constructors
  wait/notify       — always in synchronized; wait in while loop (spurious wakeups)

STRING CLASSES:
  String       — immutable, thread-safe, pool for literals
  StringBuilder — mutable, fast, single-threaded
  StringBuffer  — mutable, thread-safe (synchronized), rarely needed

MATH:
  Math.addExact() → throws on overflow (use in financial/critical code)
  Math.random()   → NOT cryptographically secure → use SecureRandom

DATE/TIME:
  NEVER use java.util.Date/Calendar in new code
  LocalDate  — date only        | LocalTime — time only
  LocalDateTime — no timezone   | ZonedDateTime — with timezone
  Instant    — machine timestamp (epoch) — use for persistence
  DateTimeFormatter — thread-safe (unlike SimpleDateFormat)

OPTIONAL:
  Return type only — NOT parameter, NOT field, NOT in collections
  orElseGet() is lazy, orElse() always evaluates
  Never call .get() without isPresent() or orElseThrow()

ENUM:
  Full class — can have fields, methods, abstract methods
  NEVER use ordinal() for logic or persistence
  EnumSet/EnumMap → always prefer over HashSet/HashMap for enum keys

RECORDS:
  Immutable data carriers (Java 16+)
  Auto: canonical constructor, equals/hashCode (ALL fields), toString, accessors
  Cannot extend classes, no mutable fields
  Compact constructor for validation/normalization

ANNOTATIONS:
  SOURCE → compile only (@Override)
  CLASS  → bytecode tools
  RUNTIME → reflection (Spring, JPA, JUnit)
  @Target, @Retention, @Documented, @Inherited are meta-annotations

CLASS LOADING:
  Bootstrap → Platform → Application → Custom (delegation: parent first)
  Three phases: Loading → Linking (verify/prepare/resolve) → Initialization
  Static initializers run during Initialization — JVM guarantees thread safety
  Initialization-on-demand holder → idiomatic thread-safe lazy singleton
```


