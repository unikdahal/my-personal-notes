---
title: "Java NIO Socket Programming — Non-Blocking I/O, Channels & Selectors"
part: 2
tags:
  - java
  - nio
  - non-blocking-io
  - selectors
  - channels
---

# 2. Non-Blocking I/O & Readiness-Based Coordination

## 2.1 Streams vs Channels: The Fundamental API Shift
- Classic I/O:
  - Stream-oriented
  - Directional (InputStream / OutputStream)
  - Blocking by default
- NIO:
  - Channel-oriented
  - Bidirectional
  - Can operate in blocking or non-blocking mode
- Design intent:
  - Streams model sequential consumption
  - Channels model controllable data transfer

## 2.2 Blocking vs Non-Blocking Semantics
- Blocking call:
  - Thread sleeps until operation can complete
  - Capacity tied to thread count
- Non-blocking call:
  - Operation returns immediately
  - Caller observes current system state
- Consequence:
  - Control flow becomes fragmented
  - Progress must be explicit

## 2.3 Non-Blocking Mode Contract
- Enabled via:
  - channel.configureBlocking(false)
- Guarantees:
  - No I/O call will block the calling thread
- New responsibility:
  - Caller must handle incomplete operations

## 2.4 `read()` Return Value Semantics (Critical)
- `> 0`:
  - Bytes were read
  - Progress made
- `0`:
  - No data available *right now*
  - Connection still open
  - Normal, non-exceptional state
- `-1`:
  - Peer closed the connection
  - Cleanup required
- Key rule:
  - `0` must never be treated as error

## 2.5 Why Returning `0` Is Essential
- Prevents thread from blocking on slow clients
- Allows one thread to serve many connections
- Enables fair progress across connections
- Eliminates thread-per-connection dependency

## 2.6 The Busy-Waiting Trap
- Anti-pattern:
  - Repeatedly calling `read()` until data arrives
- Consequences:
  - CPU spin
  - Cache thrashing
  - Starvation of other work
- Conclusion:
  - Non-blocking without coordination is useless

## 2.7 The Need for Readiness Notification
- Core requirement:
  - Efficiently wait until *any* socket is ready
- Constraints:
  - Must not poll per socket
  - Must not busy-wait
- Solution:
  - Delegate readiness detection to OS kernel

## 2.8 Selector: Central Coordination Mechanism
- Selector role:
  - Monitor many channels
  - Report readiness events
- Selector does NOT:
  - Perform I/O
  - Move bytes
- Selector DOES:
  - Block once on behalf of many channels

## 2.9 OS-Level Readiness Model
- Kernel tracks:
  - Socket receive buffer state
  - Socket send buffer state
- Readiness events:
  - READ: data available in receive buffer
  - WRITE: space available in send buffer
  - ACCEPT: new connection pending
- JVM acts as:
  - Thin wrapper over OS facilities

## 2.10 Safe Blocking in NIO
- Blocking is not eliminated
- Blocking is *relocated*
- Allowed blocking point:
  - selector.select()
- Forbidden blocking points:
  - channel.read()
  - channel.write()

## 2.11 Event Loop Pattern
- Canonical structure:
  - wait for readiness
  - process ready channels
  - repeat
- Properties:
  - Single blocking point
  - High connection fan-out
  - Deterministic scheduling

## 2.12 Interest Sets (`interestOps`)
- Channel registers interest in:
  - OP_ACCEPT
  - OP_READ
  - OP_WRITE
  - OP_CONNECT
- Registration expresses:
  - What events should wake the selector

## 2.13 Event-Driven vs State-Driven Readiness
- OP_READ:
  - Event-driven
  - Triggered by data arrival
- OP_WRITE:
  - State-driven
  - Triggered by buffer availability
- Consequence:
  - OP_WRITE is usually always ready

## 2.14 The OP_WRITE CPU Spin Hazard
- Always-registered OP_WRITE:
  - selector.select() returns immediately
  - Event loop spins
  - CPU usage spikes
- Correct rule:
  - Register OP_WRITE only when writes are incomplete

## 2.15 Selector as a Progress Engine
- Selectors do not handle requests
- They advance connection state
- Each readiness event:
  - Enables a non-blocking step

## 2.16 Summary of Part 2
- Channels replace streams to enable control
- Non-blocking I/O returns partial progress
- `read() == 0` is a normal state
- Busy-waiting is catastrophic
- Selector centralizes waiting efficiently
- Blocking is allowed only on coordination
- Readiness events drive system progress

