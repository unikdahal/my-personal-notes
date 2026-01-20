---

title: "Java Generics: Type Erasure & Variance"
day: 9
tags:

* java
* generics
* jvm-internals
* type-erasure

---

# 9. Java Generics — Internal Mechanics

## 9.1 First Principles: Why Generics Exist

* Pre–Java 5 collections accepted `Object`
* Type errors surfaced at **runtime** (`ClassCastException`)
* Compiler had no knowledge of element types
* Fundamental objective:

  * Shift type errors from runtime → compile time
  * Preserve runtime performance characteristics
  * Avoid modifying the JVM execution model

## 9.2 Primary Design Constraint: Backward Compatibility

* JVM and bytecode format predate generics
* Massive ecosystem of existing bytecode
* Hard requirement:

  * Old binaries must link and execute unchanged
* Consequence:

  * Generics implemented entirely at compile time

## 9.3 Type Erasure (Core Mechanism)

* Generic type parameters are removed during compilation
* Example transformation:

  * `List<String>` → `List`
* Runtime representation:

  * No `<T>`, `<String>`, or `<Integer>` metadata
* Compiler responsibilities:

  * Enforce generic constraints
  * Insert implicit casts at read sites

## 9.4 Why Type Safety Still Holds

* Compiler prevents insertion of incompatible types
* All unsafe casts are centralized at compile time
* Well-typed generic code:

  * Cannot introduce new runtime `ClassCastException`
* Explicit escape hatch:

  * Raw types reintroduce unsoundness

## 9.5 Consequences of Type Erasure

* No runtime generic constructs:

  * `new T()` ❌
  * `T.class` ❌
  * `instanceof List<String>` ❌
* Method signatures may collide after erasure
* Type parameters cannot appear in `catch` clauses

## 9.6 Invariance as the Default Safety Rule

* Even though `String` ⊂ `Object`:

  * `List<String>` ⊄ `List<Object>`
* Reason:

  * Prevents writes that violate heap soundness
* Design choice:

  * Invariance as the baseline

## 9.7 Variance via Wildcards

### 9.7.1 Covariance — `? extends T`

* Semantics:

  * Unknown subtype of `T`
* Safe operation:

  * Read values as `T`
* Unsafe operation:

  * Writing concrete subtypes
* Conceptual role:

  * Producer of values

### 9.7.2 Contravariance — `? super T`

* Semantics:

  * Unknown supertype of `T`
* Safe operation:

  * Write values of type `T`
* Unsafe operation:

  * Reading as `T` (only `Object` guaranteed)
* Conceptual role:

  * Consumer of values

### 9.7.3 PECS Heuristic

* Producer → `extends`
* Consumer → `super`

## 9.8 Arrays vs Generics: Fundamental Mismatch

### 9.8.1 Arrays

* Reified (runtime type preserved)
* Covariant
* Enforced by runtime checks

### 9.8.2 Generics

* Erased (no runtime type information)
* Invariant
* Enforced by compile-time checks

### 9.8.3 Why Generic Arrays Are Disallowed

* Arrays depend on runtime type verification
* Generics erase type metadata
* Combining both would permit silent heap corruption

## 9.9 Heap Pollution

* Definition:

  * Parameterized reference points to an incompatible runtime object
* Primary causes:

  * Raw types
  * Generic varargs
* Observable effect:

  * Delayed `ClassCastException`

## 9.10 Bridge Methods

### 9.10.1 The Underlying Problem

* Type erasure alters method signatures
* Overriding relationships may disappear at the bytecode level

### 9.10.2 Compiler Strategy

* Compiler generates **bridge methods**
* Properties:

  * Preserve polymorphism
  * Delegate to the real implementation
  * Marked synthetic

### 9.10.3 Secondary Effects

* Reflection APIs expose bridge methods
* Tooling and frameworks must filter via `isBridge()`

## 9.11 Common Misconceptions

* Generics have no runtime existence
* Wildcards influence compilation only
* JVM instructions operate on erased types
* Generic lambdas may be evaluated more than once

## 9.12 Stable Mental Model

* Generics = compile-time contracts
* Type erasure = compatibility tradeoff
* Wildcards = controlled variance
* Bridge methods = erased polymorphism repair

## 9.13 One-Paragraph Summary

* Java generics provide compile-time type safety while preserving the original JVM execution model by erasing type information after compilation. Invariance prevents unsound writes, while wildcards introduce controlled variance for safe abstraction. Arrays and generics differ due to reification versus erasure. Bridge methods are compiler-generated artifacts that maintain polymorphism after erasure. All observable limitations stem directly from these foundational design decisions.
