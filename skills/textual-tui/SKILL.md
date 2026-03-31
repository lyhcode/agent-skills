---
name: textual-tui
description: >
  Build terminal user interfaces (TUI) with the Python Textual framework.
  Use when the user wants to create a terminal app, TUI, CLI dashboard, interactive
  terminal tool, or any rich terminal interface using Python. Triggers on mentions of
  Textual, TUI, terminal UI, terminal app, terminal dashboard, ncurses alternative,
  Rich TUI, or building interactive CLI tools with Python. Even if the user just says
  "build me a terminal app" or "I need a dashboard in the terminal" without mentioning
  Textual specifically, this skill should be used because Textual is the best modern
  Python framework for this purpose.
version_target: "Textual 8.x"
references:
  - https://textual.textualize.io/
  - https://pypi.org/project/textual/
---

# Textual TUI Development

Build rich terminal user interfaces in Python using the Textual framework (v8.x).

## Quick Start

```bash
pip install textual
```

Minimal app:

```python
from textual.app import App, ComposeResult
from textual.widgets import Header, Footer, Static

class MyApp(App):
    TITLE = "My App"
    CSS_PATH = "app.tcss"  # or use CSS = "..." inline

    BINDINGS = [
        ("q", "quit", "Quit"),
    ]

    def compose(self) -> ComposeResult:
        yield Header()
        yield Static("Hello, World!")
        yield Footer()

if __name__ == "__main__":
    MyApp().run()
```

Run in dev mode for live CSS editing:
```bash
textual run --dev app.py
```

## Core Architecture

### App Lifecycle

1. **`compose()`** — Yield widgets to build the initial UI (called once per screen)
2. **`on_mount()`** — Post-composition setup (fetch data, configure state)
3. **`run()`** — Enters terminal mode, starts event loop
4. **`exit(return_value)`** — Leaves terminal mode

### Widget Composition

Widgets are composed declaratively by yielding them from `compose()`. Use containers for layout:

```python
from textual.containers import Horizontal, Vertical, Grid
from textual.widgets import Button, Input, Static

def compose(self) -> ComposeResult:
    yield Header()
    with Horizontal():
        with Vertical(id="sidebar"):
            yield Button("New", id="new")
            yield Button("Open", id="open")
        with Vertical(id="main"):
            yield Input(placeholder="Search...")
            yield Static("Content area")
    yield Footer()
```

The `with` context manager syntax nests widgets inside containers.

### Dynamic Widget Mounting

Add or remove widgets at runtime:

```python
async def on_button_pressed(self, event: Button.Pressed) -> None:
    await self.mount(Static("New widget"), after=self.query_one("#main"))
    self.query_one("#old").remove()
```

## Layout

Three layout modes, set on the parent container:

### Vertical (Default)

Children stack top-to-bottom. They expand to fill width automatically.

```css
#container { layout: vertical; }
```

### Horizontal

Children flow left-to-right. Set `height: 100%` on children or they collapse.

```css
#container { layout: horizontal; }
```

### Grid

Matrix layout with explicit columns/rows:

```css
#container {
    layout: grid;
    grid-size: 3 2;          /* 3 columns, 2 rows */
    grid-columns: 1fr 2fr 1fr;
    grid-rows: auto 1fr;
    grid-gutter: 1;
}
```

Span cells: `column-span: 2;` or `row-span: 2;`

### Container Widgets

| Import | Description |
|--------|-------------|
| `Container` | Vertical layout (basic) |
| `Vertical` | Expanding vertical, no scrollbars |
| `VerticalGroup` | Non-expanding vertical |
| `Horizontal` | Expanding horizontal |
| `HorizontalGroup` | Non-expanding horizontal |
| `Grid` | Grid layout |
| `ScrollableContainer` | Vertical with scrollbars + keyboard nav |
| `VerticalScroll` | Y-axis scrolling |
| `HorizontalScroll` | X-axis scrolling |
| `Center` | Center children horizontally |
| `Middle` | Center children vertically |

All from `textual.containers`.

### Docking

