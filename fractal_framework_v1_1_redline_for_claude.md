# Fractal Framework — v1.1 Frozen Draft

## Goal
Define a minimal but implementation-ready framework for fractal and self-similar generative systems.

The framework should support:
- recursive geometric fractals
- stochastic fractals
- L-systems and turtle systems
- escape-time / evaluation fractals
- hybrid systems combining visible geometry, latent structure, and evaluator-driven control

---

## Core Principle

\[
S_{t+1} = Transform(Split(S_t))
\]

This remains the **conceptual top-level law**.

It describes the evolution of state at the highest level, while concrete execution semantics are defined by the runtime mode.

---

## State

A state contains:

- `carrier`
- `iteration`
- `mode`
- `domain`
- `latent`
- `summary`

```text
State:
- carrier
- iteration
- mode
- domain
- latent
- summary
```

### Meaning
- `carrier` = the structured evolving support of the state
- `iteration` = current step index
- `mode` = execution semantics
- `domain` = whether support is fixed or generated
- `latent` = non-rendered but queryable control/evaluation structures
- `summary` = derived compact information about the current state

---

## Carrier

`carrier` is the structured evolving support of the state.

v1.1 carrier kinds:

- `set`
- `sequence`
- `grid`

### Intended meaning
- `set` = unordered or weakly ordered evolving units
- `sequence` = ordered units with position-dependent meaning
- `grid` = fixed sample lattice or sample field

This list is the **v1.1 carrier set**, not a final ontology.

---

## Execution Modes

The framework defines three execution modes.

### 1. Set mode
For independently evolving elements.

Examples:
- Sierpinski
- Koch-like systems
- recursive geometric refinement
- many stochastic branching systems

Semantics:
- `Split` is applied per unit
- `TransformSet` acts over emitted units
- aggregation = `Union`

### 2. Sequence mode
For ordered symbolic or path-based evolution.

Examples:
- L-systems
- turtle systems
- context-sensitive rewrite systems

Semantics:
- `Split` is a **batch-parallel rewrite** over the current sequence generation
- all rewrites are computed before transform begins
- `TransformSequence` is a left-to-right fold / interpreter
- aggregation = `Chain`

**Normative rule**
```text
In sequence mode, Split is batch-parallel over the current sequence generation.
All rewrites are computed before TransformSequence begins.
```

### 3. Evaluation mode
For fixed-domain iterative refinement.

Examples:
- Mandelbrot
- Julia
- orbit evaluation systems
- escape-time sample updates

Semantics:
- `Split = identity` or explicit domain refinement if the system defines it
- `TransformEvaluation` updates evaluation state
- aggregation = `Update`

For v1.1, the formal contract is:

```text
TransformEvaluation(grid_state, context) -> grid_state
```

Per-sample vectorization is treated as an implementation strategy inside this mode.

---

## Domain Types

### Fixed domain
Support does not grow.

Examples:
- pixel grid
- fixed sample lattice
- fixed evaluation domain

### Dynamic domain
Support is generated, expanded, or structurally rewritten during evolution.

Examples:
- recursive geometry
- branching systems
- symbolically generated paths

### Typical pairing
- evaluation mode <-> fixed domain
- set / sequence modes <-> dynamic domain

Other pairings are allowed only if explicitly defined by the system.

---

## Visible and Latent Structure

Each evolving unit may contribute to either visible or latent structure.

### Visible
Rendered directly.

Examples:
- point
- line
- polygon
- curve
- stroke
- visible sample output

### Latent
Not rendered directly by default, but used for control, querying, evaluation, and coherence.

Examples:
- scalar field
- color field
- occupancy field
- guide geometry
- spatial index
- turtle stack/state
- control summaries

---

## Carrier Elements

Carrier elements are the units of evolution.

In v1.1, carrier elements are represented as objects or object-like samples depending on carrier kind:

- `set` -> objects
- `sequence` -> ordered objects
- `grid` -> samples / cells / point-states

This means grid-based systems do **not** need to instantiate heavyweight geometric objects for every sample.

---

## Object Form

For object-based carriers, the basic unit form is:

```text
Object:
- type
- visibility
- content
- attrs
```

Where:
- `type` = semantic kind
- `visibility` ∈ `{visible, latent}`
- `content` = geometry / symbol / value payload
- `attrs` = additional derived or control attributes

---

## Context

Context is split into two parts.

### LocalContext
Information specific to the current unit.

```text
LocalContext:
- object
- parent
- depth
- index?
- prev?
- next?
- neighbors?
- local_to_global?
```

### GlobalContext
Information about the whole environment.

```text
GlobalContext:
- state
- latent
- summary
- rng
```

This split prevents mixing per-unit and world-level information.

---

## Coordinate Semantics

Latent queries operate in **global state-space coordinates by default**.

If a system needs local coordinates, `LocalContext` may provide a `local_to_global` transform or equivalent mapping.

So the default rule is:

```text
Latent field queries use global coordinates unless explicitly overridden by local context.
```

This applies to:
- occupancy queries
- color field sampling
- density evaluation
- guide field lookup

---

## Parameters

All tunable behavior should be evaluator-based.

```text
Param(context) -> value
```

Examples:
- `Scale(context)`
- `Color(context)`
- `Probability(context)`
- `SplitCount(context)`
- `BranchAngle(context)`

This allows:
- deterministic values
- depth-dependent behavior
- stochastic choice
- field-driven control
- parent- or neighbor-dependent variation

---

## Core Operators

### Split
```text
Split(unit, context) -> emitted_units
```

