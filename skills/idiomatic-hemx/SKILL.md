---
name: idiomatic-hemx
description: >-
  Use when a user asks to design, implement, review, debug, productionize, or
  scale an application built with Hemx and Hemplate. Keep templates semantic,
  use generated resource types, return server-owned effects, preserve the
  portable Hemplate runtime boundary, and choose proof at the behavior owner.
  Do not use for generic Rust or Axum work, Hemplate-only questions, visual
  design alone, or development of the Hemx framework itself.
---

# Idiomatic Hemx

## One job

Keep a Hemx application on its intended path:

```text
semantic .heml template
  -> hemx-build generated resources
  -> ordinary Rust handler and application state
  -> typed Hemx effects
  -> hemx-axum response or canonical push transport
  -> root-scoped browser runtime application
```

Improve and scale this path instead of creating a parallel client application,
copying generated identifiers, or assembling HTML in Rust.

## Read the selected contract first

Before naming an API or changing architecture, inspect:

1. the application's `Cargo.toml`, `build.rs`, and selected crate versions;
2. the nearest `.heml` template and its generated module usage;
3. the handler, router, state boundary, and focused tests;
4. public README, crate Rustdoc, and tests for those exact versions.

Hemx crates may version independently. In particular, do not assume
`hemx-test` has the same version as the main `hemx` facade. Do not rely on a
private example, an unpublished API, or copied generated output.

If auth, persistence, transport, replay, conflict, or deployment behavior is
not chosen, state the missing decision. Do not hide it behind a speculative
trait, provider, queue, or service.

## Choose the smallest Hemx path

Classify the requested behavior before writing code.

| User-visible behavior | Default path |
| --- | --- |
| Full page or navigation | real link or GET form -> application query -> Hemplate page response with the generated build fingerprint |
| Form or control interaction | native form/control semantics -> generated form/handle -> `#[hemx::handler]` -> command -> generated target effect |
| Reusable presentation | typed Hemplate view, composed from nested views |
| Group of checked handlers | `#[hemx::component]` against the generated component contract |
| Server push | canonical effect batch over the supported SSE/WebSocket adapter |
| Isolated high-frequency browser behavior | explicit JavaScript island |
| Server-side Wasm isolate | normal Hemx core plus the platform adapter owned by the application |
| Durable/offline synchronization | only after event, replay, deletion, ordering, and conflict semantics exist |

Start with one server process and ordinary Axum application state unless a
measured availability, scaling, ownership, or isolation pressure requires more.
A larger codebase needs module boundaries before it needs network boundaries.

## Author at the semantic level

Use `.heml` templates to declare roots, slots, forms, handles, keyed targets,
classes, and events. Let `hemx-build` generate their Rust vocabulary. A normal
build script uses the public `hemx-build` application builder; inspect the
selected version rather than recreating generated files.

In application code:

- use generated target, form, handle, class, component, and event types;
- keep handler inputs typed and return `IntoEffect` or the selected adapter's
  documented response type;
- derive a dedicated view model from authoritative application state;
- use `Slot`, keyed slots, and generated helpers instead of numeric resource IDs;
- preserve the browser-native contract: links keep `href`; forms keep their real
  `method` and `action`; controls keep names, labels, and relevant ARIA state;
- add documented `data-hemx-*` source attributes for enhancement instead of
  replacing those native semantics or hand-authoring lowered runtime markers;
- use plain CSS and stable semantic attributes for appearance.

A mutation form should still describe a real URL-encoded `POST`; Hemx adds the
generated handle and event contract and injects its hidden dispatch field. Use
multipart only when the selected release and the actual form fields require it.
Never concatenate or format HTML in Rust. Hemplate interpolation is escaped by
default; raw HTML is an explicit trust boundary, not a convenience. Keep
routing, authorization, sessions, persistence, migrations, feature flags, and
domain policy in the application or their real integration crates.

`#[hemx::component]` checks a handler group against a generated template
contract. It is not a second renderer and is unnecessary for a purely
presentational view.

Keep a JavaScript island visibly local: the server may dispatch an explicit
Hemx event into the current root, while the island owns only the browser-only
behavior that consumes it (for example a native dialog, canvas, or
high-frequency gesture). Re-query island elements after HTML morphs; do not let
the island retain rendered nodes or mirror authoritative application state.

For an Axum application, keep the reachable integration chain together: render
the Hemplate page through the selected release's `PageResponse` API with the
generated `BuildFingerprint`, serve the packaged `RuntimeJs`, extract an
`InteractionRequest`, and dispatch it through the registry produced by
`#[hemx::app]`. Generated form handlers use the selected release's documented
typed form boundary. With the current release, import the adapter's `Form` and
write `Form<T>`, or use either supported qualified path: `hemx::Form<T>` or
`hemx_axum::Form<T>`. Do not independently parse the same body or manually
register numeric handles.

