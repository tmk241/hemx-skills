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
private example, an unpublished API, or copied generated output. This guidance
targets Hemx 0.5, Hemplate 0.4, and `hemplate-runtime` 0.3.

If auth, persistence, transport, replay, conflict, or deployment behavior is
not chosen, state the missing decision. Do not hide it behind a speculative
trait, provider, queue, or service. If required behavior fails on the selected
release, report the boundary and require a verified fix; do not hand-encode
around it or present an unreleased repair as available.

## Choose the smallest Hemx path

Classify the requested behavior before writing code.

| User-visible behavior | Default path |
| --- | --- |
| Full page or navigation | real link or GET form -> application query -> Hemplate page response with the generated build fingerprint |
| Form or control interaction | native form/control semantics -> generated form/handle -> `#[hemx::handler]` -> command -> generated target effect |
| Reusable presentation | typed Hemplate view, composed from nested views |
| Group of checked handlers | `#[hemx::component]` against the generated component contract |
| Server push | canonical effect batches over SSE; application-owned sockets only when needed |
| Isolated high-frequency browser behavior | explicit JavaScript island |
| Server-side Wasm isolate | normal Hemx core plus the platform adapter owned by the application |
| Durable/offline synchronization | only after event, replay, deletion, ordering, and conflict semantics exist |

Start with one server process and ordinary Axum application state unless a
measured availability, scaling, ownership, or isolation pressure requires more.
A larger codebase needs module boundaries before it needs network boundaries.

## Author at the semantic level

Use `.heml` templates to declare roots, slots, forms, handles, keyed targets,
classes, collections, and events. Let `hemx-build` generate their Rust vocabulary.
A normal build script uses the public `hemx-build` application builder; inspect the
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

For collections in Hemx 0.5, use the generated target's `set(&rows)` to declare
complete content, or `transition(before, after, changed)` with both key orders
and every new or changed row. Rows implement Hemplate and `hemx::KeyedPartial`.
These produce `Rows`/`RowsDelta` declarations, not insertion/movement commands.
A transition applies at its before order, is skipped at its distinct after
order, and rejects other orders; recover a lost baseline with a complete `set` or page
navigation. A generated keyed row target replaces one existing row; it does not
own collection membership. Do not copy the underlying order hashes or opcodes.

Only controls and explicit `data-hemx-param-*` attributes become request fields;
ordinary `data-*` stays DOM metadata. Drag keys use the reserved `__drag` field.
HTML Boolean attributes use presence, not string truth: `disabled="false"` is
still disabled. In Hemplate 0.4, use a Boolean binding such as
`+disabled="self.disabled"` to omit false; verify rendered/browser state.

A mutation form should still describe a real URL-encoded `POST`; Hemx adds the
generated handle and event contract and injects its hidden dispatch field. Use
multipart only when the selected release and the actual form fields require it.
Do not assemble template HTML in handler strings; render a Hemplate view.
Text helpers escape; only explicit trusted HTML introduces markup.

Hemplate 0.4 checks attribute contexts, not merely HTML escaping. Parse a whole
untrusted navigation URL with `hemplate::Url::parse` and handle rejection before
rendering; a typed URL occupies the entire attribute. A fixed safe prefix such
as `/users/` also permits escaped text interpolation. A checked `Url` does not
approve a destination or authorize code loading. Move dynamic data out of
`on*`, `style`, and `srcdoc` into data attributes/classes. Use
`hemx_axum::RuntimeScript` for interpolated `<script src>`; plain strings and
navigation URLs do not meet that resource context. Do not silence context errors
with unchecked trust constructors. Raw HTML remains an explicit typed trust
boundary with a known producer. Keep application URL policy and CSP as
additional boundaries, not replacements for correct rendering.

Keep routing, authorization, sessions, persistence, migrations, feature flags,
and domain policy in the application or their real integration crates.

`#[hemx::component]` checks a handler group against a generated template
contract. It is not a second renderer and is unnecessary for a purely
presentational view.

