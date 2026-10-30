---
status: accepted
created: 2026-10-30
---

# One proving backend: bb.js produces the proofs, the keys and the verifiers

Every proof, verification key and Solidity verifier in this template comes from `@aztec/bb.js`, in node (tests, the generator) and in the browser alike. The native `bb` CLI is not used anywhere, and `nargo` is used only to compile.

## Why

A proof is only accepted by a verifier generated for the same proof format, and Noir's toolchain spans three independently versioned artifacts: `nargo` (compiles), `@noir-lang/noir_js` (executes, i.e. builds the witness) and the backend (proves). With a CLI backend as well, there would be four, and the CLI and the browser's bb.js drifting apart fails at proving time with an unreadable error.

The evidence that this is not hypothetical, gathered when the template was created (2026-10-30, the day after Noir 1.0.0): the official nargo-to-bb map that `bbup` reads (`barretenberg/bbup/bb-versions.json`) had no entry for 1.0.0 or any 1.0.0-rc; the native `bb` was at 6.0.0 while the newest published `@aztec/bb.js` was 5.3.0. There was no documented pair to install, so the pair had to be chosen and then PROVEN to work, which is far easier with one backend than two.

## What was verified

nargo 1.0.0 + noir_js 1.0.0 + bb.js 5.3.0: proofs generated and verified with both the native and the wasm backends of bb.js, verified on chain by the bb.js-generated verifier, and proved in headless chromium without cross-origin isolation. The test suites that establish this are listed in AGENTS.md; they are the gate for any version bump.

## Consequences

- The three versions are pinned together (flake.nix; exact versions in circuits/package.json) and bumped together.
- Generating verifiers needs node and bb.js, not a native binary, so the generator runs anywhere the tests run.
- bb.js decides the verifier's layout. The generator depends on it (ADR-0002) and fails loudly if it changes.