Meaning:
- subdivision
- rewrite
- replication
- branching
- identity emission
- domain-local refinement

### Transform
Transform is **mode-specific**.

```text
TransformSet(emitted_units, context) -> emitted_units
TransformSequence(sequence, context) -> sequence
TransformEvaluation(grid_state, context) -> grid_state
```

This removes the previous ambiguity of a single overloaded transform signature.

### Keep
```text
Keep(unit, context) -> bool
```

Used for pruning, rejection, density control, overlap control, and other filtering decisions.

### Size
```text
Size(unit, context) -> scalar
```

`Size` is type-dependent.

Examples:
- line -> length
- polygon -> area or diameter
- curve -> length or bounding diameter
- grid cell -> cell size

There is no universal size formula across all unit types.

---

## Aggregation

Aggregation is mode-defined:

- set mode: `Union`
- sequence mode: `Chain`
- evaluation mode: `Update`

Aggregation is an **execution semantic**, not a user-defined fractal operator.

Its role is to construct the next carrier-form from emitted or transformed results.

---

## Pruning

Pruning is explicitly defined through `Keep`.

```text
Prune(carrier, context) = { u in carrier | Keep(u, context) = true }
```

For sequence mode, pruning must preserve sequence order among retained elements.

Typical pruning uses:
- tiny element removal
- invisibility removal
- overlap rejection
- density limiting
- dead branch termination
- budget control

---

## Latent Model

Latent state is not just hidden geometry.
It may include:
- fields
- indices
- summaries
- control structures
- guide objects
- evaluation helpers

### Latent lifecycles

#### Persistent
Survives across iterations.

#### Recomputed
Rebuilt from current state each step.

#### Ephemeral
Exists only during one execution step.

---

## Latent API

```text
UpdateLatent(state, previous_latent, context) -> latent
Sample(latent_name, query, context) -> value
QueryLatent(latent_type, query, context) -> result
```

Examples:
- sample a color field at a point
- query occupancy over a region
- query local density
- query nearest guide object

---

## Summary

The framework stores summaries, not necessarily full history.

```text
Summarize(state) -> summary
```

Examples:
- bounds
- counts
- density metrics
- histograms
- escape statistics
- branch statistics

---

## Color

Color may be derived from:
- state-local metrics
- latent fields
- hybrid combinations

Canonical color modes:

- `state-based`
- `field-based`
- `hybrid`

Examples:
- escape-time coloring
- depth-based coloring
- field-coherent gradient coloring
- geometry + latent blending

---

## Step Pipeline

The runtime step is explicitly defined as:

```text
Step(state):

1. derive execution context
2. execute Split according to mode
3. aggregate split results according to mode
4. execute Transform according to mode
5. apply Keep / pruning
6. update latent state
7. compute summary
8. return next state
```

Compact operational form:

\[
S_{t+1} =
\mathrm{Summarize}(
  \mathrm{UpdateLatent}(
    \mathrm{Prune}(
      \mathrm{Transform}(
        \mathrm{Aggregate}(
          \mathrm{Split}(S_t)
        )
      )
    )
  )
)
\]

### Note
This is the **operational execution form**.

The conceptual identity remains:

\[
S_{t+1} = \mathrm{Transform}(\mathrm{Split}(S_t))
\]

---

## Rendering

```text
Render(visible, latent, summary) -> output
```

Only visible elements are rendered directly, but rendering may query latent state and summary information.

Examples:
- thickness from density
- color from latent field
- debug overlays from guide geometry
- occupancy visualizations

---

## Canonical Mappings

### L-systems
- carrier = `sequence`
- mode = `sequence`
- domain = `dynamic`
- split = parallel symbol rewrite
- transform = turtle fold
- may require `index`, `prev`, `next`, `neighbors`, `stack view`

### Mandelbrot / Julia
- carrier = `grid`
- mode = `evaluation`
- domain = `fixed`
- split = identity
- transform = iterative grid-state update
- implementation may be vectorized internally

### Recursive geometry
- carrier = `set`
- mode = `set`
- domain = `dynamic`
- split = subdivision / replication / branching
- transform = geometric map/filter over emitted units

---

## Recommended Implementation Stance

If implementing v1.1, prefer:
- `TransformSet`
- `TransformSequence`
- `TransformEvaluation`

rather than one polymorphic transform signature.

Likewise, treat:
- `set`
- `sequence`
- `grid`

as three carrier backends under one unified framework interface.

Execution modes share a unified semantic interface but may use specialized runtime backends.

---

## Non-negotiable v1.1 Additions

Compared to v1.0, v1.1 explicitly includes:

1. execution mode
2. domain type
3. local vs global context
4. latent query/update API
5. keep/prune logic
6. aggregation semantics
7. explicit step pipeline
8. sequence-mode batch rewrite rule
9. carrier/object/sample clarification
10. latent coordinate semantics

---

## Open Design Debt

The following are intentionally deferred beyond the frozen v1.1 core:

- topology / connectivity model
- carrier extension beyond set / sequence / grid
- whether latent fields are first-class objects or query services
- whether adaptive domains require a third subtype beyond fixed / dynamic
- whether future versions should formalize typed operator families more broadly
- composition rules for multiple transforms / multiple latent systems

These remain visible as tracked design debt, not unresolved core ambiguity.

---

## Core Statement

A fractal is an iterative evolution of explicit state over a carrier, executed under a specific mode, with visible and latent structure, evaluator-driven parameters, and optional pruning.

\[
S_{t+1} = Transform(Split(S_t))
\]