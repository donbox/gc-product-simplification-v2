# Product Simplification

## Purpose

This note starts a broader simplification pass over the current Gas City
product surface.

Unlike the noun-centric CLI redesign work, this note is not primarily about
finding the perfect command tree for `city`, `rig`, `session`, `agent`,
`formula`, `order`, `skill`, or `mcp`.

Instead, it starts from a different question:

> What does the product contain today, how is that surface currently grouped,
> and where do the obvious simplification opportunities live?

## Out of Scope

This note intentionally does **not** try to settle:

- the Pack/Registry v2 design
- the noun-by-noun CLI redesign work
- final command names for every existing subsystem

Those tracks matter, but they each already have their own design home.

## Current Product Map

Ignoring the deeper noun redesign for the moment, the product appears to break
down into the following major concerns.

### 1. Root Scope, Discovery, and Initialization

This is the "where am I?" and "how do I get a city on disk?" layer.

Current surface area includes:

- global `--city` / `--rig` targeting
- `gc init`
- `gc cities`
- `gc register`
- `gc unregister`
- parts of `gc rig`

This layer mixes:

- local discovery
- machine-known registration
- workspace bootstrap

That is already a sign that "finding the target" and "managing the machine's
view of targets" may be more entangled than they should be.

### 2. City Lifecycle and Machine Supervision

This is the "bring the city up and keep it managed" layer.

Current surface area includes:

- `gc start`
- `gc stop`
- `gc restart`
- `gc suspend`
- `gc resume`
- `gc status`
- `gc supervisor ...`

There are really at least two concepts mixed together here:

- city lifecycle from the operator's point of view
- machine-wide supervisor lifecycle and diagnostics

That split is visible today, but the commands still live close together and
use partially overlapping vocabulary.

### 3. Configuration, Composition, and Build Staging

This is the "what should this city look like?" layer.

Current surface area includes:

- `gc config show`
- `gc config explain`
- current `gc import ...`
- current `gc pack ...`
- `gc build-image`

This concern set is especially interesting because it mixes:

- declarative composition
- import/package sourcing
- explanation/debug of resolved config
- image/build staging for deployment

All of those are related, but they are not the same kind of task.

### 4. Runtime, Sessions, and Provider Operations

This is the "live agent process/session" layer.

Current surface area includes:

- `gc session ...`
- `gc runtime ...`
- `gc service ...`
- `gc trace ...`
- `gc prime`
- `gc hook`
- `gc handoff`
- `gc nudge`

This is one of the densest and least obviously zoned parts of the current
surface. It mixes:

- direct human interaction with sessions
- provider/process maintenance
- introspection/tracing
- prompt generation
- hook-oriented automation helpers

Even before we redesign the nouns themselves, it looks likely that this layer
would benefit from stronger zoning between:

- runtime lifecycle
- operator interaction
- maintenance/debugging
- authoring/staging helpers

### 5. Work Substrate and Durable Records

This is the "what persistent substrate does the system use?" layer.

Current surface area includes:

- `gc bd ...`
- `gc beads health`
- `gc graph`
- `gc event emit`
- `gc events`
- `gc wait ...`

What stands out here is that these commands expose a mixture of:

- raw substrate access
- health/maintenance operations
- derived inspection views

This feels powerful, but also somewhat internal-facing. A likely simplification
question is whether these commands are all front-door product concepts, or
whether some are really maintenance/admin surfaces.

### 6. Communication, Coordination, and Work Routing

This is the "how does work move?" layer.

Current surface area includes:

- `gc mail ...`
- `gc sling`
- `gc convoy ...`

These commands are conceptually related:

- mail moves messages
- sling routes work
- convoys group related work

But they are presented as fairly separate subsystems. One likely simplification
question is whether the product should surface them as:

- one coherent coordination story
- or three peer subsystems

### 7. Declarative Automation and Reconciliation

This is the "declared logic plus repeated execution" layer.

Current surface area includes:

- `gc formula ...`
- `gc order ...`
- `gc converge ...`

This is another strong seam. The product clearly has a story around:

- reusable declarative logic
- scheduled/event-driven execution
- bounded iterative refinement

The current surface exposes all three, but the boundaries are still not
especially intuitive from the outside.

### 8. Insight, Health, and Operator Confidence

This is the "tell me what is happening and whether it is healthy" layer.

Current surface area includes:

- `gc doctor`
- `gc dashboard serve`
- `gc status`
- `gc service doctor`
- `gc trace ...`
- parts of `gc events`

This is probably the most obviously fragmented operator story today.

There are multiple ways to ask:

- is the city healthy?
- what is currently happening?
- where should I look next?