Keep a JavaScript island visibly local: the server may dispatch an explicit
Hemx event into the current root, while the island owns only the browser-only
behavior that consumes it (for example a native dialog, canvas, or
high-frequency gesture). An island may retain its own nodes and browser-only
state; do not cache server-owned nodes outside it across updates or mirror
business state. Verify identity and cleanup across morphs, replacements, and
navigation rather than assuming those paths are interchangeable.

For an Axum application, keep the reachable integration chain together: lower
the page through the generated page helper (for example `heml::page`), then use
`PageResponse` with the generated `BuildFingerprint`. Rendering Hemplate alone
does not lower Hemx resources. Serve the packaged `RuntimeJs`, extract an
`InteractionRequest`, and dispatch through the registry from `#[hemx::app]`.
The typed dispatch wrapper is `hemx_axum::Form<T>` (or imported `Form<T>`),
implementing `FromInteractionForm`, not Axum's `FromRequest`. `hemx::Form<T>` is
a form resource token, not that wrapper. `InteractionRequest` and
`InteractionForm` own HTTP extraction; do not parse the body twice or manually
register numeric handles.

Native action routes still belong to the application, including submitter
URL/method overrides. Share validation and one mutation operation between native
and enhanced requests: native GET returns HTML; successful native POST normally
returns 303 to a GET, and invalid input returns accessible HTML feedback.
Enhanced validation errors need a successful canonical effect response rendering
that feedback, not an HTTP failure the runtime cannot apply. Never serve effect
bytes as a native document. Preserve values and test focus/announcements on
failure. Native constraint validation and `submit` cancellation remain valid;
an enhanced POST failure must never trigger an automatic native POST retry,
because the command may already have committed.

In 0.5, generated action forms include `__h` and the build field `__b`. Registry
dispatch rejects missing/different builds with 409 before handlers run. Manual
`InteractionForm` dispatch must first call
`form.require_build(heml::BUILD_FINGERPRINT)?`. Extraction rejects cross-site
requests (and same-site by default) with 403; missing `Sec-Fetch-Site` passes.
These are not authentication, authorization, or complete CSRF protection. Keep
application security/body limits and explicit stale-page recovery; do not retry
an ambiguous mutation.

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
- in 0.5, use canonical `EffectBatch` ABI-3 bytes for SSE or application-owned
  sockets; rebuild templates, server, and runtime together, as ABI 2 is rejected;
- use fallible `batch.to_wire()?` and respect producer/browser byte and item
  limits; do not turn an encoding failure into invalid bytes or empty success;
- `data-hemx-ws` is rejected in 0.5; a custom socket feeds canonical bytes to
  `window.hemx.applyBatch` and owns connection lifecycle and recovery;
- reject malformed or incompatible payloads instead of guessing a fallback;
- keep effects scoped to the current generated root and let the page adapter
  attach its build marker instead of writing one into the template;
- use `InteractionRequest`/generated form dispatch for supported form and file
  inputs rather than bypassing media-type and body-limit handling;
- preserve the canonical `data-hemx-*` runtime vocabulary emitted by the build
  and adapter layers; do not add short aliases or a second marker scheme.

For replayable SSE in 0.5, use `hemx_axum::Live` with `Resume`. Publish under the
lock guarding the described state; read `Live::cursor` with that same state for
the page and snapshots. `Live::sse` resumes using `Last-Event-ID` or the page's
`after` query cursor, replays its bounded in-memory window, and invokes the
application's snapshot when continuity is unavailable (including restart or
lag). `Live::new` retains 1,024 batches; `Live::with_replay` selects the window.
Snapshots declare every shown collection with `set`; use a `Visit` if the state
cannot fit the wire budget. This is not durable/offline synchronization.

SSE coalescing is lossless: it preserves every operation in order, saving
envelopes/framing, not HTML payload or DOM operations. Side effects or differing
builds are barriers; runs over wire budgets retain their original frames.
A coalesced frame is one batch: application failure stops its remaining effects.
Supply valid transitions and replay/snapshot recovery; do not claim slow clients
skip intermediate updates or that a failed batch rolls back earlier effects.

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
| JavaScript application, enhanced form submission, batch ordering and failure boundaries, generated history navigation, island event handoff, focus, layout, SSE/WebSocket reconciliation | focused real-browser test |
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
