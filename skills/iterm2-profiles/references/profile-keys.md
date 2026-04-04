# iTerm2 Profile Attribute Keys Reference

Complete reference for keys usable in Dynamic Profiles JSON files.

## How to Discover Keys

The definitive way to find the exact key names and value formats is to export a profile from iTerm2:
1. Settings > Profiles > select profile > Other Actions > Save Profile as JSON
2. Inspect the exported JSON for exact key names and formats

## Color Keys

All color keys use this format:
```json
{
  "Red Component": 0.0,
  "Green Component": 0.0,
  "Blue Component": 0.0,
  "Alpha Component": 1.0,
  "Color Space": "sRGB"
}
```

| Key | Description |
|-----|-------------|
| `Foreground Color` | Default text color |
| `Background Color` | Terminal background |
| `Bold Color` | Bold text color |
| `Cursor Color` | Cursor block/bar fill |
| `Cursor Text Color` | Character under cursor |
| `Cursor Guide Color` | Horizontal highlight at cursor row |
| `Selection Color` | Selection highlight background |
| `Selected Text Color` | Text within selection |
| `Badge Color` | Badge overlay color |
| `Link Color` | Hyperlink color |
| `Ansi 0 Color` | Black |
| `Ansi 1 Color` | Red |
| `Ansi 2 Color` | Green |
| `Ansi 3 Color` | Yellow |
| `Ansi 4 Color` | Blue |
| `Ansi 5 Color` | Magenta |
| `Ansi 6 Color` | Cyan |
| `Ansi 7 Color` | White |
| `Ansi 8 Color` | Bright Black |
| `Ansi 9 Color` | Bright Red |
| `Ansi 10 Color` | Bright Green |
| `Ansi 11 Color` | Bright Yellow |
| `Ansi 12 Color` | Bright Blue |
| `Ansi 13 Color` | Bright Magenta |
| `Ansi 14 Color` | Bright Cyan |
| `Ansi 15 Color` | Bright White |

For light/dark mode variants, append ` (Dark)` or ` (Light)` to the key name.

## Font Keys

| Key | Format | Example |
|-----|--------|---------|
| `Normal Font` | "FontName Size" | `"MesloLGS-NF-Regular 13"` |
| `Non Ascii Font` | "FontName Size" | `"Monaco 12"` |
| `ASCII Anti Aliased` | Boolean (1/0) | `1` |
| `Use Bold Font` | Boolean | `1` |
| `Use Bright Bold` | Boolean | `1` |
| `Use Italic Font` | Boolean | `1` |
| `Horizontal Spacing` | Float | `1.0` |
| `Vertical Spacing` | Float | `1.0` |

## Terminal Behavior Keys

| Key | Format | Description |
|-----|--------|-------------|
| `Scrollback Lines` | Integer | Lines of scrollback (e.g., 1000) |
| `Unlimited Scrollback` | Boolean | Ignore Scrollback Lines limit |
| `Character Encoding` | Integer | 4 = UTF-8 |
| `Terminal Type` | String | e.g., `"xterm-256color"` |
| `Mouse Reporting` | Boolean | Enable mouse events |
| `Blinking Cursor` | Boolean | Cursor blinks |
| `Silence Bell` | Boolean | Mute bell |
| `Visual Bell` | Boolean | Flash screen on bell |
| `Flashing Bell` | Boolean | Flash bell icon |
| `BM Growl` | Boolean | Notification on bell |
| `Close Sessions On End` | Boolean | Close on exit |
| `Prompt Before Closing 2` | Integer | 0=never, 1=if jobs, 2=always |

## Session Keys

| Key | Format | Description |
|-----|--------|-------------|
| `Custom Command` | String | `"Yes"` to use Command field |
| `Command` | String | Command to run (e.g., `"/usr/bin/ssh foo.com"`) |
| `Custom Directory` | String | `"Yes"`, `"Recycle"`, `"No"` |
| `Working Directory` | String | Initial directory path |
| `Send Code When Idle` | Boolean | Send keepalive |
| `Idle Code` | Integer | ASCII code to send (0 = NUL) |

## Window Keys

| Key | Format | Description |
|-----|--------|-------------|
| `Columns` | Integer | Terminal width |
| `Rows` | Integer | Terminal height |
| `Window Type` | Integer | Window style (0=normal) |
| `Disable Window Resizing` | Boolean | Lock size |
| `Transparency` | Float (0-1) | Window transparency |
| `Blur` | Boolean | Blur background |
| `Blur Radius` | Float | Blur amount |
| `Background Image Location` | String | Path to image |

## Input Keys

| Key | Format | Description |
|-----|--------|-------------|
| `Option Key Sends` | Integer | 0=Normal, 1=Meta, 2=Esc+ |
| `Right Option Key Sends` | Integer | Same as above |
| `Keyboard Map` | Dictionary | Custom key bindings |
| `Ambiguous Double Width` | Boolean | Treat ambiguous chars as double-width |

## Profile Identity Keys

| Key | Format | Description |
|-----|--------|-------------|
| `Name` | String | Profile display name |
| `Guid` | String | Unique identifier |
| `Default Bookmark` | String | `"Yes"` if this is the default profile |
| `Dynamic Profile Parent Name` | String | Parent profile to inherit from |
| `Dynamic Profile Parent GUID` | String | Parent by GUID (v3.4.9+) |
| `Tags` | Array of String | Searchable tags |

## Light/Dark Mode

Set `"Use Separate Colors for Light and Dark Mode": true` and then use:
- `"Background Color (Dark)"` / `"Background Color (Light)"`
- Same pattern for all color keys
