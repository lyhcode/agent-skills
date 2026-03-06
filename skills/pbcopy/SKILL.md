---
name: pbcopy
description: >
  Copy text to the macOS clipboard using pbcopy/pbpaste.
  Use when the user asks to copy, share, or transfer text data to the clipboard,
  or when generating output that the user will need to paste elsewhere
  (e.g., commands, code snippets, config blocks, URLs, tokens).
  Triggers on mentions of copy, clipboard, paste, pbcopy, or "copy to clipboard".
---

# Clipboard (pbcopy/pbpaste)

Automatically copy text to the macOS clipboard so the user can paste it directly.

## When to Use

- The user explicitly asks to copy something to the clipboard
- You generate a snippet (command, code, config, URL, token) that the user said they want to paste elsewhere
- The user asks you to "put it in my clipboard" or "copy that for me"

## Commands

```bash
# Copy text to clipboard
echo -n 'text to copy' | pbcopy

# Copy multi-line text to clipboard
cat <<'EOF' | pbcopy
line 1
line 2
EOF

# Copy file contents to clipboard
pbcopy < /path/to/file

# Copy command output to clipboard
some-command | pbcopy

# Read current clipboard contents
pbpaste
```

## Rules

1. **Use `echo -n`** (no trailing newline) for single-line values like URLs, tokens, and one-liners.
2. **Use heredoc with `'EOF'`** (single-quoted delimiter) for multi-line content to prevent shell expansion.
3. **Always confirm** what was copied by telling the user — e.g., "Copied the API key to your clipboard."
4. **Never silently copy** — always inform the user when you put something in their clipboard.
5. **Sensitive data** — warn the user if clipboard content contains secrets, as clipboard contents can be read by other apps.
6. **macOS only** — `pbcopy`/`pbpaste` are macOS commands. On Linux, suggest `xclip -selection clipboard` or `xsel --clipboard` as alternatives.
