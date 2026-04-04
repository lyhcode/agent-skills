# iTerm2 Proprietary Escape Codes Reference

All escape codes use the OSC (Operating System Command) format:
`ESC ] code ; params BEL` or `ESC ] code ; params ST`

In bash/zsh: `\e]` = ESC ], `\a` = BEL

## Profile & Session

```bash
# Change profile
printf "\e]1337;SetProfile=%s\a" "ProfileName"

# Set session title
printf "\e]0;%s\a" "My Title"

# Set tab title specifically
printf "\e]1;%s\a" "Tab Title"

# Set window title specifically
printf "\e]2;%s\a" "Window Title"

# Report current directory
printf "\e]1337;CurrentDir=%s\a" "$PWD"

# Report current host and user (for profile switching)
printf "\e]1337;RemoteHost=%s@%s\a" "$USER" "$HOSTNAME"
```

## Visual

```bash
# Set badge text (base64 encoded)
printf "\e]1337;SetBadgeFormat=%s\a" $(echo -n "badge text" | base64)

# Cursor shape: 0=block, 1=vertical bar, 2=underline
printf "\e]1337;CursorShape=%d\a" 1

# Cursor guide (horizontal highlight line at cursor)
printf "\e]1337;HighlightCursorLine=%s\a" "yes"  # or "no"

# Change tab color
printf "\e]6;1;bg;red;brightness;%d\a" 255
printf "\e]6;1;bg;green;brightness;%d\a" 100
printf "\e]6;1;bg;blue;brightness;%d\a" 100

# Reset tab color
printf "\e]6;1;bg;*;default\a"

# Set background color of session
printf "\e]1337;SetColors=bg=%s\a" "ff3030"  # hex RGB
# Also: fg, bold, link, selbg, selfg, curbg, curfg, underline
# And ANSI colors: black, red, green, yellow, blue, magenta, cyan, white
# Bright variants: br_black, br_red, etc.

# Reset a color to profile default
printf "\e]1337;SetColors=bg=default\a"
```

## Notifications & Feedback

```bash
# Post notification
printf "\e]9;%s\a" "Task complete!"

# Request attention (bounce Dock icon)
printf "\e]1337;RequestAttention=%s\a" "fireworks"  # or "yes" or "no"

# Progress bar (0-100, -1 to hide, -2 for indeterminate)
printf "\e]1337;Progress=%d\a" 75

# Progress with state: ok, error, warning, indeterminate
printf "\e]1337;Progress=%d;state=%s\a" 75 "ok"
```

## Clipboard

```bash
# Copy to clipboard (base64 encoded content)
printf "\e]1337;Copy=:%s\a" $(echo -n "text to copy" | base64)

# Copy to specific clipboard: unnamed, ruler, find, font
printf "\e]52;c;%s\a" $(echo -n "text" | base64)
```

## Images

```bash
# Display inline image (base64 encoded)
printf "\e]1337;File=name=%s;size=%d;inline=1:%s\a" \
  $(echo -n "image.png" | base64) \
  $(wc -c < image.png) \
  $(base64 < image.png)

# With size options
printf "\e]1337;File=inline=1;width=40;height=10;preserveAspectRatio=1:%s\a" \
  $(base64 < image.png)
```

## Marks & Annotations

```bash
# Set mark
printf "\e]1337;SetMark\a"

# Add annotation at cursor
printf "\e]1337;AddAnnotation=%s\a" "This is a note"

# Add annotation with length (covers N preceding characters)
printf "\e]1337;AddAnnotation=message=%s|length=%d\a" "note" 10
```

## User Variables

```bash
# Set user variable (value is base64 encoded)
printf "\e]1337;SetUserVar=%s=%s\a" "varName" $(echo -n "value" | base64)
```

## Shell Integration Marks

These are used by shell integration to track prompts and commands:

```bash
# Mark prompt start
printf "\e]133;A\a"
# Mark command start (after user presses enter)
printf "\e]133;C\a"
# Mark command end with exit status
printf "\e]133;D;%d\a" $?
```

## Hyperlinks (OSC 8)

```bash
# Create clickable hyperlink
printf "\e]8;;https://example.com\a"
printf "Click here"
printf "\e]8;;\a"
```

## Convenience Shell Functions

Add to `~/.zshrc` for easy access:

```zsh
# Post a notification
iterm2_notify() { printf "\e]9;%s\a" "$1"; }

# Set badge
iterm2_badge() { printf "\e]1337;SetBadgeFormat=%s\a" $(echo -n "$1" | base64); }

# Set tab color (R G B, each 0-255)
iterm2_tab_color() {
  printf "\e]6;1;bg;red;brightness;%d\a" "$1"
  printf "\e]6;1;bg;green;brightness;%d\a" "$2"
  printf "\e]6;1;bg;blue;brightness;%d\a" "$3"
}

# Reset tab color
iterm2_tab_color_reset() { printf "\e]6;1;bg;*;default\a"; }

# Show progress
iterm2_progress() { printf "\e]1337;Progress=%d\a" "$1"; }

# Change profile
iterm2_profile() { printf "\e]1337;SetProfile=%s\a" "$1"; }
```
