# Fractal Framework — Assessment Spec (v1.0)

## Goal
Define a minimal, extensible, unified framework for fractals using:
- Split + Transform
- State evolution
- Evaluator-based parameters
- Visible + latent objects
- Color derived from state

---

## Core Principle
S_{t+1} = Transform(Split(S_t))

---

## State Model
State = set of Objects

Object =
- type
- visibility (visible | latent)
- content

---

## Object Types
Visible:
- point, line, polygon, curve

Latent:
- scalar field, color field, control field, dynamical object, guide geometry

---

## Core Operators
Split(object, context) -> [objects]  
Transform(objects, context) -> [objects]

---

## Parameter System
Param(context) -> value

Examples:
- Color(context)
- SplitCount(context)
- Probability(context)
- Scale(context)

---

## Context
- object
- state
- parent
- depth
- local_coords (optional)
- rng
- latent_objects

---

## Splitting Strategies (concept)
- point → clone / radial
- line → subdivision
- polygon → triangulation / grid
- triangle → recursive split
- curve → parameter split

---

## Extension Strategies (concept)
- replication
- branching
- radial growth
- contractive refinement
- expansive growth

---

## Modes
Contractive: children inside parent  
Expansive: free growth  
Mixed: both

---

## Randomness
rng passed via context

---

## State Variables
- iteration/depth
- escape time
- accumulated metrics
- branch index

Principle: store summaries, not full history

---

## Color System
color = ColorSpec(context)

ColorSpec(context) -> color

Modes:
- state-based (escape time)
- local field (geometry gradient)

Latent color fields ensure coherence

---

## Stop Conditions
- size(object) < ε
- total_count > N
(optional depth limit)

---

## Size
size(object) -> scalar

---

## Rendering
Render only visible objects, may query latent ones

---

## Capabilities
Supports:
- recursive fractals
- stochastic systems
- trees
- Mandelbrot / Julia
- hybrid systems

---

## Strengths
- minimal
- unified
- extensible
- supports randomness
- logical color system

---

## Weaknesses
- Transform complexity
- state growth
- implicit evaluation

---

## Open Questions
- Is Split + Transform sufficient?
- Is latent object model correct?
- Is color purely derived?
- Are evaluators optimal?
- Is it still minimal?

---

## Core Statement
Fractal = iterative transformation of visible and latent objects via Split + Transform.
