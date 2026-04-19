# Fractal Framework — Assessment Spec (v1.1)

## Core Principle

S_{t+1} = Cull(Transform(Split(S_t)))

---

## Execution Modes

mode ∈ {map, global, sequence}

- map: independent per-object operations
- global: full-state operations
- sequence: ordered, left-to-right fold (turtle)

---

## Operators

Split(State, context, mode) -> State  
Transform(State, context, mode) -> State  
Cull(State, context) -> State (optional)

---

## State

State = set (or sequence) of Objects

Object =
- type
- visibility: visible | latent
- content

---

## Latent Objects

- part of State
- created/updated by Split/Transform
- persist across iterations
- accessed via:

query(latent_objects, context) -> value

---

## Parameters

Param(context) -> value

---

## Context

- object
- state
- parent
- depth
- rng
- latent_objects
- index? (sequence mode)
- neighbors? (optional)

---

## Color

color = ColorSpec(context)

---

## Mandelbrot / Julia

Special case:

- Split = identity
- Transform = iterative function
- mode = map
- stop = escape or max_iter

---

## Stop

- size(object) < ε
- total_count > N

size = SizeEvaluator(context)

---

## Core Statement

Fractal = iterative transformation of visible and latent objects via Split and Transform, executed under a defined mode (map, global, or sequence), with all parameters expressed as evaluators over context.