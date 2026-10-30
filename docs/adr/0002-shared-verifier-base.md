---
status: accepted
created: 2026-10-30
---

# Verifiers share one generated base, deployed once

`circuits/scripts/generate.ts` splits each bb.js verifier into `contracts/src/verifiers/HonkVerifierBase.sol`, shared by every circuit, and a small `<Name>Verifier.sol` holding only that circuit's sizes and verification key. The base's two linked libraries (`RelationsLib`, `ZKTranscriptLib`) are deployed once, deterministically, and every verifier links to them.

## Why

bb.js 5.3 emits a single file per circuit, but only its head (four constants and `library HonkVerificationKey`) depends on the circuit. Everything after the second `pragma` is byte-identical across circuits: checked with a 1-public-input circuit against a 2-public-input one. That shared part contains two libraries with `public` functions, which Solidity links rather than inlines, so a verifier cannot be deployed without deploying them first.

Emitting the file as-is per circuit would compile N copies of the same libraries under the same names (ambiguous artifacts), and deploy N copies of 22 KB of identical code. One base fixes both, and a game with many circuits (one per action) pays for the libraries once.

## Guard rails

- Generation FAILS if a later circuit produces a different shared part: the assumption is checked on every run rather than trusted.
- Generation fails if bb.js changes the layout the split relies on (two pragmas, `contract HonkVerifier` last, the key in the head).
- The generated files are excluded from prettier and from the two lint rules they trip, so they stay byte-for-byte what the generator wrote and a rerun reproduces them exactly.

## Size

The verifier contract is ~24.1 KB unoptimized and ~19.5 KB with the production profile, against the 24,576-byte limit; the libraries are ~11.9 KB and ~10.8 KB. Close to the limit in the default (test) profile, so it is recorded as a finding to watch on every bb.js bump.
