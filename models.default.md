# pstack model configuration (defaults)

This is the checked-in default panel for the Copilot port. `setup-pstack` writes your
personal overrides to `~/.copilot/pstack-models.md`, outside this repo so `git pull` never
conflicts with it. Every skill reads the override file first and falls back to these values.

A model choice is two fields on the `task` tool, `model` and `reasoning_effort`.
Every row below gives both.

## The panel

The defaults favor OpenAI and Anthropic for most roles, not every task.
Choose another available model when it fits a role better. pstack's
second-opinion rule is "the same prompt against a different model, and
agreement is high-signal". Prefer a reviewer from another vendor.

| Role slot | `model` | `reasoning_effort` | Why |
| --- | --- | --- | --- |
| Fast code | `gpt-5.6-sol-fast` | `high` | Bulk implementation and mechanical edits. |
| Precise-spec code | `gpt-6.1-sol` | `xhigh` | A specified sequence to execute to the letter. |
| Prose and judgment | `claude-sonnet-5.5` | `high` | Explanation, synthesis, review, vague intent. |
| Hardest design | `claude-opus-5.5` | `xhigh` | Cross-cutting design, gnarly concurrency, subtle algorithms. |

## Per-role defaults

Aliases: `inherit-parent` and `auto` both mean the role runs on the parent chat model.
Implement them by omitting `model` and `reasoning_effort` on the `task` call.

```
feature, refactoring:                   gpt-5.6-sol-fast / high
bug-fix:                                gpt-6.1-sol / xhigh
perf-issue:                             gpt-6.1-sol / xhigh
hillclimb:                              gpt-6.1-sol / xhigh
judgment and prose:                     claude-sonnet-5.5 / high
hardest tasks:                          claude-opus-5.5 / xhigh
how explorer:                           gpt-5.6-sol-fast / high
how explainer:                          claude-sonnet-5.5 / high
why investigators:                      gpt-5.6-sol-fast / high
why synthesizer:                        claude-sonnet-5.5 / high
reflect tooling:                        gpt-6.1-sol / xhigh
reflect judgment, divergent, synth:     claude-sonnet-5.5 / high
arena runners:                          claude-sonnet-5.5 / high, gpt-6.1-sol / xhigh, gpt-5.6-sol-fast / high, claude-opus-5.5 / xhigh
arena cross-judge pool:                 claude-sonnet-5.5 / high, gpt-6.1-sol / xhigh, gpt-5.6-sol-fast / high, claude-opus-5.5 / xhigh
swarm workers:                          gpt-5.6-sol-fast / high
architect runners:                      claude-sonnet-5.5 / high, gpt-6.1-sol / xhigh, gpt-5.6-sol-fast / high, claude-opus-5.5 / xhigh
interrogate reviewers:                  claude-sonnet-5.5 / high, gpt-6.1-sol / xhigh, gpt-5.6-sol-fast / high, claude-opus-5.5 / xhigh
```

Panel roles are lists. One subagent runs per entry, alias entries included, so the list
length sets the fan-out. `arena cross-judge pool` is a list Arena selects one value from,
preferring a family different from the parent's.

## Long-context work

Copilot exposes `context_tier: "long_context"` on the `task` tool, separately from the
model. Set it when a delegate must read a large corpus rather than reaching for a
different model. All four models in the panel support it.

## Panel composition

The default panel uses two OpenAI and two Anthropic models. This is not
an allowlist; change a seat when another available model fits better.
