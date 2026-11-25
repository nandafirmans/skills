---
name: resolve-conflicts
description: "Use when you need to resolve an in-progress git merge/rebase/cherry-pick conflict. Resolves each hunk from both sides' intent, lints the changed files, stages them, then stops for approval before continuing."
---

1. **See the current state.** Run `git status` to see whether this is a merge, rebase or cherry-pick and which files conflict. Note the conflict type per file (both modified, deleted by us/them, added by both, binary).

   **Know which side is which:**
   - Merge: `ours` / `HEAD` = the branch you're on; `theirs` = the branch being merged in.
   - Rebase: the sides are **swapped** — `ours` / `HEAD` = the branch being rebased onto (upstream); `theirs` = your commit being replayed.
   - Cherry-pick: `ours` = current branch; `theirs` = the picked commit.

2. **Find the intent behind each side.** Scale the effort to the conflict: trivial conflicts (imports, formatting, adjacent lines) need only the diff. For real logic conflicts, read the commits that touched the hunk (`git log -p --merge -- <file>` during a merge, or the commit being replayed during a rebase) and, when available, the related PRs or tickets.

3. **Resolve each hunk.** Preserve both intents where possible. Where they are incompatible, pick the one matching the merge's stated goal and note the trade-off. Do **not** invent new behaviour.
   - If both sides are incompatible and the goal doesn't decide it, **stop and ask** about that hunk instead of guessing. Never run `--abort`.
   - **Lockfiles and generated files** (`package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`, codegen output): don't hand-merge. Resolve the source file (e.g. `package.json`) first, then regenerate with the project's tool.
   - **Binary files:** pick one side with `git checkout --ours|--theirs <file>` (mind the rebase swap) and say which.
   - **Delete vs modify:** decide whether the deletion or the change reflects the intent; `git rm` or keep the file accordingly, and explain why.

4. **Verify the resolved files.**
   - No conflict markers remain: `git diff --check` and `git grep -nE '^(<<<<<<<|=======|>>>>>>>)( |$)' -- <files>`.
   - Run the project's linter on the changed files only (e.g. `npx eslint <files>`). Fix anything the merge broke.
   - Do **not** run tests, typecheck, builds or formatters across the repo unless the user approves.

5. **Stage, then stop.** Stage only the resolved files (`git add <files>`). Do **not** commit, `merge --continue`, `rebase --continue` or `cherry-pick --continue` until the user asks. During a multi-commit rebase, stop again at each new conflict. In a submodule, resolve inside it, but never stage or commit the parent repo's submodule pointer unless the user asks.

6. **Report** each resolved file: which side was kept per hunk and why, any trade-off, files that were regenerated, hunks left for the user to decide, the lint result, and what still needs manual checking. Then give the command to finish (`git commit --no-edit`, `git rebase --continue` or `git cherry-pick --continue`) for the user to approve or run.
