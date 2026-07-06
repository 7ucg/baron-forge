---
name: fullstack
description: "Full-stack engineering across Baron's stack: Rust (tokio, prost, axum, FFI/bridges), C/C++ (native libs, JNI), Node.js (Baileys, Anthropic SDK bots, npm package publishing), Dart/Flutter (messenger UIs, dark-mode aesthetics, HTML/CSS/JS mockups → Flutter), and React/web frontends. Use for building, refactoring, wiring cross-language bridges (Rust↔Node↔Dart), designing architecture, fast debugging, and shipping packages. Pairs with the protocols and reverse-engineering skills."
---

# Full-Stack

Operator mode applies. Match Baron's existing patterns and naming; don't impose new conventions on his repos.

## Rust
- Async: tokio. Web: axum. Proto: prost/prost-build. Serde for JSON. thiserror/anyhow for errors.
- FFI/bridges: prefer `napi-rs` for Node bindings, `flutter_rust_bridge` for Dart, raw `extern "C"` + cbindgen for C ABI. (`whatsapp-rust-bridge-baron` lives here.)
- Build: `cargo build --release`; cross-compile with `cross` or explicit targets for the server (no GPU, CPU-only).
- Debug: `RUST_BACKTRACE=1`, `cargo clippy -- -W clippy::all`, `dbg!`, `tokio-console` for async stalls.

## C / C++
- Native libs + JNI glue. `gcc`/`clang`, `-fsanitize=address,undefined` for memory bugs, `gdb`/`lldb`, `valgrind`.
- For JNI: signatures must match `Java_pkg_Class_method`; mind `JNIEnv*`, local/global refs, and threading.

## Node.js
- Baileys (`baron-baileys-v2`) bots, Anthropic SDK agent brains (tool-use dispatch, role-based admin, per-chat history). Note: bot workloads run on the **API**, separate from the Max plan.
- Publish: bump `package.json`, `npm publish` under `~baron`. Keep changelogs real (grounded in actual diffs, not invented).
- Debug: `node --inspect`, `NODE_OPTIONS=--enable-source-maps`, narrow with targeted logging not blanket dumps.

## Dart / Flutter
- Messenger apps, WhatsApp-inspired structure, black-and-white dark-mode aesthetic. Workflow: prototype as HTML/CSS/JS mockup → translate to Flutter/Dart widgets.
- State: keep it simple unless told otherwise. `flutter run`, `flutter analyze`, devtools for jank.

## React / web
- Interactive HTML prototypes (glassmorphism, cyberpunk/HUD) → components. Self-contained single-file when it's a mockup.

## Cross-language bridges (his core pattern)
- Rust core ↔ Node (napi) ↔ Dart (frb). Define the boundary as protobuf or a thin C ABI; keep one source of truth for the schema. Serialize nodes/stanzas consistently across all three.

## Shipping checklist
1. Build clean (no warnings that matter). 2. Run/smoke-test the actual path. 3. Update changelog from real diffs. 4. Version bump. 5. Publish.
