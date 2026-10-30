# check-omissions passes vacuously when HEAD and the stem have two merge bases

Found 2026-10-30 in template-zk-game. The script is shared machinery owned by `template-svelte` (the root of the tree), so the fix belongs there and was NOT made here.

`scripts/check-omissions.mjs` computes `git merge-base HEAD <stem>` and treats a path as "deleted here" only if it existed at that base. When the histories have a criss-cross (two equally recent common ancestors), `git merge-base` returns ONE of them, arbitrarily. Here the two were `4ec5fbd4` (the `integration` commit this repo was cloned from) and `594ee59c` (a template-svelte-shadcn commit that reached this repo through a stray fanout merge of jolly-roger `main`, see the other finding of this date). Git picked `594ee59c`, whose tree predates GreetingsRegistry, so all 20 deletions looked like paths this repo never had, and the check printed "covers all 0 path(s)": a pass it did not earn, exactly the failure the script exists to prevent.

Verified by hand with the right base: 20 deleted paths, all covered, nothing stale.

Fix (at the root): use `git merge-base --all` and treat a path as present at the base if it is in ANY of the bases' trees. Worth also refusing outright, rather than reporting 0, when the list is non-empty and the computed deletions are empty.