## Preserve ownership through an interaction

Keep responsibility in this order:

```text
HTTP/browser input
  -> extraction, size/origin checks, authentication, authorization
  -> typed validation
  -> application command or query
  -> authoritative state transition
  -> view-model derivation
  -> generated target and typed effect
  -> current-root runtime application
```

The application owns state, security decisions, and recovery. Hemplate owns
rendering. Hemx owns the checked effect protocol and browser application.
Neither the DOM, rendered HTML fragments, nor stored effect batches are business
truth.

Represent expected failures with variants that preserve their recovery path:
field feedback, form feedback, local retry, navigation, or terminal failure.
Keep infrastructure error sources for diagnostics, but never render raw internal
error text or turn failure into an empty success effect.

## Respect the runtime and wire boundaries

Hemplate separates host compilation from target rendering:

- template parsing and proc-macro generation stay on the host;
- generated templates render through the parser-free `hemplate-runtime`;
- a constrained target must not acquire Tree-sitter or parser crates through a
  convenience dependency;
- use the documented no-default-feature facade setup or direct runtime crate for
  the selected Hemplate version instead of copying runtime traits.

For Hemx transport:

- use `BuildFingerprint` from the generated build contract; do not substitute an
  application version string;
- return `EffectResponse` through `hemx-axum` or generated dispatch rather than
  inventing an effect media type;
- use the canonical `EffectBatch` wire representation for SSE/WebSocket delivery;
- reject malformed or incompatible payloads instead of guessing a fallback;
- keep effects scoped to the current generated root and let the page adapter
  attach its build marker instead of writing one into the template;
- use `InteractionRequest`/generated form dispatch for supported form and file
  inputs rather than bypassing media-type and body-limit handling;
- preserve the canonical `data-hemx-*` runtime vocabulary emitted by the build
  and adapter layers; do not add short aliases or a second marker scheme.

A custom transport may move canonical bytes. It must not become a second effect
schema or a place for domain policy.

## Keep the core deterministic

Prefer pure functions for validation, authorization decisions, command
application, state transitions, and view-model derivation. Keep database I/O,
HTTP extraction, clocks, random IDs, queues, sockets, and host capabilities in a
thin explicit shell.

Do not introduce a trait merely to mock one function. Add an adapter when a real
side-effect or ownership boundary exists. In a server-side Wasm isolate, expose
only the platform capabilities the application needs; do not assume ambient
filesystem, network, clock, randomness, storage, or process state.

## Prove behavior at its owner

Choose the cheapest boundary that can expose the real failure.

| Behavior | Proof |
| --- | --- |
| Domain rule or state transition | ordinary Rust unit/integration test |
| Handler's typed effects | `hemx_test::run`, `run_async`, `run_result`, or `run_async_result` plus `EffectInspector` |
| Generated target update | generated target with `EffectInspector`; do not copy IDs |
| Static page or fragment structure | `HtmlInspector`, `target_html_document`, or `target_html_fragment` |
| Generated target/handle markers | `HtmlInspector::assert_target` / `assert_handle` |
| Axum route, middleware, headers, status, body, runtime asset, or native form fallback | enable `hemx-test`'s `axum` feature and use `hemx_test::axum` against a real `Router` |
| Process startup, readiness, socket, logs, cleanup | `TestProcess::builder` |
| JavaScript application, enhanced form submission, atomic target application, generated history navigation, island event handoff, focus, layout, SSE/WebSocket reconciliation | focused real-browser test |
| Portable target boundary | compile the actual consumer for its target and inspect the normal dependency graph |

Let `cargo check` and the real build step reject invalid templates and generated
contracts. A static render does not prove browser focus or history. A mocked
handler does not prove Axum extraction or middleware. A native test does not
prove a constrained target stayed parser-free.

## Refuse attractive detours

Do not add:

- copied runtime IDs, selectors, wire opcodes, generated files, or fingerprints;
- HTML strings assembled in Rust;
- a client state graph mirroring server-owned application state;
- framework-owned database, auth, routing, or deployment policy;
- stored HTML/effect output as authoritative state;
- a bespoke WebSocket/SSE payload when canonical effect bytes exist;
- offline, broker, queue, worker, or service layers without chosen semantics;
- microservices solely because the workspace or team became large;
- compatibility shims for APIs absent from the application's selected release.

## Finish with a concrete path

A useful Hemx answer or patch should identify:

- the `.heml` template and generated resource involved;
- the Rust handler/application-state path;
- the effect and integration response boundary;
- the test boundary that proves it;
- any missing external decision that prevents honest completion.

Prefer one complete interaction over framework scaffolding. If the existing
application already follows the canonical path, change only the part that owns
the requested behavior.