Fix widgets to edges (stays visible during scroll):

```css
#sidebar { dock: left; width: 20; }
#toolbar { dock: top; height: 3; }
```

### Fractional Units

Distribute remaining space proportionally:

```css
#sidebar { width: 1fr; }   /* 1/4 of space */
#main { width: 3fr; }      /* 3/4 of space */
```

## Event Handling

### Handler Naming Convention

Message class name converts to snake_case with `on_` prefix:

```python
# Button.Pressed → on_button_pressed
def on_button_pressed(self, event: Button.Pressed) -> None:
    self.notify(f"Clicked: {event.button.id}")

# Input.Changed → on_input_changed
def on_input_changed(self, event: Input.Changed) -> None:
    self.query_one("#output", Static).update(event.value)
```

Handlers can be `async`:

```python
async def on_mount(self) -> None:
    data = await self.fetch_data()
```

### Event Flow

Events bubble up from the widget that posted them through the DOM to the App. Call `event.stop()` to prevent further propagation.

### Decorator-Based Handlers

Filter events by CSS selector:

```python
from textual.on import on

@on(Button.Pressed, "#save")
def handle_save(self, event: Button.Pressed) -> None:
    self.save()

@on(Button.Pressed, "#cancel")
def handle_cancel(self, event: Button.Pressed) -> None:
    self.app.pop_screen()
```

### Custom Messages

```python
from textual.message import Message

class SearchBox(Static):
    class Submitted(Message):
        def __init__(self, query: str) -> None:
            self.query = query
            super().__init__()

    def submit(self) -> None:
        self.post_message(self.Submitted(self.value))
```

Parent widgets handle with `on_search_box_submitted`.

## Key Bindings

```python
from textual.binding import Binding

class MyApp(App):
    BINDINGS = [
        ("q", "quit", "Quit"),                              # Simple tuple
        ("d", "toggle_dark", "Toggle Dark"),
        Binding("ctrl+s", "save", "Save", show=False),      # Hidden from footer
        Binding("ctrl+q", "quit", "Quit", priority=True),   # Always active
    ]

    def action_save(self) -> None:
        """Action methods are prefixed with action_."""
        self.notify("Saved!")
```

Bindings search from focused widget upward — widget-level bindings override app-level.

## Reactive Attributes

Reactive attributes auto-trigger widget refresh on change:

```python
from textual.reactive import reactive

class Counter(Static):
    count = reactive(0)

    def render(self) -> str:
        return f"Count: {self.count}"

    def validate_count(self, value: int) -> int:
        """Clamp value to valid range."""
        return max(0, min(value, 100))

    def watch_count(self, old: int, new: int) -> None:
        """React to changes (old value is optional param)."""
        self.log(f"Count: {old} → {new}")
```

Options:
- `reactive(default, layout=True)` — Triggers layout recalculation
- `reactive(default, recompose=True)` — Rebuilds child widgets
- `reactive(default, init=False)` — Don't call watcher on init

For mutable types (lists, dicts), call `self.mutate_reactive("attr_name")` after in-place mutation.

## CSS Styling (TCSS)

Textual uses a CSS-like language (`.tcss` files) with terminal-specific adaptations.

### Applying Styles

```python
class MyApp(App):
    CSS_PATH = "app.tcss"       # External file
    # or
    CSS = """
    #sidebar { width: 25; background: $surface; }
    """
```

Widgets can define default CSS:

```python
class MyWidget(Static):
    DEFAULT_CSS = """
    MyWidget { height: auto; padding: 1 2; }
    """
```

### Selectors

```css
Button { }                  /* Type selector */
#save { }                   /* ID selector */
.warning { }                /* Class selector */
Button.primary { }          /* Type + class */
#dialog Button { }          /* Descendant */
#dialog > Button { }        /* Direct child */
Button:hover { }            /* Pseudo-class */
Button:focus { }
*:disabled { opacity: 0.5; }
```

### CSS Variables

```css
$accent: #1e90ff;
$sidebar-width: 25;

#sidebar {
    width: $sidebar-width;
    background: $accent;
}
```

