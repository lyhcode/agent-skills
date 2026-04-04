---
name: iterm2-shell-setup
description: "iTerm2 shell integration setup and terminal environment configuration. Use this skill when the user wants to install or troubleshoot shell integration, set up command tracking, configure directory history, enable SSH shell integration on remote hosts, set up tmux integration with iTerm2, or fix issues with prompt detection, marks, or automatic features not working. Also trigger when the user mentions 'shell integration', command completion notifications, or wants features like right-click file download, automatic alerts for long-running commands, or recent directory navigation."
---

# iTerm2 Shell Integration & Environment Setup

Help the user install, configure, and troubleshoot iTerm2's shell integration features, plus tmux integration and remote SSH setup.

## User's Environment

- iTerm2 3.6.8 on macOS, using zsh
- Shell integration: **not currently installed**

## What Shell Integration Enables

Shell integration is the foundation for many of iTerm2's best features. Without it, these features don't work:

- **Command marks** — Navigate between prompts with Cmd+Shift+Up/Down
- **Command status** — Red/blue marks showing success/failure of each command
- **Automatic profile switching** — Switch profiles based on hostname/user/path
- **Recent directories** — Frecency-sorted directory list in the toolbelt
- **Command history** — Searchable history across all sessions
- **File download** — Right-click to download remote files via SCP
- **Long command alerts** — Notification when a command finishes after N seconds
- **Captured output** — Filter and interact with command output
- **Auto Composer** — Native text editing for command input

## Installation

### Method 1: Automatic (Recommended)

iTerm2 can load shell integration automatically without modifying shell config files:

Settings > Profiles > General > Command > enable "Load shell integration automatically"

This injects the integration at session start without touching `~/.zshrc`.

### Method 2: Menu Install

iTerm2 > Install Shell Integration

This downloads the scripts and adds a source line to `~/.zshrc`.

### Method 3: Manual Install for zsh

Add to `~/.zshrc`:

```zsh
test -e "${HOME}/.iterm2_shell_integration.zsh" && source "${HOME}/.iterm2_shell_integration.zsh"
```

Then download the script:
```bash
curl -L https://iterm2.com/shell_integration/zsh -o ~/.iterm2_shell_integration.zsh
```

### Verification

After installation, check that it's working:
1. Open a new iTerm2 tab
2. You should see a blue triangle mark at the left of each prompt
3. Run `echo $ITERM_SESSION_ID` — should output a value
4. Check: `type iterm2_print_user_vars` should find the function

## Remote SSH Setup

To get shell integration features on remote hosts:

### Option A: Automatic SSH Integration (iTerm2 3.5+)

iTerm2 has built-in SSH integration that doesn't require installing anything on the remote host. Enable it via the `it2ssh` command or configure in Settings.

### Option B: Install on Remote Host

SSH into the remote host and run the appropriate install for its shell:

```bash
# For bash on remote:
curl -L https://iterm2.com/shell_integration/bash -o ~/.iterm2_shell_integration.bash
echo 'test -e "${HOME}/.iterm2_shell_integration.bash" && source "${HOME}/.iterm2_shell_integration.bash"' >> ~/.bashrc

# For zsh on remote:
curl -L https://iterm2.com/shell_integration/zsh -o ~/.iterm2_shell_integration.zsh
echo 'test -e "${HOME}/.iterm2_shell_integration.zsh" && source "${HOME}/.iterm2_shell_integration.zsh"' >> ~/.zshrc
```

### Supported Shells

| Shell | Script URL |
|-------|-----------|
| zsh | `https://iterm2.com/shell_integration/zsh` |
| bash | `https://iterm2.com/shell_integration/bash` |
| fish | `https://iterm2.com/shell_integration/fish` |
| tcsh | `https://iterm2.com/shell_integration/tcsh` |

## tmux Integration

iTerm2 can natively control tmux, showing each tmux window as a native iTerm2 tab/window. This gives you iTerm2's UI (scrollback, search, profiles) with tmux's persistence.

### How to Use

```bash
# Start new tmux session with iTerm2 integration
tmux -CC

# Attach to existing session with integration
tmux -CC attach
```

The `-CC` flag is the key — it enables iTerm2's native integration mode.

### Key Points

- Each tmux window becomes an iTerm2 tab
- Each tmux pane becomes an iTerm2 split pane
- Scrollback works natively (no tmux copy-mode needed)
- If you disconnect, just `tmux -CC attach` to reconnect
- The tmux dashboard (Shell > tmux > Dashboard) lets you manage sessions

### Configuration

Settings > General > tmux:
- "Open tmux windows as" — native tabs, native windows, or tabs in existing window
- "Automatically hide the tmux client session" — hides the raw tmux control session
- "Use tmux profile rather than profile of the connecting session" — use a separate profile for tmux sessions

## User-Defined Variables

Shell integration enables user-defined variables that can be displayed in badges, status bar, and used by scripts.

Add to `~/.zshrc` (after the shell integration source line):

```zsh
iterm2_print_user_vars() {
  iterm2_set_user_var gitBranch $((git branch 2> /dev/null) | grep \* | cut -c3-)
  iterm2_set_user_var kubeContext $(kubectl config current-context 2>/dev/null)
}
```

These variables become available as `\(user.gitBranch)` in badges and interpolated strings.

## Troubleshooting

Read `references/troubleshooting.md` for common issues and solutions.

## Workflow

When the user wants to set up shell integration:

1. **Check current state** — Look for `iterm2_shell_integration` in `~/.zshrc` and check if `~/.iterm2_shell_integration.zsh` exists
2. **Install** — Use the appropriate method based on user preference
3. **Verify** — Open a new tab and check for marks, `$ITERM_SESSION_ID`, and `iterm2_print_user_vars`
4. **Configure extras** — Set up user variables, enable long command alerts, configure recent directories toolbelt

When troubleshooting:
1. Check if the integration script is sourced in the shell config
2. Check if the script file exists and is not corrupted
3. Check for conflicts with other prompt customization tools (Starship, Powerlevel10k, oh-my-zsh themes)
4. Verify the correct shell is being used
