# Fractal Framework — Merged v1.1 Redline

This document is a **keep / change / add** redline against the current v1.1 kernel.
It is intended as a direct implementation-oriented refinement pass.

---

## Keep

These should stay unchanged in spirit.

### 1. Top-level law
```text
S_{t+1} = Transform(Split(S_t))
```

Keep this as the conceptual identity.

### 2. Execution modes
- set
- sequence
- evaluation

This is the most important refinement and should remain core.

### 3. Domain split
- fixed domain
- dynamic domain

Keep this.

### 4. Local / global context split
Keep this exactly.

### 5. Latent lifecycle categories
- persistent
- recomputed
- ephemeral

Keep this taxonomy.

### 6. Keep/prune logic
```text
Keep(unit, context) -> bool
```

Keep as mandatory.

---

## Change

These are the main places where the kernel should be tightened.

### 1. Change `State`
Current:
```text
State:
- carrier
- iteration
- latent
- summary
- mode
- domain
```

Replace with:
```text
State:
- carrier
- iteration
- mode
- domain
- latent
- summary
```

Reason: puts semantic control fields first and derived/support fields later.

---

### 2. Change `carrier` wording
Current:
```text
carrier = set | sequence | grid
```

Replace with:
```text
carrier = the structured evolving support of the state

v1.1 carrier kinds:
- set
- sequence
- grid
```

Reason: avoids making it sound like the list is a final ontology.

---

### 3. Change core operators
Current:
```text
Split(unit, context) -> units
Transform(units_or_state, context) -> units_or_state
Keep(unit, context) -> bool
Size(unit, context) -> scalar
```

Replace with:
```text
Split(unit, context) -> emitted_units
Keep(unit, context) -> bool
Size(unit, context) -> scalar
```

and define transform **by mode**:

```text
TransformSet(emitted_units, context) -> emitted_units
TransformSequence(sequence, context) -> sequence
TransformEvaluation(sample_or_grid_state, context) -> sample_or_grid_state
```

Reason: `units_or_state` is too vague and will become a real implementation problem.

---

### 4. Change aggregation section
Current:
- union
- chain
- update

Keep the idea, but rewrite as:

```text
Aggregation is mode-defined:

- set mode: Union
- sequence mode: Chain
- evaluation mode: Update
```

Add:
```text
Aggregation is an execution semantic, not a user-defined fractal operator.
```

Reason: keeps minimality.

---

### 5. Change sequence mode wording
Current wording is good, but add one hard rule:

```text
In sequence mode, Split is batch-parallel over the current sequence generation.
All rewrites are computed before TransformSequence begins.
```

Reason: this must be normative, not implicit.

---

### 6. Change fixed/dynamic domain wording
Current:
- fixed domain mainly in evaluation mode
- dynamic domain mainly in set / sequence modes

Replace with:
```text
Typical pairing:
- evaluation mode <-> fixed domain
- set / sequence modes <-> dynamic domain

Other pairings are allowed only if explicitly defined by the system.
```

Reason: stronger without being overly rigid.

---

### 7. Change latent API wording
Current:
```text
UpdateLatent(state, previous_latent) -> latent
Sample(name, query, context) -> value
QueryLatent(type, query, context) -> result
```

Replace with:
```text
UpdateLatent(state, previous_latent, context) -> latent
Sample(latent_name, query, context) -> value
QueryLatent(latent_type, query, context) -> result
```

Reason: gives latent update access to execution context too.

---

## Add

These are the most important missing pieces.

### 1. Add explicit pipeline
This is the biggest missing operational clarification.

Add:

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

Compact form:
```text
S_{t+1} =
  Summarize(
    UpdateLatent(
      Prune(
        Transform(
          Aggregate(
            Split(S_t)
          )
        )
      )
    )
  )
```

Note:
- this is the operational form
- the top-level conceptual identity remains `S_{t+1} = Transform(Split(S_t))`

---

### 2. Add explicit prune definition
Add:

```text
Prune(carrier, context) = { u in carrier | Keep(u, context) = true }
```

For sequence mode, pruning must preserve sequence order among retained elements.

---

### 3. Add carrier/object relationship
Add:

```text
Carrier elements are the units of evolution.

In v1.1, carrier elements are represented as objects or object-like samples,
depending on the carrier kind:
- set -> objects
- sequence -> ordered objects
- grid -> samples / cells / point-states
```

