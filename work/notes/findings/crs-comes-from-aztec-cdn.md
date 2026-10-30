# The browser prover downloads its CRS from Aztec's CDN, with no option to self-host

Found 2026-10-30, bb.js 5.3.0 (`dest/browser/crs/net_crs.js`).

To prove, bb.js needs the structured reference string points for the circuit's size: 32 bytes per point, so about 130 KB for the 4096-gate `in_range` circuit (plus the G2 data). The browser build fetches them from `https://crs.aztec-cdn.foundation`, falling back to `https://crs.aztec-labs.com`. Both hosts are module constants; the `crsPath` option exists only in the node backends.

So a game served from IPFS still depends on a third party being up and reachable at proving time. That contradicts the "no server required" stance of the templates this one descends from.

Options, not yet chosen: cache the points in the service worker on first use (still needs one fetch, but survives an outage afterwards); vendor the points for the largest circuit into the static build and intercept the fetch in the service worker; or upstream an option to bb.js. Revisit before any public deployment.
