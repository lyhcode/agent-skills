---
name: iterm2-automation
description: "iTerm2 automation, triggers, and Python scripting. Use this skill when the user wants to set up triggers (auto-highlight errors, run commands on pattern match, send notifications), write Python scripts for iTerm2 (custom status bar components, daemons, session management), use escape codes for terminal control, or automate any aspect of iTerm2 behavior. Also trigger when the user wants to create alerts based on terminal output, auto-respond to prompts, highlight log patterns, or build custom toolbelt components."
---

# iTerm2 Automation & Scripting

Help the user automate iTerm2 with triggers, Python API scripts, and escape codes.

## User's Environment

- iTerm2 3.6.8 on macOS, using zsh
- Python scripts directory: `~/Library/Application Support/iTerm2/Scripts/`
- No existing Python scripts

## Triggers

Triggers are regex-based rules that fire actions when matching text appears in terminal output. They're the simplest form of automation — no coding required.

### Configuration Path

Settings > Profiles > Advanced > Triggers > Edit

Each trigger has:
- **Regex** — ICU regular expression (matched line-by-line)
- **Action** — What to do when matched
- **Parameter** — Action-specific value
- **Instant** — Fire immediately without waiting for newline (useful for password prompts)

### Common Trigger Patterns

**Highlight errors in red:**
- Regex: `(?i)\b(error|fail|fatal|exception)\b`
- Action: Highlight Text
- Parameter: Red text color

**Notification on build completion:**
- Regex: `(BUILD SUCCESSFUL|Build complete|✓ Done)`
- Action: Post Notification
- Parameter: "Build finished"

**Auto-fill sudo password prompt:**
- Regex: `^\[sudo\] password`
- Action: Open Password Manager
- Instant: checked

**Mark deployment events:**
- Regex: `Deployed to (production|staging)`
- Action: Set Mark + Post Notification

**Highlight IP addresses:**
- Regex: `\b\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}\b`
- Action: Make Hyperlink

For the full list of all 26 trigger actions, read `references/trigger-actions.md`.

### Trigger Tips

- Triggers process the last 3 wrapped lines by default (adjustable in Advanced Settings)
- Use capture groups `\1`, `\2` in parameters to reference matched text
- "Stop Processing Triggers" action prevents subsequent triggers from firing — useful for priority ordering
- Triggers are profile-specific; use dynamic profiles to share triggers across contexts

## Python API

iTerm2's Python API enables deeper automation than triggers. Scripts can create windows, monitor sessions, build custom status bar components, and respond to events.

### Script Types

**Simple Scripts** — Run once and exit:
```python
import iterm2

async def main(connection):
    app = await iterm2.async_get_app(connection)
    window = app.current_terminal_window
    if window:
        await window.async_create_tab()

iterm2.run_until_complete(main)
```

**Daemons** — Run continuously in the background:
```python
import iterm2

async def main(connection):
    async with iterm2.CustomControlSequenceMonitor(
        connection, "my-sequence", r'^trigger:(.+)$') as mon:
        while True:
            match = await mon.async_get()
            # Handle the matched sequence

iterm2.run_forever(main)
```

### Creating a Script

1. Go to Scripts > Manage > New Python Script
2. Choose environment (Basic or Full) and type (Simple or Daemon)
3. iTerm2 creates the file in `~/Library/Application Support/iTerm2/Scripts/`
4. Or create files there manually

### Custom Status Bar Component

```python
import iterm2

async def main(connection):
    component = iterm2.StatusBarComponent(
        short_description="K8s Context",
        detailed_description="Shows current Kubernetes context",
        knobs=[],
        exemplar="minikube",
        update_cadence=10,  # seconds
        identifier="com.example.k8s-context"
    )

    @iterm2.StatusBarRPC
    async def coro(knobs):
        import subprocess
        result = subprocess.run(
            ["kubectl", "config", "current-context"],
            capture_output=True, text=True
        )
        return result.stdout.strip() if result.returncode == 0 else "N/A"

    await component.async_register(connection, coro)

iterm2.run_forever(main)
```

After registering, the component appears in Settings > Profiles > Session > Configure Status Bar.

### Useful Python API Patterns

Read `references/python-api-quickref.md` for common patterns including:
- Session monitoring (react to new sessions, profile changes)
- Sending keystrokes/text to sessions
- Creating window arrangements
- Theme/appearance detection and response
- Custom control sequences

## Escape Codes

iTerm2 supports proprietary escape codes for programmatic control from the command line. These work in any script or shell command — no Python API needed.

### Common Escape Codes

**Set badge text:**
```bash
printf "\e]1337;SetBadgeFormat=%s\a" $(echo -n "mytext" | base64)
```

**Post notification:**
```bash
printf "\e]9;%s\a" "Task complete"
```

**Set terminal title:**
```bash
printf "\e]0;%s\a" "My Title"
```

**Change profile on the fly:**
```bash
printf "\e]1337;SetProfile=%s\a" "SSH Profile"
```

**Set cursor shape:**
```bash
printf "\e]1337;CursorShape=%d\a" 1  # 0=block, 1=vertical bar, 2=underline
```

**Show progress bar:**
```bash
printf "\e]1337;Progress=%d\a" 50  # 0-100, or -1 to hide
```

**Report current directory (for shell integration features):**
```bash
printf "\e]1337;CurrentDir=%s\a" "$PWD"
```

**Set user variable:**
```bash
printf "\e]1337;SetUserVar=%s=%s\a" "myvar" $(echo -n "value" | base64)
```

For the complete escape code reference, read `references/escape-codes.md`.

## Workflow

When the user wants to set up triggers:
1. Understand what pattern they want to match and what action to take
2. Help them build the regex and choose the right action from the 26 available
3. Guide them through Settings > Profiles > Advanced > Triggers, or create a dynamic profile with triggers embedded

When the user wants a Python script:
1. Determine if it's a one-time script or a daemon
2. Write the script and save to `~/Library/Application Support/iTerm2/Scripts/`
3. Test by running from Scripts menu or via `Scripts > script-name`
4. For daemons, optionally enable auto-launch: Scripts > Manage > check "AutoLaunch" for the script

When the user wants escape code automation:
1. Identify the right escape code from the reference
2. Create shell functions or aliases in `~/.zshrc` for convenient access
3. Test in the terminal
