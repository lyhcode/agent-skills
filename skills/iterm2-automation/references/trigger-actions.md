# iTerm2 Trigger Actions Reference

All 26 trigger actions available in iTerm2.

## Text & Visual Actions

| Action | Parameter | Description |
|--------|-----------|-------------|
| **Highlight Text** | Text/background color | Colors the matched text |
| **Highlight Line** | Text/background color | Colors the entire line |
| **Annotate** | Annotation text (supports `\1` captures) | Adds a floating annotation to the matched line |
| **Set Mark** | — | Places a navigation mark at the line |
| **Set Named Mark** | Mark name | Places a named mark (navigable by name) |
| **Fold to Named Mark** | Mark name | Collapses lines between named mark and this trigger into a fold |

## Notifications & Alerts

| Action | Parameter | Description |
|--------|-----------|-------------|
| **Post Notification** | Notification text | Sends a macOS notification |
| **Bounce Dock Icon** | — | Bounces iTerm2 icon in the Dock |
| **Ring Bell** | — | Triggers the terminal bell |
| **Show Alert** | Alert text | Shows a modal alert dialog |

## Text Manipulation

| Action | Parameter | Description |
|--------|-----------|-------------|
| **Send Text** | Text to send | Sends text to the terminal as if typed |
| **Inject Data** | Data to inject | Injects data into the terminal output stream (appears but isn't actually received from the process) |

## Session & Profile

| Action | Parameter | Description |
|--------|-----------|-------------|
| **Set Title** | Title text | Changes the session title |
| **Change Style** | Style properties | Changes text style (bold, italic, etc.) |
| **Make Hyperlink** | URL (supports `\1` captures) | Makes matched text a clickable link |

## Command Execution

| Action | Parameter | Description |
|--------|-----------|-------------|
| **Run Command** | Shell command | Runs a command in a subprocess (not in the terminal session) |
| **Run Coprocess** | Command | Starts a coprocess that can read terminal output and write input |
| **Run Silent Coprocess** | Command | Like coprocess but output is hidden |

## Shell Integration Actions

These work best with shell integration installed:

| Action | Parameter | Description |
|--------|-----------|-------------|
| **Capture Output** | — | Captures the command's output for the Captured Output toolbelt |
| **Prompt Detected** | — | Marks this line as a prompt (useful when shell integration can't detect it) |
| **Report Directory** | Directory path | Sets the current directory for directory tracking |
| **Report User & Host** | `user@host` | Sets current user and hostname for profile switching |

## Scripting Actions

| Action | Parameter | Description |
|--------|-----------|-------------|
| **Invoke Script Function** | Function call | Calls a registered Python API function |
| **Set User Variable** | `name=value` | Sets a user-defined variable |

## Control Flow

| Action | Parameter | Description |
|--------|-----------|-------------|
| **Stop Processing Triggers** | — | Prevents any further triggers from being evaluated for this line |
| **Open Password Manager** | Account name (optional) | Opens iTerm2's password manager |

## Tips for Trigger Design

- Order triggers from most specific to least specific, with "Stop Processing Triggers" after definitive matches
- Use `(?i)` at the start of regex for case-insensitive matching
- Capture groups: `(pattern)` in regex, reference as `\1`, `\2` in parameters
- The "Instant" flag makes the trigger fire character-by-character instead of waiting for a complete line — essential for password prompts but uses more CPU
- Triggers are evaluated per profile; different profiles can have different trigger sets
