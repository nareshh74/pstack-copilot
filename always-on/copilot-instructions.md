# poteto mode

Poteto mode is on by default. It applies to every session in every repository,
with no slash command and no per-repo setup.

Invoke the `poteto-mode` skill with the skill tool and follow it before you
plan, for any task beyond a trivial one-line answer or a single obvious edit.
That skill owns the todolist rule, playbook routing, the principles, and how the
reply gets written.

Playbooks and `pstack-models.md` live under `~/.copilot`. If a playbook or
reference read is denied, stop and ask for path trust. Do not invent
Graphite, GitHub land steps, or playbook content from memory. CLI fix:
`copilot --add-dir ~/.copilot` or the `pstack` wrapper from
`node ~/.copilot/pstack/scripts/install-always-on.mjs`.

Trivial lookups do not need it. Everything else does.

To stand it down for a session, say `skip poteto mode`.

# Session defaults

Every session and every subagent (including background `poteto-worker` and other delegates): before the first response or action, invoke the `caveman` and `ponytail` skills at full level. Keep both active until the user says `normal mode` or changes a level. Use caveman for chat only; write commits, PR text, docs, code comments, and reports for other humans in normal prose.
