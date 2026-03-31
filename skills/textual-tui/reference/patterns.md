# Textual Advanced Patterns

## Table of Contents

- [Screens & Navigation](#screens--navigation)
- [Workers (Async Operations)](#workers-async-operations)
- [Custom Widgets](#custom-widgets)
- [Command Palette](#command-palette)
- [DOM Queries](#dom-queries)
- [Testing](#testing)
- [Project Structure](#project-structure)

---

## Screens & Navigation

### Creating Screens

```python
from textual.screen import Screen, ModalScreen
from textual.app import ComposeResult

class MainScreen(Screen):
    TITLE = "Main"

    def compose(self) -> ComposeResult:
        yield Header()
        yield Static("Main content")
        yield Footer()
```

### Screen Registration

```python
class MyApp(App):
    SCREENS = {
        "main": MainScreen,
        "settings": SettingsScreen,
    }
```

### Screen Stack Operations

```python
# Push — adds on top, previous screen stays in stack
self.app.push_screen("settings")

# Pop — removes current screen, returns to previous
self.app.pop_screen()

# Switch — replaces current screen entirely
self.app.switch_screen("main")

# Install/uninstall dynamic screens
self.app.install_screen(TempScreen(), name="temp")
self.app.uninstall_screen("temp")
```

### Modal Screens

Modal screens overlay the current screen and block interaction with it:

```python
from textual.screen import ModalScreen

class ConfirmDialog(ModalScreen[bool]):
    """Modal that returns True/False."""

    DEFAULT_CSS = """
    ConfirmDialog {
        align: center middle;
    }
    #dialog {
        width: 60;
        height: auto;
        border: thick $primary;
        padding: 1 2;
    }
    """

    def compose(self) -> ComposeResult:
        with Vertical(id="dialog"):
            yield Static("Are you sure?")
            with Horizontal():
                yield Button("Yes", variant="primary", id="yes")
                yield Button("No", id="no")

    def on_button_pressed(self, event: Button.Pressed) -> None:
        self.dismiss(event.button.id == "yes")
```

### Getting Results from Screens

```python
# Callback style
def handle_result(confirmed: bool) -> None:
    if confirmed:
        self.delete_item()

self.app.push_screen(ConfirmDialog(), handle_result)

# Await style (in a worker)
confirmed = await self.app.push_screen_wait(ConfirmDialog())
if confirmed:
    self.delete_item()
```

### Modes

Independent screen stacks for different app states:

```python
class MyApp(App):
    MODES = {
        "browse": BrowseScreen,
        "edit": EditScreen,
    }

    BINDINGS = [
        ("b", "switch_mode('browse')", "Browse"),
        ("e", "switch_mode('edit')", "Edit"),
    ]
```

---

## Workers (Async Operations)

Workers run long operations without blocking the UI.

### @work Decorator (Async)

```python
from textual.worker import work

class MyApp(App):
    @work(exclusive=True)
    async def fetch_data(self, url: str) -> None:
        """Exclusive: cancels previous worker if called again."""
        import httpx
        async with httpx.AsyncClient() as client:
            response = await client.get(url)
            data = response.json()
        # Safe to update UI directly in async workers
        self.query_one("#result", Static).update(str(data))
```

### Thread Workers (for sync/blocking libraries)

```python
from textual.worker import work, get_current_worker

class MyApp(App):
    @work(exclusive=True, thread=True)
    def fetch_data(self, url: str) -> None:
        """Runs in a thread — must use call_from_thread for UI updates."""
        import requests
        worker = get_current_worker()
        response = requests.get(url)
        if not worker.is_cancelled:
            self.call_from_thread(
                self.query_one("#result", Static).update,
                response.text
            )
```

### Manual Workers

```python
self.run_worker(self.some_coroutine(), exclusive=True, name="my-task")
```

### Worker States

`PENDING` → `RUNNING` → `SUCCESS` | `CANCELLED` | `ERROR`

```python
def on_worker_state_changed(self, event: Worker.StateChanged) -> None:
    if event.state == WorkerState.SUCCESS:
        self.notify("Done!")
    elif event.state == WorkerState.ERROR:
        self.notify("Failed!", severity="error")
```

### Key Options

| Option | Effect |
|--------|--------|
| `exclusive=True` | Cancels previous worker with same method |
| `thread=True` | Runs in thread instead of async |
| `group="name"` | Group workers for collective cancellation |
| `exit_on_error=True` | Exit app on unhandled exception |

---

## Custom Widgets

### Basic Widget with Render

```python
from textual.widget import Widget

class Clock(Widget):
    time = reactive("00:00:00")

    def render(self) -> str:
        return self.time
```

### Composite Widget

```python
from textual.containers import Horizontal
from textual.widgets import Input, Button, Static
from textual.message import Message

class SearchBar(Static):
    """Reusable search component."""

    DEFAULT_CSS = """
    SearchBar {
        layout: horizontal;
        height: auto;
    }
    SearchBar Input { width: 1fr; }
    SearchBar Button { width: auto; }
    """

    class Searched(Message):
        """Posted when user searches."""
        def __init__(self, query: str) -> None:
            self.query = query
            super().__init__()

    def compose(self) -> ComposeResult:
        yield Input(placeholder="Search...", id="search-input")
        yield Button("Go", variant="primary", id="search-btn")

    def on_button_pressed(self, event: Button.Pressed) -> None:
        query = self.query_one("#search-input", Input).value
        self.post_message(self.Searched(query))

    def on_input_submitted(self, event: Input.Submitted) -> None:
        self.post_message(self.Searched(event.value))
```

Parent handles: `on_search_bar_searched(self, event: SearchBar.Searched)`

### Widget with DEFAULT_CSS

Provide default styles that users can override:

```python
class Card(Static):
    DEFAULT_CSS = """
    Card {
        height: auto;
        padding: 1 2;
        margin: 1;
        border: round $primary;
        background: $surface;
    }
    """
```

### Widget with Bindings

```python
class Editor(Widget):
    BINDINGS = [
        ("ctrl+s", "save", "Save"),
        ("ctrl+z", "undo", "Undo"),
    ]

    def action_save(self) -> None:
        self.post_message(self.Saved())

    def action_undo(self) -> None:
        ...
```

### CSS Classes on Widgets

Toggle visual states:

```python
class MyWidget(Widget):
    def toggle_active(self) -> None:
        self.toggle_class("active")

    def set_error(self) -> None:
        self.add_class("error")
        self.remove_class("success")
```

```css
MyWidget.active { background: $accent; }
MyWidget.error { border: heavy $error; }
MyWidget.success { border: heavy $success; }
```

---

## Command Palette

Built-in fuzzy search (Ctrl+P by default).

### Configuration

```python
class MyApp(App):
    ENABLE_COMMAND_PALETTE = True  # Default
    COMMAND_PALETTE_BINDING = "ctrl+p"  # Default key
```

### Adding System Commands

```python
from textual.command import SystemCommand

class MyApp(App):
    def get_system_commands(self, screen) -> Iterable[SystemCommand]:
        yield from super().get_system_commands(screen)
        yield SystemCommand("Clear log", "Clear the application log", self.clear_log)
        yield SystemCommand("Export data", "Export to CSV", self.export_csv)
```

### Custom Command Provider

```python
from textual.command import Provider, Hit, DiscoveryHit

class FileProvider(Provider):
    async def startup(self) -> None:
        """Initialize resources (called once)."""
        self.files = await self.load_files()

    async def search(self, query: str) -> Hits:
        """Return matches for the query."""
        for f in self.files:
            if query.lower() in f.lower():
                yield Hit(
                    match_display=f,       # Shown in results
                    command=lambda: self.open_file(f),
                    help="Open this file",
                )

    async def discover(self) -> DiscoveryHits:
        """Shown when input is empty."""
        yield DiscoveryHit("Recent files", "Show recently opened")

class MyApp(App):
    COMMANDS = App.COMMANDS | {FileProvider}
```

---

## DOM Queries

### Single Widget

```python
# By ID (raises NoMatches if not found)
button = self.query_one("#save", Button)

# By type
header = self.query_one(Header)
```

### Multiple Widgets

```python
# All buttons
buttons = self.query("Button")

# By class
warnings = self.query(".warning")

# By descendant
dialog_buttons = self.query("#dialog Button")

# Filter and chain
active = self.query("Button").filter(".active").exclude(".hidden")
```

### Bulk Operations

```python
self.query("Button").add_class("highlighted")
self.query("Button").remove_class("disabled")
self.query(".old").remove()
self.query("Input").set(disabled=True)
```

### Navigation

```python
buttons = self.query("Button")
first = buttons.first()
last = buttons.last()
```

---

## Testing

### Setup

```bash
pip install pytest pytest-asyncio
# Optional for snapshot testing:
pip install pytest-textual-snapshot
```

### Basic Test

```python
import pytest
from textual.app import App, ComposeResult
from textual.widgets import Button, Static

class MyApp(App):
    def compose(self) -> ComposeResult:
        yield Button("Click", id="btn")
        yield Static("", id="output")

    def on_button_pressed(self) -> None:
        self.query_one("#output", Static).update("Clicked!")

@pytest.mark.asyncio
async def test_button_click():
    app = MyApp()
    async with app.run_test() as pilot:
        await pilot.click("#btn")
        await pilot.pause()  # Wait for message processing
        assert app.query_one("#output", Static).renderable == "Clicked!"
```

### Pilot Methods

```python
async with app.run_test(size=(80, 24)) as pilot:
    # Keyboard
    await pilot.press("tab")
    await pilot.press("enter")
    await pilot.press("ctrl+s")
    await pilot.press("h", "e", "l", "l", "o")  # Type sequence

    # Mouse
    await pilot.click("#widget")
    await pilot.click(Button)
    await pilot.click("#btn", offset=(5, 0))
    await pilot.click("#btn", times=2)        # Double-click

    # Wait for processing
    await pilot.pause()

    # Resize terminal
    await pilot.resize_terminal(100, 50)
```

### Snapshot Testing

Compare rendered output against saved snapshots:

```python
def test_app_snapshot(snap_compare):
    assert snap_compare("app.py")

def test_after_interaction(snap_compare):
    assert snap_compare(
        "app.py",
        press=["tab", "enter"],
        terminal_size=(80, 24),
    )
```

First run saves the snapshot. Subsequent runs compare. Update with `pytest --snapshot-update`.

---

## Project Structure

Recommended structure for a Textual app:

```
my-app/
├── app.py              # Main App class and entry point
├── app.tcss            # Global styles
├── screens/
│   ├── __init__.py
│   ├── main.py         # MainScreen
│   └── settings.py     # SettingsScreen
├── widgets/
│   ├── __init__.py
│   ├── search_bar.py   # Custom SearchBar widget
│   └── card.py         # Custom Card widget
├── models/
│   └── data.py         # Data models
├── pyproject.toml
└── README.md
```

### Entry Point Pattern

```python
# app.py
from textual.app import App, ComposeResult

class MyApp(App):
    CSS_PATH = "app.tcss"
    TITLE = "My App"
    ...

def main():
    app = MyApp()
    app.run()

if __name__ == "__main__":
    main()
```

### pyproject.toml

```toml
[project]
name = "my-app"
requires-python = ">=3.9"
dependencies = ["textual>=8.0"]

[project.scripts]
my-app = "app:main"
```

### Dev Workflow

```bash
# Run with live CSS reload
textual run --dev app.py

# Serve in browser (for sharing/demos)
textual serve app.py

# Preview colors
textual colors

# Preview easing functions
textual easing

# Run tests
pytest
pytest --snapshot-update  # Update snapshots
```
