# Integer inputs cost range checks; a counter compared only for equality should be a Field

Found 2026-10-31 building duel-in-the-dark's milestone D (its `work` branch, work/notes/findings/gas-per-duel-action.md, has the numbers).

Its transition circuits gained four turn counters in the public state, first declared `u32`. The circuits went from 2^9 to 2^12 and 2^13 gates. Noir range checks every integer input, and a 32-bit range check pulls in a lookup structure much larger than the rest of a small circuit. The rules only ever compared those counters for equality (`acted != turn`, `turn == sealed_turn + 1`), which needs no range at all, so they became `Field`s, and the circuits went back to 2^9 and 2^10. `u8` fields cost little: their range is small.

Bitwise operators on witnesses (`&`, `|`, `<<`, `>>` with a variable operand) have the same kind of cost: they are lookup-table black boxes. A two-bit mask read and updated arithmetically (`armed == 1 | armed == 3`, `armed + bit`, `armed - bit`) avoids them.

Rule of thumb for circuits on this template: a public value that is only compared for equality, or only incremented by the contract, is a `Field`; keep integer types for values the circuit orders or bounds, and prefer the narrowest. After any change to a circuit's inputs, read `LOG_N` in its generated verifier: a jump there is the symptom. The proving key, the browser download and the proving time all grow with it; on-chain verification cost does not much.