Reason: clarifies that grid evaluation need not instantiate heavyweight geometric objects.

---

### 4. Add sequence-specific context rule
Add:

```text
Sequence mode may require:
- index
- prev / next
- neighbors
- stack view

These are optional in generic context but required when the rewrite/interpreter depends on them.
```

---

### 5. Add design-debt section
Add a short final section:

## Open design debt
- topology / connectivity model
- carrier extension beyond set/sequence/grid
- whether latent fields are first-class objects or services
- whether transform dispatch remains mode-based or becomes typed interface families
- whether adaptive domains require a third domain subtype beyond fixed/dynamic

Reason: keeps unresolved issues visible without polluting the kernel.

---

### 6. Add explicit note on color
Doc A had a useful distinction that should not be lost.

Add:

```text
Color may be derived from:
- state-local metrics
- latent fields
- hybrid combination

Canonical color modes:
- state-based
- field-based
- hybrid
```

---

## Recommended implementation stance

If implementing now, prefer:

```text
TransformSet
TransformSequence
TransformEvaluation
```

instead of one polymorphic `Transform(units_or_state, context)` signature.

Likewise, treat:
- `set`
- `sequence`
- `grid`

as three carrier backends under one framework interface.

---

## Final instruction for Claude

Please use this redline to revise the kernel into a cleaner **v1.1 implementation-ready spec**.

Priority order:
1. resolve operator contracts
2. insert the explicit step pipeline
3. make sequence-mode rewrite semantics normative
4. clarify carrier/object/sample relationship
5. preserve minimality — do not over-generalize

The goal is:
- keep the unified conceptual identity
- but make the runtime semantics explicit enough that an engineer could build a first engine from it

<!--
Claude Assessment of this Redline (2026-04-19)

== Overall verdict ==

High quality. This redline directly resolves the three main blockers identified in
the Claude/Gemini assessments embedded in the kernel. Would advance
implementation-readiness from ~60% to ~75-80%.

== What it resolves correctly ==

1. Transform signature — The biggest blocker. Replacing Transform(units_or_state, context)
   with three typed operators (TransformSet, TransformSequence, TransformEvaluation)
   is the right call. Eliminates the deferred type error.

2. UpdateLatent placement — The explicit 8-step pipeline Step(state) solves the ambiguity
   about when UpdateLatent runs. The compact formula
   Summarize(UpdateLatent(Prune(Transform(Aggregate(Split(S_t))))))
   is consistent with the pipeline and directly usable.

3. Split batching contract — Making the sequence-mode parallel batch rule normative
   (not just implied in canonical mappings) closes the implementation trap.

4. Keep/Prune formal connection — Prune(carrier, context) = { u in carrier | Keep(u, context) = true }
   was explicitly missing and is now defined.

5. Domain/mode coupling — "Typical pairing... other pairings allowed only if explicitly defined"
   is meaningfully stronger than "mainly."

6. UpdateLatent context arg — Small but correct. Latent update logic clearly needs
   execution context.

7. Design-debt section — Honest accounting. Keeps open issues visible without
   polluting the kernel.

== Remaining gaps not resolved ==

- Topology / connectivity: Deferred to design debt — correct.
- Latent spatial coordinate system: NOT addressed. Sample(latent_name, query, context)
  with no coordinate system definition means latent queries are still black-box
  functions at runtime. Gemini flagged this; the redline does not engage with it.
  This is the most consequential omission.
- Grid cell as Object vs. lightweight sample: Partially addressed
  (grid -> samples/cells/point-states) but no formal distinction.
- TransformEvaluation(sample_or_grid_state, ...): "sample_or_grid_state" retains a
  mild type ambiguity — per-sample or whole-grid? Should be resolved explicitly.

== Minor observations ==

- State field reordering (Change #1) is cosmetic; correct but low-value.
- Carrier wording change (Change #2) is good — removes false closure on the ontology.
- Color modes (Add #6) is a useful recovery from Doc A — restores specificity that
  was demoted to a mere Param example.
- Compact pipeline form drops context derivation (Step 1) from the notation —
  intentionally implicit, but worth noting when implementing.

== Recommended addition after applying this redline ==

Add one statement on latent coordinate space to close Gemini's concern:
  "Latent field queries operate in global state-space coordinates unless explicitly
  overridden by a LocalContext transform."
This would close the last major black-box without adding new design burden.
-->

