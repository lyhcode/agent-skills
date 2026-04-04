# iTerm2 Shell Integration Troubleshooting

## No Marks Appearing at Prompts

**Symptoms:** No blue triangle marks, no command status indicators.

**Causes & Fixes:**
1. Shell integration not sourced — check `~/.zshrc` for the source line or enable auto-load in Settings > Profiles > General
2. Script file missing — verify `~/.iterm2_shell_integration.zsh` exists; re-download if needed
3. Custom prompt overriding — some themes (Powerlevel10k, Starship) rewrite PS1 after shell integration sets it up

**Powerlevel10k compatibility:**
Powerlevel10k is compatible with iTerm2 shell integration. Make sure shell integration is sourced *before* Powerlevel10k in `~/.zshrc`. If using Powerlevel10k's instant prompt, the shell integration source line should go after the instant prompt block but before `source ~/powerlevel10k/powerlevel10k.zsh-theme`.

**Starship compatibility:**
Starship replaces the prompt entirely. Shell integration should still work if sourced before Starship is initialized, but some features like prompt detection may not function correctly. Consider using iTerm2's trigger-based prompt detection as a fallback.

## Automatic Profile Switching Not Working

1. Confirm shell integration is installed on the *remote* host (not just locally)
2. Check that the profile has switching rules defined (Settings > Profiles > Advanced)
3. The hostname reported by shell integration must match the rule — check with `echo $HOST`
4. SSH through jump hosts may not propagate shell integration; install on each host in the chain

## Command History Not Populating

1. Shell integration must be active — history is only tracked for sessions with integration
2. Check Settings > General > Services > "Save copy/paste and command history to disk" is enabled
3. History is per-machine; SSH sessions track to the remote host's history

## `iterm2_print_user_vars` Not Called

1. The function must be defined *after* sourcing shell integration
2. Verify the function exists: `type iterm2_print_user_vars`
3. If using auto-load method, define the function in `~/.zshrc` anyway — the auto-load will call it if defined

## Recent Directories Not Showing

1. Open the Toolbelt: View > Show Toolbelt (or Cmd+Shift+B)
2. Enable "Recent Directories" in the toolbelt menu
3. Shell integration must be active for directory tracking

## Long Command Alert Not Firing

1. Settings > Profiles > Session > set "After N seconds of idle, post a notification"
2. This requires shell integration to detect when a command finishes
3. Background commands and commands that take less than the threshold won't trigger

## tmux Integration Issues

**"tmux -CC" shows raw control output:**
- This means iTerm2 isn't intercepting the control mode. Make sure you're using iTerm2 (not Terminal.app or another emulator)

**Tmux windows not appearing as tabs:**
- Check Settings > General > tmux for the window creation preference
- Try `tmux -CC new` instead of `tmux -CC attach` to rule out session state issues

**Scrollback not working in tmux:**
- Native scrollback only works in `-CC` mode
- Check Settings > General > tmux > "Sync scrollback history" is enabled

## SSH File Download Not Working

Right-click > "Download with scp from hostname" requires:
1. Shell integration on the remote host
2. SCP access to the remote host from your Mac
3. The hostname must be resolvable from your Mac