### Theme Colors

Use semantic theme variables for consistent light/dark theming:

```css
Button {
    background: $primary;
    color: $text;
}
.error { color: $error; }
.success { color: $success; }
.muted { color: $text-muted; }
```

Available: `$primary`, `$secondary`, `$accent`, `$warning`, `$error`, `$success`, plus `-lighten-*`, `-darken-*`, `-muted` variants.

### Common Properties

```css
Widget {
    width: 50%;              /* or: auto, 1fr, 10, 50vw */
    height: 100%;            /* or: auto, 1fr, 10, 50vh */
    min-width: 20;
    max-width: 80;
    padding: 1 2;            /* vertical horizontal */
    margin: 1;
    background: $surface;
    color: $text;
    border: heavy $primary;  /* types: ascii, dashed, double, heavy, round, solid, thick */
    border-title-align: center;
    display: block;          /* or: none */
    overflow-y: auto;        /* auto, hidden, scroll */
    opacity: 1.0;
    text-align: center;
    text-style: bold;
    content-align: center middle;
    align: center middle;
}
```

### Sizing Units

| Unit | Example | Meaning |
|------|---------|---------|
| cells | `width: 20;` | Fixed terminal columns |
| `%` | `width: 50%;` | Percentage of parent |
| `fr` | `width: 1fr;` | Fraction of remaining space |
| `vw`/`vh` | `width: 50vw;` | Viewport percentage |
| `w`/`h` | `width: 50w;` | Container percentage |
| `auto` | `width: auto;` | Content-driven |

## Common Recipes

### Dashboard Layout

```python
def compose(self) -> ComposeResult:
    yield Header()
    with Horizontal():
        with Vertical(id="sidebar"):
            yield Static("Navigation", classes="title")
            yield ListView(ListItem(Label("Item 1")), ListItem(Label("Item 2")))
        with Vertical(id="content"):
            yield Static("Dashboard", classes="title")
            with Grid(id="metrics"):
                yield Static("Metric 1", classes="card")
                yield Static("Metric 2", classes="card")
                yield Static("Metric 3", classes="card")
    yield Footer()
```

### Form with Validation

```python
def compose(self) -> ComposeResult:
    yield Input(placeholder="Name", id="name")
    yield Input(placeholder="Email", id="email", validators=[Function(lambda v: "@" in v, "Invalid email")])
    yield Button("Submit", variant="primary", id="submit")

def on_button_pressed(self, event: Button.Pressed) -> None:
    if event.button.id == "submit":
        name = self.query_one("#name", Input).value
        email = self.query_one("#email", Input).value
        self.notify(f"Submitted: {name} ({email})")
```

### DataTable

```python
from textual.widgets import DataTable

def compose(self) -> ComposeResult:
    yield DataTable()

def on_mount(self) -> None:
    table = self.query_one(DataTable)
    table.add_columns("Name", "Age", "City")
    table.add_rows([
        ("Alice", 30, "NYC"),
        ("Bob", 25, "London"),
    ])
```

### Async Data Loading with Workers

```python
from textual.worker import work

class MyApp(App):
    @work(exclusive=True)
    async def fetch_data(self, url: str) -> None:
        self.query_one("#status").update("Loading...")
        response = await asyncio.to_thread(requests.get, url)
        data = response.json()
        self.query_one("#status").update(f"Loaded {len(data)} items")

    def on_button_pressed(self, event: Button.Pressed) -> None:
        self.fetch_data("https://api.example.com/data")
```

## Dev Tools

```bash
textual run --dev app.py     # Live CSS editing, debug console
textual serve app.py         # Serve in browser
textual colors               # Preview terminal colors
textual easing               # Preview animation easing functions
```

## Reference Documentation

For detailed widget, CSS, and advanced pattern references, read these files:

- **`reference/widgets.md`** — All built-in widgets: imports, constructors, events, examples
- **`reference/css.md`** — Complete CSS properties, selectors, layout, colors, borders
- **`reference/patterns.md`** — Screens, workers, testing, command palette, custom widgets
