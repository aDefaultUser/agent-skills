# agent-skills

My custom user-level agent skills for Codex, Claude Code, and Antigravity.

## Layout

Each skill is a self-contained folder with a `SKILL.md` and any supporting files it needs.

`skills/` holds the shared version of each skill.

- [babysit-pr](skills/babysit-pr/SKILL.md)
- [council](skills/council/SKILL.md)
- [file-pr](skills/file-pr/SKILL.md)
- [full-send](skills/full-send/SKILL.md)
- [full-stop](skills/full-stop/SKILL.md)
- [leaving-pr-comments](skills/leaving-pr-comments/SKILL.md)
- [model-failover](skills/model-failover/SKILL.md)
- [modernize](skills/modernize/SKILL.md)
- [reverse-engineer-anything](skills/reverse-engineer-anything/SKILL.md)
- [sol-loop](skills/sol-loop/SKILL.md)
- [takeover](skills/takeover/SKILL.md)

Install the [Claude Code version of Reverse Engineer Anything](variants/claude-code/reverse-engineer-anything/SKILL.md)
from `variants/claude-code/` when setting up that host.

## Installing a skill

If the host's personal skills directory is missing, create it first. Copy the whole skill folder
into that directory so the host can find its supporting files.

| Host        | Personal skills directory                  |
|-------------|--------------------------------------------|
| Codex       | `~/.codex/skills` or `~/.agents/skills`    |
| Claude Code | `~/.claude/skills`                         |
| Antigravity | `~/.gemini/config/skills`                  |

Run the commands for your host from the repo root.

```sh
# Codex
cp -a skills/babysit-pr ~/.codex/skills/

# Claude Code (use the variant where one exists)
cp -a skills/babysit-pr ~/.claude/skills/
cp -a variants/claude-code/reverse-engineer-anything ~/.claude/skills/

# Antigravity
cp -a skills/babysit-pr ~/.gemini/config/skills/
```

`cp -a` keeps file modes, so bundled scripts such as `babysit-pr/scripts/pr-wait` stay executable.
Restart the agent or start a new session if a newly copied skill does not show up.

## Dependencies

Some skills depend on tools such as the `gh` CLI for the PR skills, Python 3 for
`babysit-pr/scripts/pr-wait`, T3 Code orchestration tools for `model-failover`, or a configured
REA MCP server for `reverse-engineer-anything`. The `agents/openai.yaml` files provide Codex
metadata. Check the skill's `SKILL.md` for requirements before using it on a new machine.
