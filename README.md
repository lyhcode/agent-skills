# Agent Skills

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

A collection of agent skills by lyhcode, following the [Agent Skills specification](https://agentskills.io).

## Skills

| Skill | Description | Requires |
|-------|-------------|----------|
| [1password-cli](skills/1password-cli/) | Manage 1Password items, vaults, and secrets using the `op` CLI | [`op`](https://developer.1password.com/docs/cli/) |
| [awscli](skills/awscli/) | Manage AWS services using the AWS CLI (S3, EC2, IAM, Lambda, CloudFormation, ECS, and more) | [`aws`](https://aws.amazon.com/cli/) |
| [gandi-cli](skills/gandi-cli/) | Manage Gandi.net domains, DNS records, certificates, and email using the `gandi` CLI | [`gandi`](https://cli.gandi.net/) |
| [pbcopy](skills/pbcopy/) | Copy text, HTML, or images to the macOS clipboard using `pbcopy` and `osascript` | macOS |
| [syncthing](skills/syncthing/) | Manage Syncthing file synchronization via the REST API and CLI for device pairing, folder management, sync status, and versioning | [`syncthing`](https://syncthing.net/downloads/) |
| [synology-cli](skills/synology-cli/) | Manage Synology NAS via SSH using administrative CLI utilities (`synouser`, `synogroup`, `synoshare`, `synonet`, `synoservice`, `synowin`) | SSH access to DSM |
| [tailscale-cli](skills/tailscale-cli/) | Manage Tailscale VPN using the `tailscale` CLI for networking, file sharing, and service exposure | [`tailscale`](https://tailscale.com/download) |
| [textual-tui](skills/textual-tui/) | Build terminal user interfaces (TUI) with the Python Textual framework | Python 3.9+, [`textual`](https://textual.textualize.io/) |

## Installation

### npx skills (Recommended)

Install skills using the [`skills`](https://github.com/vercel-labs/skills) CLI, which supports Claude Code, Cursor, Codex, Gemini CLI, and more:

```bash
# Install all skills (interactive selection)
npx skills add lyhcode/agent-skills

# Install a specific skill
npx skills add lyhcode/agent-skills@pbcopy
```

### Manual

Copy a skill directory to your agent's skills folder:

```bash
# Claude Code (personal, available to all projects)
cp -r skills/<skill-name> ~/.claude/skills/<skill-name>

# Claude Code (project scope)
cp -r skills/<skill-name> .claude/skills/<skill-name>
```

## Contributing

1. Copy `template/SKILL.md` to `skills/<skill-name>/SKILL.md`
2. Edit frontmatter — `name` (required, max 64 chars, lowercase letters/numbers/hyphens) and `description` (required, max 1024 chars)
3. Write instructions (recommended < 5k tokens)
4. Add supporting files (`scripts/`, `reference/`) if needed, and reference them in SKILL.md
5. Register the skill in `.claude-plugin/marketplace.json`
6. Add the skill to the table in this README

## License

[MIT](LICENSE)
