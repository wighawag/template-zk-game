---
status: accepted
created: 2026-10-30
---

# Matches, turns and queries: what ZKMatches owns and what a game owns

`contracts/src/zk-game/ZKMatches.sol` is an abstract contract every game inherits. It owns the lifecycle that does not depend on rules; a game adds its actions as functions and uses the framework's helpers inside them.

## What the framework owns

- **Matches, not a world.** Many concurrent matches keyed by id, each created with a stake and joined by matching it. This is the deliberate difference from template-commit-reveal, whose contracts are one shared world per deployment.
- **A commitment per seat** for that player's hidden state, replaced only after a proven transition from the previous one. The chain of commitments is the player's history without revealing it.
- **Alternating turns in action points**, passed by `_settle` once the turn holder has none left and nothing is pending, or by `endTurn`. A game may hold the turn open with `_turnMayEnd` while it owes an action that is neither paid in points nor a query (added 2026-10-31 for duel-in-the-dark's forced move after a shot; the Remote Charge going off at the start of a turn is the next user).
- **Queries**: an action that needs other seats to answer with a proof before the turn continues (a guess here; a shot, a detonation or a listen in duel-in-the-dark). The waited-for seats are a bitmask, so a query to several responders is already expressible.
- **Deadlines on every wait**, and `claimTimeout`, callable by anyone: whoever was being waited for loses (the responder if a query is open, else the turn holder). Without this, a losing player could simply stop, and the stake would mean nothing.
- **Payout by pull**: winnings and refunds are credited, and only the player itself can withdraw, never a delegate.

## What a game owns

Its actions; building every proof's PUBLIC INPUTS from storage (commitments) and from the pending query (what was asked), with only the claimed result taken from the caller; and two hooks: what a valid starting commitment is, and how many action points a turn starts with.

## Deliberately not done yet

- **More than two seats.** Waiting is written over a bitmask, but who wins a timeout and who plays next are the two-seat answers. A 3-4 player mode extends them.
- **Simultaneous turns.** The plan called for a stub game to keep that seam honest; it was not written. The seam today is `_requireTurn` / `_settle`; a simultaneous policy would replace both. Revisit when a game needs it rather than guessing its shape.
- **Who goes first** is a coin flip on `block.prevrandao`, which a proposer can bias by one bit. Fine for that, never for anything that pays.
- **A query restarts the turn holder's clock** when answered (`turnDuration` from the answer), rather than resuming the time they had left. Simpler, and nobody is penalised for time spent waiting.
