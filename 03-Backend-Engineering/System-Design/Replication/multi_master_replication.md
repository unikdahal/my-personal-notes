---
title: "Multi-Master Replication: Architecture, Trade-offs & Internals"
tags:
  - distributed-systems
  - replication
  - databases
  - consistency
  - consensus
---

# 1. First Principles: Why Replication Exists
- Single-node systems fail in multiple dimensions
  - Availability: node crash = total outage
  - Scalability: write throughput bounded by one machine
  - Latency: distant clients suffer RTT penalties
- Replication introduces *multiple copies* of data to solve these
- Core tension introduced by replication
  - Consistency vs Availability vs Partition tolerance (CAP)

# 2. Replication Models: A Taxonomy
- Single-Leader (Primary–Replica)
  - One node accepts writes
  - Others follow
- Multi-Leader (Multi-Master)
  - Multiple nodes accept writes
  - Conflicts become inevitable
- Leaderless
  - Any node can accept reads/writes
  - Conflict resolution pushed to clients or background processes

# 3. Definition: What Is Multi-Master Replication
- Multiple nodes act as *authoritative writers*
- Each master
  - Accepts local writes
  - Replicates writes to peer masters
- System invariant
  - No single global write serialization point

# 4. The Core Problem Introduced by Multi-Master
- Concurrent writes to the *same logical data item*
- Network partitions make coordination impossible
- Resulting fundamental issue
  - Write–write conflicts

# 5. Conflict Scenarios (From First Principles)
- Same key updated concurrently in two regions
- Different fields of same record updated concurrently
- Delete vs update races
- Reinsert-after-delete anomalies

# 6. Consistency Guarantees in Multi-Master Systems
- Strong consistency is generally impossible without sacrificing availability
- Typical guarantees offered
  - Eventual consistency
  - Causal consistency (in advanced systems)
- System-level invariant
  - All replicas *eventually converge* given no new writes

# 7. Write Propagation Models
- Asynchronous replication
  - Writes acknowledged locally
  - Replicated in background
  - Lower latency, higher conflict probability
- Synchronous replication (limited use)
  - Requires quorum across masters
  - Often collapses into single-leader behavior

# 8. Conflict Detection: How Systems Know a Conflict Happened
- Version vectors
- Vector clocks
- Hybrid logical clocks (HLC)
- Causal histories embedded in metadata

# 9. Version Vectors (Internal Mechanics)
- Each master maintains a logical counter
- Write metadata
  - {node_id → counter}
- Comparison rules
  - Dominates: strictly newer
  - Concurrent: conflict

# 10. Conflict Resolution Strategies
- Last Write Wins (LWW)
  - Timestamp-based overwrite
  - Simple but lossy
- Application-level resolution
  - Custom merge logic
- Data-type-specific resolution
  - CRDTs

# 11. CRDTs in Multi-Master Systems
- Conflict-Free Replicated Data Types
- Mathematical guarantee
  - Commutative
  - Associative
  - Idempotent
- Examples
  - G-Counter, PN-Counter
  - OR-Set

# 12. Read Semantics
- Local reads
  - Fast
  - Possibly stale
- Quorum reads
  - Reduced staleness
  - Increased latency
- Read-your-writes consistency
  - Session stickiness

# 13. Failure Modes Unique to Multi-Master
- Network partitions
- Clock skew
- Replica divergence
- Write amplification storms

# 14. Split-Brain Is the Default State
- Unlike single-leader systems
- Multi-master *assumes* split-brain
- Design goal
  - Survive split-brain
  - Reconcile later

# 15. Topology Patterns
- Full mesh replication
  - Every master talks to every other
  - O(n²) replication cost
- Hub-and-spoke
  - Reduced connections
  - Central bottleneck
- Hierarchical multi-master
  - Regional masters

# 16. Write Amplification Analysis
- Each logical write
  - Replicated to N−1 masters
- Metadata growth
  - Version vectors grow with cluster size
- Practical limits
  - Often ≤ 5–7 masters

# 17. Schema Design Implications
- Immutable records preferred
- Append-only models reduce conflicts
- Field-level ownership patterns

# 18. Transaction Semantics
- Cross-master ACID is generally infeasible
- Local transactions only
- Global invariants must be *eventually enforced*

# 19. Secondary Index Challenges
- Index updates replicated separately
- Conflicts in index entries
- Often rebuilt asynchronously

# 20. Comparison: Multi-Master vs Single-Leader
- Write latency
  - Lower in multi-master (local writes)
- Operational complexity
  - Significantly higher
- Correctness burden
  - Shifted to application layer

# 21. When Multi-Master Is the Right Choice
- Geo-distributed writes required
- Offline-first applications
- High write availability requirements

# 22. When Multi-Master Is the Wrong Choice
- Financial ledgers
- Strong global invariants
- Simpler operational environments

# 23. Real-World Design Philosophy
- Multi-master trades simplicity for availability
- Conflicts are *not edge cases*
- They are a core execution path

# 24. Mental Model for Staff-Level Design
- Treat conflicts as data, not errors
- Design merges before designing writes
- Assume partitions, clock skew, retries

# 25. Summary Invariants
- Multi-master systems
  - Cannot avoid conflicts
  - Must converge
  - Must expose or resolve ambiguity
- Correctness emerges from *merge semantics*, not coordination

