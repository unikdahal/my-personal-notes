---

title: "Java Generics: Type Erasure & Variance" day: 9 tags:

- java
- generics
- jvm-internals
- type-erasure



---

# 9. Java Generics (Deep Internals)

## 9.1 First Principles: Why Generics Exist

- Pre-Java 5 collections accepted `Object`
- Type errors surfaced at **runtime** (`ClassCastException`)
- Compiler had no knowledge of element types
- Goal of generics:
  - Shift errors from runtime → compile time
  - Enforce type safety without changing JVM

## 9.2 Design Constraint: Backward Compatibility

- JVM and bytecode pre-existed generics
- Billions of lines of legacy code
- Requirement:
  - Old bytecode must run with new generic code
- Resulting design choice:
  - **Compile-time only generics**

## 9.3 Type Erasure (Core Mechanism)

- Generic type information removed at compile time
- Example:
  - `List<String>` → `List`
- Runtime sees:
  - No `<T>`, `<String>`, `<Integer>`
- Compiler responsibilities:
  - Enforce type constraints
  - Insert implicit casts

## 9.4 How Type Safety Is Still Preserved

- Compiler prevents inserting wrong types
- Compiler inserts casts on read paths
- Well-typed generic code:
  - Cannot throw `ClassCastException`
- Exception:
  - Raw types reintroduce heap pollution

## 9.5 Consequences of Type Erasure

- No runtime generic operations:
  - `new T()` ❌
  - `T.class` ❌
  - `instanceof List<String>` ❌
- Method overloading conflicts after erasure
- Generics cannot be used in `catch` clauses

## 9.6 Invariance of Java Generics

- Even if `String extends Object`:
  - `List<String>` ❌ `List<Object>`
- Reason:
  - Prevent unsafe writes that break type safety
- Java chooses:
  - Invariance by default

## 9.7 Variance via Wildcards

### 9.7.1 Covariance – `? extends T`

- Meaning:
  - Unknown subtype of `T`
- Safe operations:
  - Read as `T`
- Unsafe operations:
  - Write any concrete subtype
- Use case:
  - Producer of values

### 9.7.2 Contravariance – `? super T`

- Meaning:
  - Unknown supertype of `T`
- Safe operations:
  - Write `T`
- Unsafe operations:
  - Read as `T` (only `Object` guaranteed)
- Use case:
  - Consumer of values

### 9.7.3 PECS Rule

- Producer → `extends`
- Consumer → `super`

## 9.8 Arrays vs Generics

### 9.8.1 Arrays

- Reified (runtime type known)
- Covariant
- Runtime type checks enforced

### 9.8.2 Generics

- Erased (no runtime type info)
- Invariant
- Compile-time safety only

### 9.8.3 Why Generic Arrays Are Forbidden

- Arrays rely on runtime checks
- Generics have no runtime type info
- Allowing generic arrays would cause silent heap pollution

## 9.9 Heap Pollution

- Definition:
  - Parameterized reference points to wrong runtime object
- Common cause:
  - Raw types
  - Generic varargs
- Result:
  - Runtime `ClassCastException`

## 9.10 Bridge Methods

### 9.10.1 The Problem

- Type erasure breaks method overriding
- JVM sees erased signatures

### 9.10.2 The Solution

- Compiler generates **bridge methods**
- Bridge methods:
  - Preserve polymorphism
  - Delegate to actual implementation
  - Are synthetic and invisible in source

### 9.10.3 Side Effects

- Reflection exposes bridge methods
- Frameworks must filter `isBridge()`

## 9.11 Key Interview Traps

- Generics do NOT exist at runtime
- Wildcards affect compile time only
- CAS / JVM does not know `<T>`
- `computeIfAbsent` lambdas may run multiple times

## 9.12 Final Mental Model

- Generics = compile-time contract
- Type erasure = compatibility tradeoff
- Wildcards = controlled variance
- Bridge methods = erased polymorphism repair

## 9.13 One-Paragraph Summary

- Java generics enforce compile-time type safety without changing the JVM by erasing type information after compilation. Invariance prevents unsafe writes, while wildcards (`extends` / `super`) introduce controlled variance. Arrays and generics differ fundamentally due to reification vs erasure. Bridge methods are compiler-generated to preserve polymorphism after erasure. All generic limitations stem directly from this design choice.

