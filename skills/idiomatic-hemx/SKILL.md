---
name: idiomatic-hemx
description: >-
  Use when a user explicitly asks how to design, implement, review,
  productionize, or scale an application built with Hemx and Hemplate,
  including whether microservices belong. Preserve Hemx's generated-resource
  and server-owned effect model and choose the smallest architecture that meets
  the demonstrated need. Do not use for generic Rust/Axum advice, visual design
  alone, or development of the Hemx framework itself.
---

# Idiomatic Hemx

## One job

Keep a Hemx application on its intended path as it grows:

```text
semantic .heml templates
  -> generated typed resources
  -> ordinary Rust application code
  -> typed UI effects
  -> the small Hemx browser runtime
```

Scale storage, providers, processes, and deployment around this loop. Do not
replace it with selectors, HTML assembled in Rust, a client state graph, or
speculative services.

## Establish the real contract

Read the application's instructions, dependency versions, nearest templates,
public APIs, tests, and relevant Hemx documentation before naming APIs or
changing architecture. In the Hemx repository, use `README.md`, Rustdoc, and
tests.

If authentication, storage, transport, replay, conflict, deployment, or service
contracts are missing, identify the missing product decision. Do not invent a
provider or abstraction that makes an unknown external effect look complete.

## Trace one path end to end

Start with the concrete mutation, navigation, push update, offline action, or
recovery path that must work:

```text
browser or HTTP input
  -> authentication, CSRF, and typed validation
  -> application command or query
  -> authoritative state transition
  -> generated page or target
  -> typed effect response
  -> root-scoped runtime application
  -> visible success or explicit recovery
```

Architecture changes are premature until this path and its failure behavior are
clear.

## Use the canonical authoring level

Default to:

- semantic `.heml` templates rendered through Hemplate;
- ordinary Rust domain and application code;
- `#[hemx::handler]`, typed forms, generated resources, and `IntoEffect`;
- generated targets, forms, controls, keyed slots, and event helpers;
- real links and GET forms for navigation and history;
- plain CSS for appearance;
- Hemx Axum adapters and shared runtime paths instead of parallel glue.

Never construct, concatenate, interpolate, or format HTML in Rust. Keep routing,
authorization, sessions, persistence, feature flags, observability, and domain
policy in the application or its integration crates rather than Hemx core.

Use optional layers only when the user job proves the need:

- opaque JavaScript islands for isolated high-frequency browser behavior;
- client-local/Wasm handlers as explicit opt-ins, not the default state model;
- SSE or WebSocket transport for canonical typed effects, not domain policy;
- durable sync only after command, event, projection, replay, deletion, and
  conflict semantics are defined.

The DOM, stored HTML fragments, and stored effect batches are not business
truth. Keep authoritative state in application models and durable providers;
derive UI output from that state.

## Keep the core functional and the shell explicit

Prefer deterministic validation, authorization decisions, command application,
state transitions, and view-model derivation over explicit values. Keep HTTP
extraction, sessions, database I/O, clocks, IDs, queues, push connections, and
host capabilities in a thin imperative shell.

Do not create a trait for every function merely to mock it. Introduce an adapter
where ownership or side effects form a real boundary.

Expected failures must remain typed and useful. Validation should target
field/form feedback; recoverable request failures need an explicit local outlet;
malformed or incompatible responses must fail closed. Never turn infrastructure
failure into an empty success effect.

## Scale one demonstrated pressure at a time

Use the first stage that meets the actual load, availability, ownership, or
compliance need:

1. **One process:** one deployable and the simplest durable store that satisfies
   restart, backup, and recovery needs.
2. **Production monolith:** external durable providers where required, with
   integration proof for auth, persistence, migrations, failure, and rollback.
3. **Horizontal web tier:** stateless replicas around shared authoritative
   providers; process memory stays ephemeral unless loss semantics are explicit.
4. **Push or worker tier:** split only a measured connection, latency, CPU, queue,
   or failure domain; define ordering, idempotency, retry, and recovery first.
5. **Microservices:** only for an independently owned bounded capability with its
   own release, scaling, or service-level pressure and an explicit data/API/event
   contract.

Prefer query and index fixes, bounded concurrency, explicit caching, resource
tuning, and replicas before network boundaries. A large codebase is a module
boundary problem before it is a microservice problem.

## Refuse attractive detours

Do not add:

- selector strings or raw wire operations when generated helpers exist;
- a second client application model for server-owned UI;
- framework-owned database, auth, routing, or deployment policy;
- stored HTML/effect output as authoritative state;
- offline, broker, queue, or service abstractions without chosen semantics;
- speculative compatibility layers or extension seams.

## Proof

Verify the smallest real path at its owning boundary: deterministic domain tests,
render/effect tests, Axum integration tests, and browser/runtime tests only where
the browser owns behavior. For scaling changes, require evidence for the pressure
being addressed and test the new failure mode introduced by the boundary.
