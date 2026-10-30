# A proven action costs about 3.5 to 4 million gas

Measured 2026-10-30 on HideAndSeek (default, unoptimised compiler profile; EDR):

| action | gas |
| --- | --- |
| createMatch (in_range proof) | 4,038,261 |
| joinMatch (in_range proof) | 4,004,145 |
| move (move_on_ring proof) | 3,963,744 |
| answerGuess (answer_guess proof) | 3,408,407 |
| guess (no proof) | 64,595 |

Nearly all of it is the UltraHonk ZK verifier. The zero-knowledge target cannot be dropped (`evm-no-zk` would leak the witness, i.e. the hidden state), so this is the price of a hidden-information action on the EVM.

Consequences: such a game belongs on an L2 or a cheap chain, not on L1. A Revolver-Noir-style turn costs several proofs (an action plus each answer), so the signer's credits (`creditsGasMultiplier` in rocketh/config.ts) must be sized per proof, not per click. Worth measuring again with the production profile, and on every bb.js bump, since the verifier is generated.