That suggests a major simplification opportunity around operator confidence and
debugging entry points.

### 9. Deferred Noun-Centric Surfaces

The current product also exposes top-level noun families whose redesign is
being tracked elsewhere:

- `gc agent ...`
- `gc rig ...`
- `gc session ...`
- `gc formula ...`
- `gc order ...`
- `gc skill ...`
- `gc mcp ...`

This note does not try to settle those nouns. It only notes that any broader
simplification pass will have to coexist with them.

## What the Product Feels Like Today

A rough characterization of the current product is:

- part orchestration SDK
- part operator console
- part runtime maintenance tool
- part work-routing shell
- part configuration/composition system

None of those are wrong. The challenge is that they all appear at the top
level, and not always with obvious zoning.

That leads to a product that is powerful, but easy to experience as:

- broad
- unevenly grouped
- rich in specialized front doors

## The Most Obvious Simplification Seams

These are the first seams I would want to pressure-test.

### 1. Machine-Wide vs City-Local Needs Better Separation

Examples:

- `cities`, `register`, `unregister`, and `supervisor` are clearly machine-wide
- `start`, `stop`, `status`, and much of the rest are city-local
- `rig` sits across both worlds because it is both discovery and city-local

This is a classic simplification seam. The product likely gets easier to teach
once machine-admin and city workflow are more deliberately separated.

### 2. Operator Surfaces Are Fragmented

Right now an operator may need to mentally choose among:

- `status`
- `doctor`
- `dashboard`
- `service`
- `trace`
- `events`

That is a lot of entry points for a single underlying need:

> "Tell me what is wrong, what is running, and where I should look next."

### 3. Runtime Interaction and Runtime Maintenance Are Mixed Together

Session interaction commands sit near provider/runtime maintenance commands,
but they serve different kinds of work:

- attach, peek, nudge, submit
- drain, restart requests, trace, service restarts

There may be a cleaner product if "work with the session" and "maintain the
runtime" are treated as distinct zones.

### 4. Raw Substrate Commands and User-Level Workflow Commands Coexist

Commands like `gc bd` and `gc event emit` are powerful and sometimes necessary,
but they feel closer to substrate access than to core product workflow.

That does not mean they are wrong. It does suggest they may deserve clearer
framing as:

- advanced tools
- admin/maintenance tools
- developer/internal tools

rather than just one more peer front door.

### 5. There Are Several Overlapping "Action Engines"

Today, GC appears to have at least these execution stories:

- slinging work
- running orders
- cooking formulas
- convergence loops
- session submission and nudging

That is a lot of ways to make the system do something. Some of them are truly
different. Some may be unnecessarily far apart in the product story.

### 6. Singular/Plural and Command-Family Consistency Is Uneven

Examples:

- `event` and `events`
- `skill` and `skills`
- `mail` is both noun and workflow
- `pack` and `import` overlap today

This is a smaller issue than the architectural seams above, but it is still
part of the product's felt complexity.

## Initial Working Hypotheses

These are not decisions yet. They are the most promising simplification
directions visible from the current map.

### Hypothesis 1: GC needs stronger zoning before it needs more commands

The main problem may not be that GC has too few abstractions. It may be that
operator, authoring, substrate, and machine-admin workflows are all visible at
once.

### Hypothesis 2: "Advanced/admin" surfaces should be named or grouped more deliberately

Substrate-facing and maintenance-facing commands may belong in:

- clearer admin families
- narrower advanced namespaces
- or at least more explicit documentation zones

### Hypothesis 3: The operator story can probably be reduced to fewer front doors

A simpler product likely offers a smaller number of obvious first stops for:

- city health
- runtime health
- current activity
- debugging

### Hypothesis 4: Current execution pathways should be compared side by side

`sling`, `order`, `formula`, `converge`, `session submit`, and related flows
probably need one comparative design pass that asks:

- what each one is for
- where they overlap
- which ones should be primary vs specialized

### Hypothesis 5: Product simplification and noun simplification should stay separate, but inform each other

The noun redesign work is important, but it should not carry the burden of
solving every broader product-zoning problem by itself.

At the same time, whatever simplification happens here will affect which noun
surfaces feel first-class and which feel secondary.

## Near-Term Questions

These are the questions I would want this repo to work through next.

1. What are the best top-level zones for the current product, independent of
   individual noun redesign?
2. Which current commands are true front-door product concepts, and which are
   really admin/maintenance/internal tools?
3. Can the operator-confidence story be reduced to fewer entry points?
4. Which execution pathways are truly distinct, and which are just different
   faces of the same underlying mechanism?
5. Where should machine-wide configuration and city-local workflow be more
   deliberately separated?
