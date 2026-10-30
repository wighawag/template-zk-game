# The inherited service worker precaches the prover: about 20 MB on a first visit

Found 2026-10-30, measured on a production build of template-zk-game.

The service worker inherited from jolly-roger precaches 87 immutable files, 19.9 MB, in the background on the first visit. About 12 MB of that is the prover, which every other part of the app goes out of its way to load lazily: `acvm_js_bg.wasm` (3.1 MB) and `noirc_abi_wasm_bg.wasm` (0.8 MB) from noir_js, and two ~4.1 MB chunks from bb.js that appear to carry the Barretenberg wasm twice (likely its single-threaded and threaded builds). The inherited offline-demo route adds a 5.9 MB page chunk, because the in-tab chain bundles every contract artifact, verifiers included.

It is not a bug: precaching is what lets a game work offline after one visit. It is a trade-off the template should make on purpose rather than by inheritance: 20 MB is a lot for a phone on a metered connection, for a page that may never prove.

Options: exclude the prover chunks from precaching and cache them on first use (runtime cache) instead; keep precaching only on the routes that play; or keep it and say so. Check also whether one of the two Barretenberg builds can be dropped (single-threaded is what static hosting gets anyway).
