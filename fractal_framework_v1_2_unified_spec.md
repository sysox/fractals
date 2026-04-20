# Fractal Framework — v1.2 Frozen Spec

## Goal
Define a minimal, unified, and extensible framework for fractal and self-similar generative systems by treating them as a single iterative evolution of heterogeneous state.

The framework should support:
- recursive geometric fractals
- stochastic fractals
- L-systems and turtle systems
- escape-time and other evaluation fractals
- cellular and field-driven systems
- hybrid systems combining symbolic, geometric, sampled, and latent structure

---

## 1. Core Principle

\[
S_{t+1} = Transform(Split(S_t))
\]

This is the conceptual top-level law.

All systems in the framework are expressed as iterative evolution of explicit state under this law.

---

## 2. State Model

The global state is a collection of carrier segments plus shared latent and summary data.

```text
State:
- segments
- iteration
- latent
- summary
```

Where:
- `segments` = explicit evolving support
- `iteration` = global step index
- `latent` = shared non-rendered but queryable structures
- `summary` = compact global information computed from the previous completed step

### Normative rule
The summary available during step `t` is the summary computed at the end of step `t-1`.

---

## 3. Carrier Segments

A segment is a coherent block of evolution data with its own carrier structure and execution semantics.

```text
CarrierSegment:
- id
- carrier_kind
- units
- semantics
- iteration?
- domain
- attrs
```

Where:
- `id` = segment identity
- `carrier_kind` ∈ `{set, sequence, grid}`
- `units` = the actual evolving elements
- `semantics` = execution semantics object
- `iteration?` = optional local step count
- `domain` ∈ `{fixed, dynamic}`
- `attrs` = optional metadata

### Carrier kinds
- `set` = unordered or weakly ordered units
- `sequence` = ordered units with position-dependent meaning
- `grid` = sampled lattice or sampled field support

### Domain kinds
- `fixed` = support does not grow structurally
- `dynamic` = support may be generated, rewritten, expanded, or pruned

### Time model
The framework advances by a global state iteration `t`.

A segment may additionally maintain a local iteration counter.
By default, segment-local iteration increments once per global step unless a future scheduling model defines otherwise.

---

## 4. Units

The framework uses one unified unit abstraction.

```text
Unit:
- kind
- type
- visibility
- payload
- attrs
```

Where:
- `kind` ∈ `{object, sample}`
- `type` = semantic kind
- `visibility` ∈ `{visible, latent}`
- `payload` = geometry / symbol / value / local state
- `attrs` = derived or control attributes

### Interpretation
- `object` is typically used in `set` and `sequence`
- `sample` is typically used in `grid`

This preserves conceptual unification while allowing lightweight sampled representations where needed.

---

## 5. Latent Registry

The latent registry is the shared blackboard that enables interoperability between segments.

```text
LatentRegistry:
- named_fields
- named_indices
- named_guides
- named_structures
- persistent
- recomputed
- ephemeral
```

### Normative rule
All segments may read from and write to the same latent registry.

This is the main mechanism by which different generative paradigms interact.

### Example
A grid segment may write a density field to latent state, and a sequence segment may query that field to determine branching or scaling.

---

## 6. Latent API

```text
UpdateLatent(state, previous_latent, context) -> latent
Sample(latent_name, query, context) -> value
QueryLatent(latent_type, query, context) -> result
WriteLatent(latent_name, value, write_policy, context) -> latent_update
```

### Default write policy
`last-write-wins`

More advanced write policies such as averaging, additive accumulation, or stochastic fusion are deferred beyond v1.2.

---

## 7. Context

Context is split into local and global scopes.

### LocalContext

```text
LocalContext:
- unit
- parent?
- depth
- index?
- prev?
- next?
- neighbors?
- local_to_global?
```

Where:
- `unit` = current unit
- `parent?` = optional provenance unit or generator that emitted this unit
- `depth` = recursion or derivation depth
- `index?` = position in sequence when relevant
- `prev?`, `next?` = adjacent sequence context when relevant
- `neighbors?` = adjacent units for neighborhood-sensitive systems
- `local_to_global?` = mapping from local coordinates to global state-space

### GlobalContext

```text
GlobalContext:
- state
- latent
- summary
- rng
```

Where:
- `state` = full current state
- `latent` = shared latent registry
- `summary` = summary from previous completed step
- `rng` = random number source

### Normative rule
All parameters and operators see the world only through context.

---

## 8. Universal Coordinate Space

All latent queries operate in global state-space coordinates by default.

If a system uses local coordinates, `LocalContext` must provide a mapping into global coordinates.

### Normative rule
Global coordinates are the default interoperability space.

This allows units from different carrier kinds to query the same latent structures consistently.

---

## 9. Parameters and Evaluators

All parameters are evaluator-based.

```text
Param(context) -> value
```

Examples:
- `Scale(context)`
- `Color(context)`
- `Probability(context)`
- `SplitCount(context)`
- `BranchAngle(context)`
- `IterationLimit(context)`

### Normative rule
Parameters may depend on:
- local unit data
- parent or neighbor context
- latent queries
- previous-step global summary
- randomness from `rng`

### Example
```text
SplitCount(context) -> f(context.unit, context.global.summary)
```

---

## 10. Segment Semantics

Execution behavior is defined by a segment semantics object.

