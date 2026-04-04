---
name: iterm2-profiles
description: "iTerm2 profile management and visual customization. Use this skill whenever the user wants to change iTerm2 appearance, create or modify profiles, adjust colors/themes/fonts, set up key mappings, configure automatic profile switching, or customize the status bar layout. Also trigger when the user mentions terminal color schemes, iTerm2 settings, dynamic profiles, or wants their terminal to look different based on context (SSH, root, project)."
---

# iTerm2 Profile Management & Visual Customization

Help the user create, manage, and customize iTerm2 profiles — including colors, fonts, key mappings, automatic profile switching, and status bar configuration.

## User's Environment

- iTerm2 3.6.8 on macOS, using zsh
- Preferences stored at: `defaults read com.googlecode.iterm2`
- Dynamic Profiles directory: `~/Library/Application Support/iTerm2/DynamicProfiles/`

## Core Concepts

### Dynamic Profiles

Dynamic Profiles are the preferred way to manage iTerm2 profiles programmatically. They are JSON (or plist) files placed in `~/Library/Application Support/iTerm2/DynamicProfiles/`. iTerm2 watches this directory and reloads automatically on changes.

Each file contains a `Profiles` array:

```json
{
  "Profiles": [
    {
      "Name": "My Profile",
      "Guid": "unique-uuid-here",
      "Dynamic Profile Parent Name": "Default",
      "Background Color": {
        "Red Component": 0.1,
        "Green Component": 0.1,
        "Blue Component": 0.15,
        "Alpha Component": 1,
        "Color Space": "sRGB"
      },
      "Normal Font": "MesloLGS-NF-Regular 13"
    }
  ]
}
```

Key rules:
- Every profile needs a unique `Guid` (use `uuidgen` to generate)
- `Name` is the display name in iTerm2's profile list
- Use `Dynamic Profile Parent Name` to inherit from another profile — only override what you want to change
- Colors use the `{"Red Component": float, "Green Component": float, "Blue Component": float, "Alpha Component": float, "Color Space": "sRGB"}` format, with values from 0.0 to 1.0

### Color Scheme Structure

iTerm2 uses 22 named colors in a profile:

| Key | Purpose |
|-----|---------|
| `Foreground Color` | Default text |
| `Background Color` | Terminal background |
| `Bold Color` | Bold text |
| `Cursor Color` | Cursor fill |
| `Cursor Text Color` | Text under cursor |
| `Cursor Guide Color` | Horizontal line at cursor (set alpha ~0.25) |
| `Selection Color` | Selection background |
| `Selected Text Color` | Selected text |
| `Badge Color` | Badge overlay text |
| `Link Color` | Clickable links |
| `Ansi 0 Color` – `Ansi 15 Color` | The 16 ANSI colors (0-7 normal, 8-15 bright) |

The standard ANSI color mapping:
- 0/8: Black/Bright Black
- 1/9: Red/Bright Red
- 2/10: Green/Bright Green
- 3/11: Yellow/Bright Yellow
- 4/12: Blue/Bright Blue
- 5/13: Magenta/Bright Magenta
- 6/14: Cyan/Bright Cyan
- 7/15: White/Bright White

iTerm2 supports separate light/dark mode colors. Append ` (Dark)` or ` (Light)` to the key name (e.g., `"Background Color (Dark)"`). Enable with `"Use Separate Colors for Light and Dark Mode": true`.

### Automatic Profile Switching

Requires Shell Integration to be installed. Rules are set in profile's Advanced settings.

Rule format: `[!][username@]hostname[:path][&job]`

Scoring: exact hostname = 16, wildcard hostname = 8, job = 4, username = 2, exact path = 1. Highest score wins.

Prefix `!` makes the rule "sticky" — it won't revert when conditions stop matching.

Examples:
- `root@` — switch when logged in as root (any host)
- `*@production-*` — switch for any user on hosts starting with "production-"
- `george@myserver.com:/var/log` — specific user, host, and path
- `&vim` — switch when vim is the foreground job

### Key Mappings

Key mappings live in the `Keyboard Map` profile attribute. The format is a dictionary keyed by hex key codes:

```json
"Keyboard Map": {
  "0x61-0x80000": {
    "Action": 10,
    "Text": "custom action text"
  }
}
```

For complex key mapping changes, it's often easier to configure them in the iTerm2 GUI (Settings > Profiles > Keys) and then export the profile.

### Status Bar

Enable in Settings > Profiles > Session > "Status bar enabled".

Built-in components include: battery, CPU, memory, network throughput, current directory, hostname, username, git state, clock, composer, search, and more.

Configuration is done through the GUI drag-and-drop interface. The status bar can be placed at top or bottom of the session.

For custom status bar components via Python API, see the `iterm2-automation` skill.

## Workflow

When the user asks to customize their iTerm2 profile:

1. **Read current state** — Check existing profiles in `~/Library/Application Support/iTerm2/DynamicProfiles/` and current defaults via `defaults read com.googlecode.iterm2 "New Bookmarks"`
2. **Create/modify dynamic profile** — Write JSON to the DynamicProfiles directory. Always inherit from "Default" unless the user specifies otherwise.
3. **Verify** — Read the file back and confirm the changes are syntactically valid JSON.

When the user wants a specific theme or color scheme:
1. Check if it's a well-known theme (Dracula, Solarized, Nord, Catppuccin, Gruvbox, One Dark, Tokyo Night, etc.) and generate the correct color values
2. Create a dynamic profile with those colors
3. Tell the user to select the profile in iTerm2 (Settings > Profiles > select it > "Set as Default" or just use it for specific sessions)

When the user wants automatic profile switching:
1. Confirm shell integration is installed (check for `iterm2_shell_integration` in `~/.zshrc`). If not, suggest installing it first (see `iterm2-shell-setup` skill).
2. Create the target profiles as dynamic profiles
3. Guide the user to add switching rules in Settings > Profiles > Advanced for each profile

## Reference

For the complete list of profile attribute keys, read `references/profile-keys.md`.
