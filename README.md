# Agent Skills

A collection of agent skills by lyhcode, following the [Agent Skills specification](https://agentskills.io).

## Usage

Agent Skills 可在不同平台使用，各平台安裝方式不同：

### Claude Code (Plugin)

```
/plugin marketplace add lyhcode/agent-skills
/plugin install <skill-name>@lyhcode-agent-skills
```

### Claude Code (Manual)

將 skill 目錄複製到個人或專案的 skills 資料夾：

```bash
# 個人（所有專案可用）
cp -r skills/<skill-name> ~/.claude/skills/<skill-name>

# 專案（僅該專案可用）
cp -r skills/<skill-name> .claude/skills/<skill-name>
```

### Claude.ai

將 skill 目錄打包為 zip，到 Settings > Features 上傳。僅限個人使用，不會跨組織同步。

### Claude API

透過 Skills API (`/v1/skills`) 上傳，workspace 內所有成員可用。

## Skills

| Skill | Description |
|-------|-------------|
| [1password-cli](skills/1password-cli/) | Manage 1Password items, vaults, and secrets using the `op` CLI |
| [pbcopy](skills/pbcopy/) | Copy text, HTML, or images to the macOS clipboard using `pbcopy` and `osascript` |
| [awscli](skills/awscli/) | Manage AWS services using the AWS CLI (S3, EC2, IAM, Lambda, CloudFormation, ECS, and more) |

## Creating a New Skill

1. 複製 `template/SKILL.md` 到 `skills/<skill-name>/SKILL.md`
2. 編輯 frontmatter — `name`（必填，最多 64 字元，小寫字母/數字/連字號）和 `description`（必填，最多 1024 字元）
3. 撰寫指令內容（建議 < 5k tokens）
4. 如需更多資料，加入附屬檔案（`scripts/`、`reference/`），並在 SKILL.md 中引用
5. 更新 `.claude-plugin/marketplace.json` 註冊 skill
6. 更新本 README 的 skill 列表

## License

[MIT](LICENSE)
