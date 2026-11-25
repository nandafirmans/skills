---
name: resolving-merge-conflicts
description: "Use when you need to resolve an in-progress git merge/rebase conflict."
---

1. **See the current state** of the merge/rebase. Check git history, and the conflicting files.

2. **Find the primary sources** for each conflict. Understand deeply why each change was made, and what the original intent was. Read the commit messages, check the PRs, check original issues/tickets.

3. **Resolve each hunk.** Preserve both intents where possible. Where incompatible, pick the one matching the merge's stated goal and note the trade-off. Do **not** invent new behaviour. Always resolve; never `--abort`.

4. **Lint the resolved files.** Check that no conflict markers remain, then run the project's linter on the changed files only (e.g. `npx eslint <files>`). Fix anything the merge broke. Do **not** run tests, typecheck, builds, or formatters across the repo unless the user approves.

5. **Stage, then stop.** Stage only the resolved files (`git add <files>`). Do **not** commit, `merge --continue`, or `rebase --continue` until the user asks. In a submodule (e.g. `src/lib`), resolve inside it but never stage or commit the parent repo's submodule pointer unless the user asks.

6. **Report** each resolved file: which side was kept per hunk and why, any trade-off, the lint result, and what still needs manual checking. Then give the command to finish (`git commit --no-edit` or `git rebase --continue`) for the user to approve or run.
