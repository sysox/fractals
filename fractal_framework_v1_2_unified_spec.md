# Fractal Framework — v1.2 Unified Spec

## Goal
To define a minimal, truly unified, and extensible framework for all fractal and self-similar systems (IFS, L-Systems, Escape-time, Stochastic) by treating them as a single iterative evolution of heterogeneous state segments.

---

## 1. Core Principle
\[
S_{t+1} = Transform(Split(S_t))
\]
All fractal systems follow this top-level law.

---

## 2. State Model
The **State** ($S$) is a collection of **Carrier Segments**.

### Carrier Segment
A segment is a contiguous block of evolution data with a specific structure:
- `carrier_kind`: { set | sequence | grid }
- `units`: The actual elements (Objects or Samples)
- `semantics`: Forward reference: defines `split_kind` and `aggregate_kind` (see Section 6)
- `iteration`: Current local step count
- `mode`: { set | sequence | evaluation }
- `domain`: { fixed | dynamic }

### Units (The Object/Sample Bridge)
- **Objects:** Heavyweight entities with type, visibility, and attributes (used in `set` and `sequence`).
- **Samples:** Lightweight point-states (used in `grid` to prevent memory explosion).

---

## 3. The Latent Registry (Shared Blackboard)
The Latent Registry is the "glue" that allows different fractal types to interact.
- **Global Visibility:** All segments can read from and write to the same latent pool.
- **Hybridization:** An Evaluation-mode grid can write a density field that a Sequence-mode L-system queries to decide its branching factor.

### Latent API
- `UpdateLatent(state, previous_latent, context) -> latent`
- `Sample(latent_name, query, context) -> value`
- `QueryLatent(latent_type, query, context) -> result`
- `WriteLatent(latent_name, value, write_policy, context)`
  - *Default write_policy:* `last-write-wins`. Richer conflict resolution is deferred to design debt.

---

## 4. Context
Context is strictly split into two scopes to ensure interoperability.

### LocalContext (Unit-specific)
- `unit`: The current element.
- `parent`: The generator of this unit.
- `depth`: Recursion level.
- `index`: Position in sequence (optional).
- `neighbors`: Adjacent units (optional, for context-sensitive rules).
- `local_to_global`: Transform matrix for coordinate space mapping.

### GlobalContext (Environment-specific)
- `state`: The full system state.
- `latent`: The Shared Blackboard.
- `summary`: **The summary available at step t is the summary computed at the end of step t−1.**
- `rng`: Random number generator.

---

## 5. The Normative Pipeline (The Engine Heartbeat)

Step(state $S_t$):

1. **Derive Execution Contexts** (Local and Global).
2. **Execute Split:** Generate new units/segments based on $S_t$.
   - **Note:** Segments spawned by `Split` in step $t$ are merged after `Transform` completes; they participate in evolution from step $t+1$.
3. **Execute Transform:** Apply transformation logic to the *surviving* units of $S_t$ based on their mode.
   - **Set Mode:** Parallel `map`.
   - **Sequence Mode:** Sequential `fold`. All rewrites are computed before the sequence segment's `Transform` begins.
   - **Evaluation Mode:** In-place `update`.
4. **Apply Pruning:**
   - `Prune(carrier, context) = { u in carrier | Keep(u, context) = true }`
   - **Lifecycle:** When all units in a segment are pruned, the segment itself is removed.
5. **Merge:** Combine surviving transformed units and newly spawned segments.
6. **Update Latents:** Execute `UpdateLatent` to refresh the Shared Blackboard.
7. **Compute Summary:** Global statistics for step $t$.
8. **Return $S_{t+1}$**.

---

## 6. Segment Semantics Object
Defines how a segment behaves during the pipeline.
- `split_kind`: { replicate | subdivide | identity }
- `aggregate_kind`: { union | chain | update }
- `transform_kind`: { map | fold | in-place }

---

## 7. Parameters & Evaluators
All parameters are functions of context:
`Param(context) -> value`

This allows for global feedback:
`SplitCount(context) -> f(context.unit, context.global_summary)`

---

## 8. Universal Coordinate Space
To ensure hybridization works:
- All latent field queries operate in **Global State-Space** coordinates unless explicitly overridden by a `LocalContext` transform.
- Each `LocalContext` provides a mapping from internal unit space to world space.

---

## 9. Canonical Hybrid Mappings

### "Analytic-Controlled L-System"
- **Segment A:** `Grid` (Evaluation mode). Calculates Mandelbrot escape time and writes it to a latent "GrowthField."
- **Segment B:** `Sequence` (Sequence mode). L-system queries "GrowthField" to determine `Scale` and `SplitCount`.

### "IFS-Generated Texture"
- **Segment A:** `Set` (Set mode). IFS geometry generation.
- **Segment B:** `Grid` (Evaluation mode). Queries IFS geometry density to color pixels.

---

## 10. Open Design Debt
- **Topology/Connectivity:** Tracking shared vertices/edges across segments.
- **Async Execution:** Handling different iteration speeds for different segments.
- **Advanced Write Policies:** Averaging, additive, or stochastic latent writes.

---

## Core Statement
A fractal is a unified iterative evolution of heterogeneous carrier segments, communicating through a shared latent registry and executing through a polymorphic Split/Transform pipeline.
