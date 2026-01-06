# Artefact Specification Language

## Overview

**Artefact Specification Language (ASL)** is a state-centric declarative Domain Specific Language (DSL) for defining **immutable system artefacts** that compile directly into reproducible WASM or native binaries.

An artefact describes *what* a computation is, not *how* to execute it procedurally. Artefacts describe the **desired state** of a system.

ASL is designed to:

- Follow **data-centric programming** paradiam
- Represent system logic as **versioned, immutable specifications**
- Compile artefacts into **deterministic WASM or native binaries**
- Execute exclusively via **host-provided, versioned capabilities**
- Embed **ownership, governance, and audit metadata** by default
- Eliminate configuration and execution drift

> ASL is **not** a general-purpose programming language.
> All side effects are explicitly mediated by the host runtime.

Think of ASL as a **deterministic logic layer** for business, operational, and infrastructure workflows.

---

## Philosophy

ASL is a **logic forge**, not a programming language.

It exists to make system behavior:

- Explicit
- Deterministic
- Governable
- Reproducible

Where general-purpose languages optimize for flexibility,
ASL optimizes for **correctness, auditability, and trust**.

---

## Artefact Structure

Each artefact is defined in a single YAML file.

```yaml
artefact: transform_invoice
owner: finance_team
maintainers:
  - alice@example.com
  - bob@example.com
version: 1.0.0

inputs:
  order: artefact:schema_order

outputs:
  total_tax: decimal

imports:
  - func: host.math.calculate_tax(amount: decimal, rate: decimal) -> decimal

implementation:
  total_tax: host.math.calculate_tax(order.total, 0.20)
```

### Core Fields

| Field            | Description                                           |
| ---------------- | ----------------------------------------------------- |
| `artefact`       | Globally unique artefact identifier                   |
| `owner`          | Owning team or organization                           |
| `maintainers`    | Responsible individuals                               |
| `version`        | Semantic version of the artefact                      |
| `inputs`         | Named inputs with explicit types                      |
| `outputs`        | Named outputs with explicit types                     |
| `imports`        | Host-provided deterministic functions                 |
| `implementation` | Pure expressions composed from primitives and imports |

Artefacts are **content-addressed**, immutable, and independently compilable.

---

## Type System & Primitives

ASL provides a small, deterministic type system:

* **Int** — signed 64-bit integer
* **Decimal** — 128-bit fixed-point (scale = 10⁹)
* **Bool** — boolean
* **String** — interned string reference
* **Timestamp** — `i64` milliseconds since epoch
* **List<T>** — homogeneous ordered collection
* **Map<K,V>** — key-sorted map with deterministic iteration
* **Optional<T>** — nullable value
* **Struct** — named collection of typed fields

No implicit coercions. All types are resolved at compile time.

---

## Host Functions

Artefacts interact with the outside world **only** through imported host functions.

```yaml
imports:
  - func: host.utils.log_info(msg: string)
  - func: host.math.calculate_tax(amount: decimal, rate: decimal) -> decimal

implementation:
  tax: host.math.calculate_tax(order.total, 0.20)
  _log: host.utils.log_info("Tax calculated")
```

### Host Function Constraints

* Versioned and explicitly declared
* Deterministic by contract
* No ambient I/O, global state, or hidden side effects
* Provided by the runtime, not the artefact

This separation ensures artefacts remain **pure, auditable, and reproducible**.

---

## Compilation Pipeline

ASL uses a fixed, fully deterministic compilation pipeline:

1. **Parse** — YAML → untyped AST
2. **Typecheck** — semantic validation → typed DAG
3. **Lowering** — typed DAG → ephemeral IR
4. **Codegen** — LLVM IR emission
5. **Optimize** — fixed, ordered LLVM passes
6. **Emit** — WASM or native binary
7. **Cache** — `.asl-cache/<artefact_hash>.wasm`

```text
ASL Source
   ↓
Typed DAG
   ↓
Ephemeral IR
   ↓
LLVM IR
   ↓
Deterministic Binary
```

---

## Determinism Guarantees

ASL enforces determinism by construction:

- Lexicographically sorted DAG nodes
- Deterministic map iteration (key-sorted)
- Fixed-point decimal arithmetic
- Interned strings for memory stability
- No ambient randomness or clocks
- Embedded debug metadata (`.asl_debug`)
- **Same artefact + compiler + target → identical binary**

This makes ASL artefacts suitable for **caching, verification, and formal audit**.

---

## Debugging & Introspection

### Compile-Time

```bash
asl-compile --emit-ir artefacts/transform_invoice.aml
```

- Inspect ephemeral IR
- Validate type resolution and DAG structure

### Runtime Errors

```text
Error: Constraint violation
Artefact: transform_invoice
Line: 22
Condition: order.total >= 0
```

- Errors reference artefact, version, and source line
- WASM offsets map back to source via `.asl_debug`

---

## Developer Workflow

```bash
# Create an artefact
vim artefacts/transform_invoice.aml

# Compile to WASM
asl-compile \
  --artefact artefacts/transform_invoice.aml \
  --target wasm32-unknown-unknown

# Execute artefact
asl-run dist/transform_invoice.wasm --input order.json

# Inspect binary and metadata
asl-inspect dist/transform_invoice.wasm
```

### Key Properties

- Deterministic rebuilds, every time
- Ownership and version metadata visible at runtime
- Host capabilities evolve independently of artefacts
- No recompilation needed for new UDFs

---

## Testing Strategy

- **Golden tests**: artefact → expected WASM hash
- **Property tests**: decimals, maps, DAG ordering
- **Differential tests**: compare against reference LLVM IR
- **Replay tests**: identical inputs → identical outputs

---

## Roadmap

- Conditional expressions and derived fields
- Filters and comprehensions
- Artefact composition
- Metadata registry integration
- IDE tooling (linting, highlighting)
- Optional CUE-inspired constraint semantics

---
