# Port rules: upstream pstack to this Copilot port

Apply these rules to every file that upstream added or changed. Give this file to any agent that resolves files.

## Goal per file

The final file is upstream's new content and structure, with every Copilot and Azure DevOps adaptation reapplied.

- Upstream wins on generic content: wording, prose tightening, new steps, removed steps, new guidance, restructuring.
- The port wins on anything platform-specific: tool names, model names, paths, VCS commands, Copilot mechanics.
- When upstream deletes a section that the port adapted, delete it.
- When upstream adds text with Cursor-isms, port that text with the rules below.
- Do not keep old upstream text only because the port had adapted it.
- Do not invent new features.
- Keep port-only safety text unless upstream removed the matching upstream text. Examples are the ADO rules in shipping, babysit, and autopilot: `deleteSourceBranch` on PRs with children, arming only the frontier PR, the `autoCompleteSetBy` check, and the policy-list verdict.

## Mapping

| upstream (Cursor) | here (Copilot) |
| --- | --- |
| `Task` tool, Task subagent | the `task` tool, task subagent |
| `subagent_type: generalPurpose` | `agent_type: "general-purpose"` |
| `subagent_type: poteto-agent` | `agent_type: "poteto-worker"` |
| `readonly: true` | `agent_type: "explore"` (reads only, no MCP) |
| `readonly: false`, or agent mode because MCP is needed | `agent_type: "general-purpose"` |
| `run_in_background` | `mode: "background"` |
| `AskQuestion` | the `ask_user` tool |
| `/loop`, loop skill, `/goal` | autopilot mode, autopilot objective |
| `.mdc` rule files | `.md` |
| `~/.cursor/rules/pstack-models.mdc` | `~/.copilot/pstack-models.md` |
| `.cursor/` project paths | `.github/` (skills at `.github/skills/`) |
| `~/.cursor/` user paths | `~/.copilot/` |
| Cursor transcripts, transcript globs | `session_store_sql` |
| `/add-plugin`, "install the plugin" | register the skills directory |
| `/deslop`, deslop skill | the **unslop** skill, Code path |
| `control-cli`, `control-ui`, "the control skill" | project `.github/skills/verify-*` skills, or **create-verification-skill** |
| Cursor built-in `babysit` | `playbooks/babysit.md` |
| Cursor built-in `create-skill` | the **create-skill** skill in this repo |
| Graphite (`gt`, stacks, merge queue), `gh pr`, GitHub review threads and checks | Azure DevOps PR chains, `az repos`, the ADO MCP. Follow the existing port wording and `skills/poteto-mode/references/ado.md`. |
| trunk `main`, `origin/main` in playbooks and scripts | `master`, `origin/master` (the ADO trunk the playbooks target) |

Frontmatter may hold only `name` and `description`. Strip every other key, for example `disable-model-invocation`, `paths`, `mode`, `icon`, `color`, `reminder`, `alwaysApply`, and `globs`. `name` must equal the directory name. Skill names in prose stay lowercase kebab-case.

Scripts ship inside the skill folder. Reference them relative to the skill (`scripts/x.mjs`), never by a path that resolves only inside this repo. Keep prose in sync with what a script really prints.

## Models

A model choice here is two `task` tool fields, `model` and `reasoning_effort`, written `model / effort`. Upstream uses one Cursor slug. The per-role defaults and the four-seat panel in `models.default.md` are the source of truth. Keep four seats where upstream has three. When upstream adds a role, add it to `models.default.md` with a mapped default.
These mappings favor OpenAI and Anthropic, but are not an allowlist. Use another available model when it fits the role better.

| upstream slug | here |
| --- | --- |
| `claude-opus-*` | `claude-opus-5.5 / xhigh` |
| `gpt-5.6-sol-*` | `gpt-6.1-sol / xhigh` |
| `grok-*` | `gpt-5.6-sol-fast / high` |
| `claude-fable-*`, any prose or judgment seat | `claude-sonnet-5.5 / high` |

Update this table when `models.default.md` changes.

Replace upstream's `pstack-models.mdc` paragraph with this one. Adjust only the "Each spawn below" subject when upstream's sentence differs:

> Each spawn below names a role line in `~/.copilot/pstack-models.md` and a default. Set `model` and `reasoning_effort` from that line's `model / effort` value, or from the default if the file or the line is missing. Omit both when the value is `auto` or `inherit-parent`. If the `task` tool rejects a model, use the default and say so. If it rejects the default, use the closest valid model of the same family from its error message.

## Leftover grep

After resolving, grep the changed files for these patterns: `cursor`, `Cursor`, `gt `, `Graphite`, `gh pr`, `\.mdc`, `readonly`, `subagent_type`, `generalPurpose`, `/goal`, `-max\b`, `fable`, `Task tool`, `origin/main`, and any new upstream model slug. Each hit must be intentional. Examples are the README upstream table, a text cursor, or the TypeScript `readonly` keyword.

## Rules for delegated agents

- Edit only the files assigned to you. Report cross-file issues, for example a link to a deleted file, in your final message.
- Run no git command that writes: no add, commit, checkout, reset, merge, or stash. Read-only git is fine.
- Leave no conflict markers.
- Run `node scripts/check-copilot-port.mjs`. Your files must have zero violations.
- Final message: one line per file that names the upstream change you took and any judgment call.
