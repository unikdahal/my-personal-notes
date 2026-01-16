---
title: "Java Reflection: Internals, Performance & Security"
day: 10
tags:
  - java
  - reflection
  - jvm-internals
  - spring
  - sde2
---

# 10. Java Reflection (Deep Internals)

## 10.1 First Principles: Why Reflection Exists
- Normal Java uses static binding
- Classes, methods, fields usually known at compile time
- Framework problem:
  - Types unknown until runtime
  - Behavior driven by configuration & annotations
- Reflection enables:
  - Runtime inspection
  - Runtime invocation
  - Dynamic object construction

## 10.2 Definition of Reflection
- Ability of a program to inspect and manipulate its own structure at runtime
- Operates on:
  - Classes
  - Fields
  - Methods
  - Constructors
  - Annotations
- Turns program structure into runtime metadata

## 10.3 Reflection and Class Loading
- Reflection operates on `Class<?>` objects
- `Class` object:
  - JVM runtime representation of a loaded class
  - Created during class loading
- One `Class` object per (ClassLoader + class name)
- Same class name loaded by different classloaders → different `Class` objects

## 10.4 Reflection Object Model
- From `Class<?>`, JVM exposes:
  - `Field`
  - `Method`
  - `Constructor`
  - `Annotation`
- These are descriptors, not actual code
- Class structure is immutable after loading

## 10.5 Access Control & Encapsulation
- Normal Java enforces access via compiler + JVM
- Reflection can bypass using:
  - `setAccessible(true)`
- Consequences:
  - Breaks encapsulation
  - Violates invariants
  - Introduces non-local bugs

## 10.6 Performance Costs of Reflection
- Dynamic dispatch prevents:
  - JIT inlining
  - Devirtualization
  - Escape analysis
- Additional overhead:
  - Boxing / unboxing
  - Varargs allocation
  - Access checks
- Reflection is always slower than direct calls

## 10.7 JIT Interaction
- JIT requires predictable call targets
- Reflection resolves targets at runtime
- JVM cannot optimize aggressively
- Result:
  - Higher latency
  - Lower throughput

## 10.8 Reflection Usage Pattern in Frameworks
- Reflect once, execute many times
- Typical flow:
  - Startup scanning
  - Metadata caching
  - Runtime execution via cached handles
- Reflection avoided in hot paths

## 10.9 Security Risks of Reflection
- Can access private data
- Can mutate internal state
- Common source of vulnerabilities:
  - Deserialization exploits
  - Sandbox escapes
- `setAccessible(true)` is a major risk point

## 10.10 Java 9+ Module System (JPMS)
- Introduced strong encapsulation
- Code organized into modules
- Modules explicitly control:
  - Exported packages
  - Opened packages (for reflection)

## 10.11 Reflection Restrictions in Java 9+
- Reflection across modules blocked by default
- `exports`:
  - Compile-time + runtime access
- `opens`:
  - Runtime reflective access only
- Illegal reflective access:
  - Warnings → errors in newer Java versions

## 10.12 Escape Hatches
- JVM flags:
  - `--add-opens`
  - `--illegal-access`
- Explicit opt-in to break encapsulation
- Responsibility shifts to developer

## 10.13 Reflection vs Alternatives
- Reflection:
  - Powerful
  - Slow
  - Dangerous
- Alternatives:
  - Interfaces & polymorphism
  - Code generation
  - MethodHandles / VarHandles

## 10.14 When Reflection Is Justified
- Infrastructure / framework code
- Startup-time operations
- Unknown types at compile time
- Controlled and audited usage

## 10.15 When Reflection Should Be Avoided
- Business logic
- Hot execution paths
- Security-sensitive code
- Frequent method invocation

## 10.16 Final Mental Model
- Reflection trades safety and performance for runtime flexibility
- Essential for frameworks, risky for applications
- Must be minimized, cached, and explicitly controlled

## 10.17 SDE-2 One-Paragraph Summary
- Java reflection enables runtime introspection and dynamic behavior required by frameworks such as Spring, but it bypasses compile-time safety, blocks JVM optimizations, and breaks encapsulation. Java 9 strengthened controls via the module system, restricting reflective access unless explicitly permitted. Reflection should be used sparingly, primarily during startup and infrastructure-level code, and avoided in hot paths and business logic.

