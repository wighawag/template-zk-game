---
status: in-progress
created: 2026-10-30
---

# Plan: template-zk-game and the duel-in-the-dark remake

## Goal

A template for onchain games whose players keep part of their state hidden and prove facts about it in zero knowledge (Noir), and duel-in-the-dark rebuilt on it: a digital, re-themed take on the card game *Revolver Noir* (alien versus humans on a ship), as close to the card game's rules as possible before exploring variations.

The original duel-in-the-dark (2023) used circom + snarkjs PLONK, one global 2-player game in a contract, and a Next.js front. It is the reference for "parity", not a codebase to port.

## Decisions already taken

- **Noir instead of circom.** UltraHonk via bb.js; no trusted-setup ceremony to run and keep.
- **Stem: jolly-roger's `integration` branch** (local signer + embedded chain), not template-commit-reveal: commit-reveal's contracts assume one global world per deployment, while these games are many concurrent matches. Signer delegation comes from `@etherplay/delegation`; useful web pieces from commit-reveal (lobby/seats, acquire) are borrowed deliberately, not inherited.
- **Alternating turns first** (that is the card game), but the turn policy is a seam so simultaneous commit-reveal variants can come later. A stub game keeps that seam honest from the start.
- **Stake: ETH** for now, behind an identity seam so NFT avatars can replace account identity later.
- **Signer delegation: yes**, a turn is several transactions and none should prompt the wallet.
- **Licence: AGPL-3.0-only**, public repo.
- **Do not build the framework in the abstract.** Extract what we believe is common while duel-in-the-dark is built; a second game is what will confirm the shape.

## Milestones

### A. Noir plumbing: DONE (2026-10-30)

Devshell pinning nargo; circuits workspace; prover over noir_js + bb.js in node and browser; generator emitting committed artifacts and verifiers (ADR-0002); deploy script; tests at every layer (Noir, prover, on-chain, browser, embedded chain); `/zk` demo. See README.zk-game.md.

### B. Match framework (template): DONE except as noted (2026-10-30)

Contracts (ZKMatches + HideAndSeek, ADR-0003), two more circuits, the shared client module, and the /hide-and-seek page. Not done: the simultaneous-turn stub game (deferred, see ADR-0003), an end-to-end browser test of the SENDING side (it mirrors routes/demo), and the precache decision (finding).


- Matches keyed by id: create with stake and an initial commitment, join, payout, abandonment by timeout.
- Private state as a commitment chain: every action proves old commitment -> new commitment.
- **Queries**: an action that needs the opponent to answer with a proof before the turn continues (a shot, a detonation, a listen). Addressed to a SET of responders, so 3-4 player variants do not need a redesign. Each has its own deadline; not answering forfeits.
- Turn policy: alternating with action points (built), simultaneous (stub test game only).
- Web: match lobby, per-match secret storage with backup/restore, proving progress, signer delegation.
- A tiny reference game so the template runs without duel's rules.

### C. duel-in-the-dark at parity (branch `noir` in duel-in-the-dark, stem template-zk-game)

8-room ship map (the old circom table is already the card game's graph), move + shoot + hit honesty, 2 lives, ETH stake, the ship SVG UI. Then merge into duel-in-the-dark `main` with `--allow-unrelated-histories`.

### D. The full card game

Action points (2, or 3 from the energised room), listen, traps (set, detonate, at most two), room effects, an upper and a lower deck. Generic pieces this forces (queries, forced moves inside an answer, public "room inaccessible" flags) go back up to the template.

### E. Later

NFT avatars as identity; simultaneous-reveal variants; 3-4 player modes; mobile proving performance; self-hosted CRS.

## Protocol notes carried into B and D

- Action TYPES are public (the card game announces them); only their arguments are hidden. The chain therefore counts action points.
- A shot reveals the shooter's room, so the shoot table is public map data; the shooter's forced move re-hides them.
- A trap is the setter's own secret and is revealed on detonation, so it is the same pattern as a shot: no comparison of two players' secrets is ever needed.
- An answer may carry a state transition (a forced move into another room), and can keep it hidden.

## duel-in-the-dark game decisions (2026-10-30)

The rules themselves are not written down here: the game follows the card game, referred to as inspiration. Only what this re-theme and its digital form had to DECIDE is recorded.

- **Theme**: alien versus humans on a ship; both sides play by the same rules for now (the card game is symmetric).
- **Rooms**, with the old ship SVG's numbering, which is already the card game's room graph: 1 Cryo Bay, 2 Bridge, 3 Science Lab, 4 Observation Gantry (upper deck); 5 Galley, 6 Mess Hall, 7 Cargo Hold, 8 Engine Room (lower deck). Effects: Coolant Purge (Cryo Bay), Maintenance Chute (Science Lab), Drop Down (Gantry), Stim Rations (Galley), Motion Tracker (Cargo Hold), Remote Charge (Engine Room).
- **Remote Charge does NOT count toward the two-trap limit.**
- **Maintenance Chute is applied as the card says and nothing more**: the forced move is silent, and only a resulting hit (Engine Room purged) is public.
