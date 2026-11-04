# The browser prover downloads its CRS from Aztec's CDN, with no option to self-host

**Resolved 2026-11-04 (template-zk-game main 632be8a7):** the CRS is committed (`circuits/crs/index.ts`, written by `scripts/crs.ts`) and handed to bb.js with `skipSrsInit` + `srsInitSrs`, on its WASM build in node and the browser alike, so proving never touches the network. Measured on bb.js 5.3.0: the default is now 2^19 G1 points (16 MB) plus 4 MB of Grumpkin points, not the circuit's size; UltraHonk needs exactly the largest circuit's dyadic size (fewer fails with "too many points in MemBn254CrsFactory") and no Grumpkin points; compressed points are only accepted in 4 MB chunks, so the script decompresses once and commits uncompressed points (64 bytes each); in node the default backend is the native binary, which downloads into ~/.bb-crs regardless of what was passed, hence forcing WASM. Verified with `unshare -n`: prover tests, contract tests, and the browser client project.

Found 2026-10-30, bb.js 5.3.0 (`dest/browser/crs/net_crs.js`).

To prove, bb.js needs the structured reference string points for the circuit's size: 32 bytes per point, so about 130 KB for the 4096-gate `in_range` circuit (plus the G2 data). The browser build fetches them from `https://crs.aztec-cdn.foundation`, falling back to `https://crs.aztec-labs.com`. Both hosts are module constants; the `crsPath` option exists only in the node backends.

So a game served from IPFS still depends on a third party being up and reachable at proving time. That contradicts the "no server required" stance of the templates this one descends from.

Options, not yet chosen: cache the points in the service worker on first use (still needs one fetch, but survives an outage afterwards); vendor the points for the largest circuit into the static build and intercept the fetch in the service worker; or upstream an option to bb.js. Revisit before any public deployment.
