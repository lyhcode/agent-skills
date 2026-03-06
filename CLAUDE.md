# Agent Skills Repository

A collection of agent skills following the [Agent Skills specification](https://agentskills.io).

## Project Structure

```
agent-skills/
├── .claude-plugin/
│   └── marketplace.json       # Claude Code Plugin manifest
├── skills/
│   └── <skill-name>/
│       ├── SKILL.md           # Skill definition (required)
│       ├── scripts/           # Executable scripts (optional)
│       └── reference/         # Supporting docs (optional)
├── template/
│   └── SKILL.md               # Template for new skills
├── CLAUDE.md
├── README.md
└── LICENSE
```

## Conventions

- Skill directories use **kebab-case** naming
- Each skill must have a `SKILL.md` with YAML frontmatter:
  - `name`: required, max 64 chars, lowercase letters/numbers/hyphens only, cannot contain "anthropic" or "claude"
  - `description`: required, max 1024 chars, should describe what the skill does AND when to use it
- SKILL.md body should be < 5k tokens; put detailed reference in separate files
- Skills use progressive disclosure: metadata → instructions → linked files
- After adding a new skill, update `marketplace.json` and `README.md`
