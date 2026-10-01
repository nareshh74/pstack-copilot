---
name: sync-upstream
description: Pull the latest pstack changes from cursor/plugins main into this Copilot port without breaking Copilot compatibility. Subtree-merges upstream's pstack/ folder, resolves conflicts by taking upstream's generic content and reapplying the Copilot CLI and Azure DevOps port rules, then verifies and opens a GitHub PR. After the merge, refreshes the local install at ~/.copilot/pstack and proves Copilot CLI loads the new skills. Use for sync-upstream, "pull upstream", "sync with cursor", "update from cursor/plugins".
---

# Sync upstream

This repo is a fork of `cursor/plugins` that keeps only `pstack/`, lifted to the repo root, and ports it to Copilot CLI and Azure DevOps. The immediate GitHub parent (`doublexmax/pstack-copilot`) is not the source. Sync from `cursor/plugins` directly.

The port edited the shared skills inline. There are no Copilot-only copies to protect. So "take upstream for generic skills" breaks Copilot: a `--theirs` resolution once gave 73 checker violations. Every conflicted or upstream-changed file needs upstream's content with the port rules reapplied.

## Steps

1. **Preflight.** Require a clean tree on `main`, this repo's default branch. Add the remote once, then fetch:
   ```
   git remote add cursor https://github.com/cursor/plugins.git   # skip if present
   git fetch cursor main
   git log --oneline main..cursor/main -- pstack    # what is new
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
   gh pr create --repo nareshh74/pstack-copilot --base main --title "<title>" --body-file <body.md>
   ```
   Fill the body from `.github/pull_request_template.md`. Merge with a merge commit, never squash or rebase, so the upstream parent survives.
9. **Refresh the local install.** Do this only after the PR is merged. If it is still open, stop and tell the user to ask for the refresh after the merge. The install is a clone of `origin` at `~/.copilot/pstack`. Copilot CLI reads skills from its `skills/` folder, so `git pull` is the update.
   ```
   $p = "$HOME\.copilot\pstack"
   git -C $p status --porcelain                  # must be empty, else stop and report
   $old = git -C $p rev-parse HEAD
   git -C $p pull --ff-only origin main
   $new = git -C $p rev-parse HEAD
   ```
   Then:
   - Confirm `skillDirectories` in `~/.copilot/settings.json` lists `$p\skills`. If not, run `copilot skill add "$p\skills"`.
   - For each `$p\agents\<name>.agent.md` that differs from `~/.copilot/agents\<name>.agent.md`, check whether the installed file is the old repo version. Compare blob hashes, because a literal compare fails on CRLF checkouts:
     ```
     git -C $p hash-object --path agents/<name>.agent.md "$HOME\.copilot\agents\<name>.agent.md"
     git -C $p rev-parse "${old}:agents/<name>.agent.md"
     ```
     Equal hashes mean copy the new file over it. Different hashes mean the installed file has local edits. Skip it and report it.
   - Run `node "$p\scripts\install-always-on.mjs"`. It is idempotent.
10. **Prove the CLI loads the new skills.** A file on disk is not proof. Compare `copilot skill list` against the range `$old..$new`:
    - Each `skills/*/SKILL.md` that the range added must be listed. Each one it deleted must be absent: `git -C $p diff --name-status $old $new -- 'skills/*/SKILL.md'`.
    - Each description that the range changed must show its new text in the list: `git -C $p diff -U0 $old $new -- 'skills/*/SKILL.md' | Select-String '^\+description:'`.
    - `copilot skill list` must report no load failures.
    If no description changed, start a fresh session and invoke one skill whose body changed: `pstack -p "<question that only the new body answers>"`. The session that ran the sync loaded its skills at startup, so it can show stale skills. New sessions get the update.

## Decisions to surface, not make silently

- Upstream model default changes. Keep `models.default.md` per-role defaults and the four-vendor panel. Report new upstream slugs that Copilot also offers, as a follow-up.
- Dropping a whole upstream skill or feature.
- Any change to the trunk name inside skills. The playbooks target ADO repos whose trunk is `master`; this repo's own default branch is `main`.

**Reply:** the upstream range synced (`<old>..<new>`), the conflicts resolved, the features ported or dropped, the checker and review results, the PR link, and the install refresh (`$old..$new` and the skill-list evidence, or "pending merge").