```text
Semantics:
- split_kind
- aggregate_kind
- transform_kind
```

Where:
- `split_kind` ∈ `{replicate, subdivide, rewrite, identity}`
- `aggregate_kind` ∈ `{union, chain, update}`
- `transform_kind` ∈ `{map, fold, in_place}`

### Meaning
- `map` = parallel per-unit transform
- `fold` = sequential ordered interpretation
- `in_place` = sampled or cell-wise update over existing support

This removes the need for a separate global `mode` field.

---

## 11. Core Operators

### Split

```text
Split(unit, context) -> emission
```

```text
Emission:
- units
- spawned_segments
- latent_writes?
```

Meaning:
- create derived units
- rewrite or subdivide existing structure
- emit newly spawned segments, possibly of another carrier kind
- optionally request latent writes

### Carrier transition rule
A unit may emit output associated with a different carrier kind.

Examples:
- a sequence leaf may emit a grid segment
- a grid cell may emit set-based particles
- a set object may emit a symbolic sequence

This is the formal bridge enabling hybrid systems.

---

### Transform

```text
Transform(segment, context) -> segment
```

Interpretation depends on `segment.semantics.transform_kind`:
- `map` = parallel transform over units
- `fold` = ordered sequential interpretation
- `in_place` = update over sampled support

---

### Keep

```text
Keep(unit, context) -> bool
```

Used for:
- pruning
- rejection
- density control
- overlap control
- dead branch termination
- budget enforcement

---

### Size

```text
Size(unit, context) -> scalar
```

This is type-dependent.

Examples:
- line -> length
- polygon -> area or diameter
- branch -> projected extent
- grid cell -> cell size

---

## 12. Aggregation

Aggregation constructs the next support of a segment from split output.

Typical cases:
- `union` for set-like segments
- `chain` for sequence-like segments
- `update` for grid-like segments

Aggregation is defined by segment semantics, not by a global runtime mode.

---

## 13. Pruning

Pruning is defined through `Keep`.

```text
Prune(segment, context) = retain all units u such that Keep(u, context) = true
```

### Normative rules
- v1.2 uses **post-transform pruning** as the default order
- if all units in a segment are pruned, the segment is removed
- sequence pruning must preserve order among retained elements

---

## 14. Normative Step Pipeline

Given state `S_t`, one global step is:

```text
Step(state S_t):

1. Derive global context from S_t.
2. For each existing segment in S_t:
   a. derive local contexts for its units
   b. execute Split on its units
   c. aggregate emitted units for that segment
3. Execute Transform on each surviving segment according to its semantics.
4. Apply Prune using Keep.
5. Remove empty segments.
6. Merge surviving transformed segments with spawned segments.
7. Update latent registry.
8. Compute summary.
9. Return S_{t+1}.
```

### Normative timing rule
Segments spawned by `Split` during step `t` are merged into the resulting state after transform/prune complete and participate in evolution starting from step `t+1`.

### Sequence rule
For sequence semantics, all rewrites are computed before the segment’s transform begins.

---

## 15. Rendering

```text
Render(visible, latent, summary) -> output
```

Only visible units are rendered directly, but rendering may query latent state and summary.

Examples:
- thickness from density
- color from latent fields
- debug overlays from guide geometry
- occupancy visualization

---

## 16. Canonical Mappings

### Recursive geometry
- `carrier_kind = set`
- `domain = dynamic`
- `split_kind = subdivide` or `replicate`
- `aggregate_kind = union`
- `transform_kind = map`

### L-systems / turtle systems
- `carrier_kind = sequence`
- `domain = dynamic`
- `split_kind = rewrite`
- `aggregate_kind = chain`
- `transform_kind = fold`

### Escape-time / sampled evaluation systems
- `carrier_kind = grid`
- `domain = fixed`
- `split_kind = identity`
- `aggregate_kind = update`
- `transform_kind = in_place`

### Cellular systems
- `carrier_kind = grid`
- `domain = fixed` or adaptive
- `transform_kind = in_place`
- `neighbors` may be required in local context

---

## 17. Canonical Hybrid Mappings

### Analytic-controlled L-system
- Segment A: `grid`
  - computes Mandelbrot escape-time or density
  - writes latent `"GrowthField"`
- Segment B: `sequence`
  - queries `"GrowthField"`
  - uses it to determine scale, branching, or split count

### IFS-generated texture
- Segment A: `set`
  - generates sparse or dense geometry
- Segment B: `grid`
  - queries geometry density or occupancy
  - colors or modulates samples accordingly

### Symbol-to-field bridge
- A sequence or set segment emits a grid segment for local texture or evaluation

### Field-to-object bridge
- A grid segment emits set-based particles or branches when sampled conditions are satisfied

---

## 18. Open Design Debt

Deferred beyond v1.2:
- topology and connectivity beyond current carrier abstractions
- asynchronous or multi-rate execution across segments
- advanced latent write policies
- conflict resolution for concurrent latent writes
- adaptive multiresolution grids
- formal scheduling for very large heterogeneous states

---

## Core Statement

A fractal or self-similar generative system is a unified iterative evolution of heterogeneous carrier segments, communicating through a shared latent registry and executing through common split / transform / prune semantics over explicit state.

\[
S_{t+1} = Transform(Split(S_t))
\]