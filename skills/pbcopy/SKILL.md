---
name: pbcopy
description: >
  Copy text, HTML, or images to the macOS clipboard using pbcopy and osascript.
  Use when the user asks to copy, share, or transfer content to the clipboard,
  or when generating output that the user will need to paste elsewhere
  (e.g., commands, code snippets, config blocks, URLs, tokens, images, rich text).
  Triggers on mentions of copy, clipboard, paste, pbcopy, or "copy to clipboard".
---

# Clipboard (pbcopy/pbpaste/osascript)

Automatically copy text, HTML, or images to the macOS clipboard so the user can paste it directly.

## When to Use

- The user explicitly asks to copy something to the clipboard
- You generate a snippet (command, code, config, URL, token) that the user said they want to paste elsewhere
- The user asks to copy an image or rich text/HTML to the clipboard
- The user asks you to "put it in my clipboard" or "copy that for me"

## Plain Text

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

## HTML / Rich Text

`pbcopy` automatically detects RTF headers, so convert HTML to RTF with `textutil` then pipe to `pbcopy`. The clipboard will contain rich text pasteable into Mail, Notion, Google Docs, etc.

```bash
# Convert HTML to RTF and copy to clipboard (preferred)
echo '<h1>Title</h1><p>This is <b>bold</b></p>' | textutil -stdin -format html -convert rtf -stdout | pbcopy

# From an HTML file
textutil -convert rtf /path/to/page.html -stdout | pbcopy

# Copy raw RTF directly (pbcopy auto-detects RTF header)
cat <<'EOF' | pbcopy
{\rtf1\ansi\deff0{\fonttbl{\f0 Helvetica;}}\f0\fs24 Hello \b Bold\b0  World}
EOF

# Paste as RTF
pbpaste -Prefer rtf
```

If you need the clipboard to contain native HTML type (not RTF), use `osascript`:

```bash
cat <<'EOF' > /tmp/clip.html
<h1>Title</h1>
<ul><li>Item 1</li><li>Item 2</li></ul>
EOF
osascript <<'APPLESCRIPT'
use framework "AppKit"
set htmlContent to do shell script "cat /tmp/clip.html"
set pb to current application's NSPasteboard's generalPasteboard()
pb's clearContents()
pb's setString:htmlContent forType:(current application's NSPasteboardTypeHTML)
APPLESCRIPT
rm /tmp/clip.html
```

## Images

`pbcopy` only handles text/RTF/EPS — images require `osascript` with NSPasteboard.

```bash
# Copy image to clipboard (PNG, JPG, GIF, TIFF, BMP, WebP)
osascript <<APPLESCRIPT
use framework "AppKit"
set img to (current application's NSImage's alloc()'s initWithContentsOfFile:"/path/to/image.png")
set pb to current application's NSPasteboard's generalPasteboard()
pb's clearContents()
pb's writeObjects:{img}
APPLESCRIPT
```

## Reading Clipboard

```bash
# Read plain text
pbpaste

# Read as RTF
pbpaste -Prefer rtf

# Check clipboard content types
osascript -e 'the clipboard as record'

# Save clipboard image to file
osascript <<'APPLESCRIPT'
use framework "AppKit"
set pb to current application's NSPasteboard's generalPasteboard()
set imgData to pb's dataForType:(current application's NSPasteboardTypePNG)
if imgData is not missing value then
  imgData's writeToFile:"/tmp/clipboard.png" atomically:true
  return "Saved to /tmp/clipboard.png"
else
  return "No image on clipboard"
end if
APPLESCRIPT
```

## Rules

1. **Use `echo -n`** (no trailing newline) for single-line values like URLs, tokens, and one-liners.
2. **Use heredoc with `'EOF'`** (single-quoted delimiter) for multi-line content to prevent shell expansion.
3. **Always confirm** what was copied by telling the user — e.g., "Copied the API key to your clipboard."
4. **Never silently copy** — always inform the user when you put something in their clipboard.
5. **Sensitive data** — warn the user if clipboard content contains secrets, as clipboard contents can be read by other apps.
6. **Temp files** — delete any temp files (`/tmp/clip.html`, etc.) after copying to clipboard.
7. **Choose the right method:**
   - Plain text → `echo | pbcopy`
   - HTML/rich text → `textutil -convert rtf -stdout | pbcopy` (preferred) or `osascript` for native HTML type
   - Images → `osascript` with `NSImage` + `NSPasteboard` (pbcopy cannot handle images)
8. **macOS only** — these commands use macOS APIs. On Linux, suggest `xclip -selection clipboard` or `xsel --clipboard` for text (rich content requires `xclip -t text/html` or similar).
