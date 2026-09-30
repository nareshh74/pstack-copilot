---
name: sync-upstream
description: Pull the latest pstack changes from cursor/plugins main into this Copilot port without breaking Copilot compatibility. Subtree-merges upstream's pstack/ folder, resolves conflicts by taking upstream's generic content and reapplying the Copilot CLI and Azure DevOps port rules, then verifies and opens a GitHub PR. Use for sync-upstream, "pull upstream", "sync with cursor", "update from cursor/plugins".
---

# Sync upstream

This repo is a fork of `cursor/plugins` that keeps only `pstack/`, lifted to the repo root, and ports it to Copilot CLI and Azure DevOps. The immediate GitHub parent (`doublexmax/pstack-copilot`) is not the source. Sync from `cursor/plugins` directly.

The port edited the shared skills inline. There are no Copilot-only copies to protect. So "take upstream for generic skills" breaks Copilot: a `--theirs` resolution once gave 73 checker violations. Every conflicted or upstream-changed file needs upstream's content with the port rules reapplied.

## Steps

1. **Preflight.** Require a clean tree on `master`. Add the remote once, then fetch:
   ```
   git remote add cursor https://github.com/cursor/plugins.git   # skip if present
   git fetch cursor main
   git log --oneline master..cursor/main -- pstack    # what is new
   ```
   Nothing listed means nothing to sync. Stop and say so.
2. **Merge on a branch.**
   ```
   git switch -c sync-upstream-<yyyy-mm-dd>
   git merge --no-commit --no-ff -X subtree=pstack cursor/main
   ```
   `-X subtree=pstack` maps upstream's `pstack/` onto the root and ignores the other plugins. Record the merge base: `git merge-base HEAD cursor/main`.
3. **Inventory.** List what upstream changed (`git diff --name-status <base> cursor/main -- pstack`), the conflicted paths (`git diff --name-only --diff-filter=U`), and the clean-merged paths. Clean-merged files still need a port pass, because new upstream text arrives with Cursor-isms.
4. **Resolve each file** with [references/port-rules.md](references/port-rules.md). Upstream wins on generic content: wording, new or removed steps, restructuring. The port wins on anything platform-specific: tool names, models, paths, VCS mechanics. Useful views for a path `P`:
   - `git show :1:P`, `:2:P`, `:3:P` for base, ours, theirs
   - `git diff <base> cursor/main -- pstack/P` for exactly what upstream changed

   Handle new upstream files the same way. Drop Cursor-only features that have no Copilot equivalent, and add a row to the README "what changed from upstream" table. Delete files that upstream deleted.
   For more than about 10 files, fan out `general-purpose` task agents on disjoint file sets. Give each one the port-rules reference and forbid git writes. The coordinator stages the files.
5. **Keep scope.** Edit only files upstream changed, plus the direct fallout (README counts and tables, `models.default.md` roles, links to moved files). This check must print nothing unexpected:
   ```
   $up = git diff --name-only <base> cursor/main -- pstack | % { $_ -replace '^pstack/','' }
   git diff --name-only HEAD | ? { $_ -notin $up }
   ```
6. **Verify.**
   - `node scripts/check-copilot-port.mjs` must print `ok`.
   - Search for leftover conflict markers: `git grep -nE '^(<<<<<<<|>>>>>>>) '`.
   - Grep the leftover list in the port-rules reference. Only intentional mentions may remain: the README upstream table and links, and text cursors or loop variables named `cursor`.
   - Run a `code-review` agent over `git diff HEAD` with the port-rules reference. The checker misses semantic breaks. Past catches: ADO safety rules lost in a hunk (`deleteSourceBranch` on a PR with children), prose renamed away from a script's real output, script paths that only resolve inside this repo, and frontmatter keys the checker did not list yet. When a new class of break shows up, extend `check-copilot-port.mjs`.
7. **Commit the merge.** Run `git add -A` and confirm `git ls-files -u` is empty. Commit with a body that lists the upstream features ported, dropped, or changed and any model or default decisions. Keep the merge commit, and never squash it. Its second parent is what makes the next sync start from this upstream point.
8. **Open the PR on GitHub.** `origin` is `github.com/nareshh74/pstack-copilot`. gh may default to another host, so pin it:
   ```
   git push -u origin HEAD
   $env:GH_HOST = 'github.com'
   gh pr create --repo nareshh74/pstack-copilot --base master --fill-first --body-file <body.md>
   ```
   Fill the body from `.github/pull_request_template.md`. Merge with a merge commit, never squash or rebase, so the upstream parent survives.

## Decisions to surface, not make silently

- Upstream model default changes. Keep `models.default.md` per-role defaults and the four-vendor panel. Report new upstream slugs that Copilot also offers, as a follow-up.
- Dropping a whole upstream skill or feature.
- Any change to the trunk name. This fork's playbooks use `master`.

**Reply:** the upstream range synced (`<old>..<new>`), the conflicts resolved, the features ported or dropped, the checker and review results, and the PR link.
