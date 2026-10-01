### Opening a PR

Invoked at the end of every other playbook.

**Worktree.** Work from a git worktree off the default branch (`master` on ADO repos that use it, otherwise `main`). Subagents inherit it. Multiple `task` calls on the same branch each get their own worktree, or `git fetch && git reset --hard origin/<branch>` between them. Dirty branch with unrelated work: patch out, fresh worktree, apply. Snarled worktree: reset from the default branch, redo minimally.

**Commits.** Commit liberally. Rebase into small, ordered commits before opening PRs. Each commit is a future PR: landable, ordered to tell the story. Amend when the fix belongs in a just-made commit. New commit when separable.

**PRs.** Apply the **unslop** skill to the diff, the PR description, and the commit bodies. Run `/no-comments` before review. Write every PR title, PR description, and commit body with **technical-writing**, then apply **unslop**. Apply every technical-writing layer except Diátaxis. Use one word for each action, keep articles, and avoid `-ing` when a plain verb works.

**Titles.** Use Conventional Commits in the form `type(scope): subject`. Use `feat`, `fix`, `docs`, `refactor`, `test`, `chore`, or `perf` as the type. Use the changed area, such as `pstack` or `poteto-mode`, as the scope. Keep the subject short and imperative. Name a real symbol when one carries the change. For example, `fix(pstack): retarget opening-a-pr babysit trigger`. Do not add a trailing period.

**Descriptions.** The PR body is a briefing, not the lab notebook. A reviewer who has the diff should learn why the change exists, what is out of scope, and how you proved the change works. The squash commit body is the PR body. If the body would make the squash commit longer than about 40 lines, cut the body.

Use these sections in order. Drop a section when it has nothing to say.

- `## Why`. State the intent and approach in one or two short paragraphs. Do not list SHAs or rebase genealogy. Do not add a "based on master" preamble.
- `## Scope`. Use bullets to list real symbols and paths. Name both sides of a rename or retarget. State what is in and out only when the boundary matters. Do not write a file-by-file essay.
- `## Tradeoffs`. Name only rejected alternatives that a reviewer would otherwise ask about. Skip this section when there was no real choice.
- `## Blast Radius`. In one to three sentences, name who or what the change touches and why the change is safe or risky. State the continuing cost if the default branch stays red without the fix.
- `## Verification`. Name each real run path and its outcome. For a performance change, report one primary number with its unit in `before -> after` form. Link the arena or swarm directory for the remaining evidence. Do not include sample-size methodology, swarm recitals, or metric tables.

After these sections, attach videos or screenshots when they prove a claim. Do not paste full SHAs, swarm or arena lane recitals, lever-correction essays, file-by-file checklists, or `CLEAN` verdicts. Put these details in a linked artifact. Do not use `## Summary` or `## Test plan` boilerplate. A commit body does not restate its subject.

**Forge.** Use Azure DevOps for PR operations. Read `../references/ado.md` before the first PR operation and keep that mechanics choice for create, edit, view, watch, and merge. Create PRs with the repo's ADO path or `az repos pr create`, and read or mutate them with the ADO MCP tools named in the reference. Use ADO tooling only.

**Size and chains.** Prefer five narrow PRs to one large PR. A chain is a target-ref chain. The root PR targets `master`. Each child branch rebases onto its parent's exact tip and its PR targets `refs/heads/<parent-branch>`. Create a child PR with ADO using that target branch. Retarget an existing child with `repo_pull_request_write action=update targetRefName=refs/heads/<parent-branch>`. Branch from the default branch only for independent work. Rebase on the default branch before substantial chain work. ADO does not retarget children, has no stacking tool, and autocomplete must be armed only on the frontier; see `../references/ado.md` before building a chain.

**Readiness.** Open every PR ready, never as a draft. If a PR still opens as a draft, update it to ready before asking for review. Run `repo_pull_request action=get` before you refer to PR status.

**Babysit.** Opening a PR does not start a babysit. Post the URL and keep building. Finish the phase or chain first. Run a separate babysit pass only when the user asks for one after the whole chain exists. A babysit for each new PR stalls the build and spends checks on commits that later waves restart. Push back when feedback drifts from intent.

A subagent that opens a PR runs `interrogate`, applies **unslop** and `/no-comments`, and posts the URL (`https://msazure.visualstudio.com/<project>/_git/<repo>/pullrequest/<id>`). Then it returns to the parent without babysitting, unless it is an Autopilot-full or Autopilot-stack owner. That owner's brief assigns the babysit loop and is the ask `playbooks/babysit.md` waits for. The owner starts the loop after its code-ready report and reports merge-ready or STACK-READY as its playbook says. The rules here and in `playbooks/babysit.md` that hold babysitting until a whole chain is built do not apply to that owner.
