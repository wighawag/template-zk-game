# A fanout run merged jolly-roger `main` before this repo declared its stem branch

Found 2026-10-30, while creating this repo.

The repo was cloned from jolly-roger's `integration` branch with the remote named `stem`, and about ten minutes later, before the `offshoot` config branch existed, an `offshoot-fanout` run elsewhere on the machine merged `jolly-roger@main` into this repo's `main` (commit 51b61a09). Without `fanout.config.json` it could only assume the stem's default branch.

No harm this time: the merge carried 15 lines that `integration` also had by then. But it would not be harmless in general, since `main` lacks the local-signer and embedded-chain work that `integration` adds, and a merge in that direction is a no-op only while `main` is an ancestor of `integration`.

Lesson for creating the next template in the tree: write the `offshoot` branch (with `stemBranch`) BEFORE adding a `stem` remote, or clone with a different remote name and rename it once the config exists.
