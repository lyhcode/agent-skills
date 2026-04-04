# iTerm2 Python API Quick Reference

## Setup

Install the Python runtime: Scripts > Manage > Install Python Runtime

All scripts use this basic structure:
```python
import iterm2

async def main(connection):
    app = await iterm2.async_get_app(connection)
    # Your code here

# For one-time scripts:
iterm2.run_until_complete(main)

# For daemons:
iterm2.run_forever(main)
```

## App Navigation

```python
app = await iterm2.async_get_app(connection)

# Windows
windows = app.terminal_windows
window = app.current_terminal_window

# Tabs
tabs = window.tabs
tab = window.current_tab

# Sessions
session = tab.current_session
sessions = tab.sessions  # all sessions in tab (including splits)
```

## Creating Windows & Tabs

```python
# New window with default profile
window = await iterm2.Window.async_create(connection)

# New window with specific profile
window = await iterm2.Window.async_create(connection, profile="SSH")

# New tab in existing window
tab = await window.async_create_tab()

# Split pane
new_session = await session.async_split_pane(vertical=True)
```

## Sending Commands

```python
# Send text (as if typed)
await session.async_send_text("ls -la\n")

# Send text without newline
await session.async_send_text("cd ")
```

## Reading Session Content

```python
# Get screen contents
contents = await session.async_get_screen_contents()
for line_num in range(contents.number_of_lines):
    line = contents.line(line_num)
    print(line.string)
```

## Session Variables

```python
# Get a variable
hostname = await session.async_get_variable("hostname")
path = await session.async_get_variable("path")

# Set a user variable
await session.async_set_variable("user.myVar", "myValue")
```

## Profile Management

```python
# Get all profiles
profiles = await iterm2.PartialProfile.async_query(connection)

# Get current session's profile
profile = await session.async_get_profile()

# Modify profile (changes are live)
await profile.async_set_background_color(
    iterm2.Color(red=30, green=30, blue=40)
)
await profile.async_set_normal_font("MesloLGS-NF-Regular 14")
```

## Monitoring Events

```python
# Monitor for new sessions
async with iterm2.NewSessionMonitor(connection) as mon:
    while True:
        session_id = await mon.async_get()
        session = app.get_session_by_id(session_id)
        # React to new session

# Monitor for session termination
async with iterm2.SessionTerminationMonitor(connection) as mon:
    while True:
        session_id = await mon.async_get()
        # React to session ending

# Monitor for layout changes
async with iterm2.LayoutChangeMonitor(connection) as mon:
    while True:
        await mon.async_get()
        # React to layout change

# Monitor for focus changes
async with iterm2.FocusMonitor(connection) as mon:
    while True:
        update = await mon.async_get_next_update()
        # React to focus change
```

## Custom Status Bar Component

```python
component = iterm2.StatusBarComponent(
    short_description="Short Name",
    detailed_description="What this shows",
    knobs=[
        iterm2.StringKnob("Label", "default", "key"),
        iterm2.CheckboxKnob("Enable X", True, "enable_x"),
        iterm2.PositiveFloatingPointKnob("Interval", 5.0, "interval"),
        iterm2.ColorKnob("Text Color", iterm2.Color(), "color"),
    ],
    exemplar="Example Output",
    update_cadence=10,
    identifier="com.yourname.component-id"
)

@iterm2.StatusBarRPC
async def coro(knobs):
    label = knobs["key"]
    return f"{label}: some value"

await component.async_register(connection, coro)
```

## Registering Functions (RPCs)

Functions that can be called from key bindings, triggers, or interpolated strings:

```python
@iterm2.RPC
async def my_function(session_id):
    session = app.get_session_by_id(session_id)
    await session.async_send_text("hello\n")

await my_function.async_register(connection)
```

Call from key binding: set action to "Invoke Script Function" with `my_function(session_id: id)`

## Theme Detection

```python
theme = await app.async_get_theme()
# Returns list like ["dark"] or ["light"]

async with iterm2.VariableMonitor(
    connection, iterm2.VariableScopes.APP, "effectiveTheme", None
) as mon:
    while True:
        theme = await mon.async_get()
        # React to theme change
```

## Color Objects

```python
# Create color (0-255 range)
color = iterm2.Color(red=40, green=44, blue=52, alpha=255)

# From hex
r, g, b = int("28", 16), int("2c", 16), int("34", 16)
color = iterm2.Color(red=r, green=g, blue=b)
```

## File Locations

| Type | Path |
|------|------|
| Scripts | `~/Library/Application Support/iTerm2/Scripts/` |
| AutoLaunch | `~/Library/Application Support/iTerm2/Scripts/AutoLaunch/` |
| Python runtime | Managed by iTerm2 |
