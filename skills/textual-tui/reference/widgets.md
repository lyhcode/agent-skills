# Textual Widget Reference

All widgets are imported from `textual.widgets` unless noted.

## Table of Contents

- [Input Controls](#input-controls)
- [Display Widgets](#display-widgets)
- [Text & Content](#text--content)
- [Layout & Navigation](#layout--navigation)
- [Data & Lists](#data--lists)
- [Progress & Status](#progress--status)

---

## Input Controls

### Button

```python
from textual.widgets import Button
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `label` | str | Button text |
| `variant` | str | `"default"`, `"primary"`, `"success"`, `"warning"`, `"error"` |
| `disabled` | bool | Disable interaction |
| `compact` | bool | Borderless compact style |

**Events:** `Button.Pressed` — properties: `button`, `control`

```python
yield Button("Save", variant="primary", id="save")
yield Button.success("OK")
yield Button.error("Delete")

def on_button_pressed(self, event: Button.Pressed) -> None:
    if event.button.id == "save":
        self.save()
```

### Input

```python
from textual.widgets import Input
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `value` | str | Initial value |
| `placeholder` | str | Hint text |
| `password` | bool | Mask content |
| `type` | str | `"text"`, `"integer"`, `"number"` |
| `max_length` | int | Character limit (0 = unlimited) |
| `restrict` | str | Regex to filter characters |
| `validators` | list | Validation rules |
| `validate_on` | list | When to validate: `"blur"`, `"changed"`, `"submitted"` |
| `select_on_focus` | bool | Auto-select on focus |

**Events:**
- `Input.Changed` — properties: `value`, `validation_result`, `input`
- `Input.Submitted` — properties: `value`, `input` (fires on Enter)

```python
yield Input(placeholder="Enter name", id="name")
yield Input(type="integer", placeholder="Age")
yield Input(password=True, placeholder="Password")
```

### TextArea

```python
from textual.widgets import TextArea
```

Multi-line text editor with optional syntax highlighting (tree-sitter).

| Parameter | Type | Description |
|-----------|------|-------------|
| `text` | str | Initial content |
| `language` | str | Syntax highlighting language |
| `show_line_numbers` | bool | Show line numbers |
| `read_only` | bool | Disable editing |
| `tab_behavior` | str | `"indent"` or `"focus"` |

**Events:** `TextArea.Changed`, `TextArea.SelectionChanged`

Supported languages: `python`, `markdown`, `json`, `toml`, `yaml`, `html`, `css`, `javascript`, `rust`, `go`, `sql`, `java`, `bash`, `xml`

```python
yield TextArea("print('hello')", language="python", show_line_numbers=True)
```

### Select

```python
from textual.widgets import Select
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `options` | iterable | `(label, value)` tuples |
| `prompt` | str | Text when empty (default: `"Select"`) |
| `allow_blank` | bool | Allow deselection |
| `value` | any | Initial value |
| `type_to_search` | bool | Typing filters options |

**Events:** `Select.Changed` — property: `value`

```python
yield Select([("Red", "red"), ("Green", "green"), ("Blue", "blue")])
yield Select.from_values(["Small", "Medium", "Large"])
```

### Checkbox

```python
from textual.widgets import Checkbox
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `label` | str | Text next to checkbox |
| `value` | bool | Initial state |

**Events:** `Checkbox.Changed` — property: `value`
**Bindings:** Enter/Space to toggle

```python
yield Checkbox("Enable notifications", value=True)
```

### RadioButton / RadioSet

```python
from textual.widgets import RadioButton, RadioSet
```

Use `RadioSet` for mutual exclusivity:

```python
with RadioSet():
    yield RadioButton("Option A")
    yield RadioButton("Option B", value=True)  # Pre-selected
    yield RadioButton("Option C")
```

**Events:** `RadioSet.Changed` — property: `pressed` (the selected RadioButton)

### Switch

```python
from textual.widgets import Switch
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `value` | bool | Initial state |
| `animate` | bool | Toggle animation |

**Events:** `Switch.Changed` — properties: `value`, `switch`

```python
yield Switch(value=False)
```

### OptionList

```python
from textual.widgets import OptionList
from textual.widgets.option_list import Option
```

Scrollable list of selectable options.

**Events:** `OptionList.OptionHighlighted`, `OptionList.OptionSelected`

```python
yield OptionList(
    Option("Item 1", id="opt1"),
    Option("Item 2", id="opt2"),
    None,  # Separator
    Option("Item 3", id="opt3"),
)
```

Methods: `add_option()`, `remove_option()`, `clear_options()`, `enable_option()`, `disable_option()`

---

## Display Widgets

### Static

```python
from textual.widgets import Static
```

Base widget for displaying text or Rich renderables. Supports console markup.

| Parameter | Type | Description |
|-----------|------|-------------|
| `content` | str/Renderable | Content to display |
| `markup` | bool | Parse Rich markup (default: True) |
| `expand` | bool | Expand to fill container |

```python
yield Static("Hello [bold]World[/bold]!")
yield Static(Panel("Rich panel content"))
```

Update dynamically: `widget.update("New content")`

### Label

```python
from textual.widgets import Label
```

Inherits from Static. For non-interactive text display.

```python
yield Label("Username:")
```

### Digits

```python
from textual.widgets import Digits
```

Large numeric display. Supports: 0-9, A-F, +, -, ^, :, ×

```python
yield Digits("3.14159")
yield Digits("12:30")
```

### Rule

```python
from textual.widgets import Rule
```

Visual separator line.

| Parameter | Type | Description |
|-----------|------|-------------|
| `orientation` | str | `"horizontal"` or `"vertical"` |
| `line_style` | str | `"solid"`, `"heavy"`, `"dashed"`, `"double"`, `"ascii"`, `"thick"` |

```python
yield Rule()
yield Rule(line_style="heavy")
yield Rule.vertical()
```

### Sparkline

```python
from textual.widgets import Sparkline
```

Compact data visualization as proportional bars.

```python
data = [1, 2, 4, 3, 1, 8, 2, 5]
yield Sparkline(data, summary_function=max)
```

### Placeholder

```python
from textual.widgets import Placeholder
```

Layout prototyping widget. Variants: `"default"`, `"size"`, `"text"`.

```python
yield Placeholder("Sidebar", variant="size")
```

---

## Text & Content

### Markdown / MarkdownViewer

```python
from textual.widgets import Markdown, MarkdownViewer
```

Render markdown content. `MarkdownViewer` adds a table of contents sidebar.

**Events:** `Markdown.LinkClicked`, `Markdown.TableOfContentsUpdated`

```python
yield Markdown("# Title\n\nSome **bold** text")
yield MarkdownViewer("# Docs\n\nContent here")
```

Methods: `update(md)`, `append(md)`, `load(path)`, `goto_anchor(anchor)`

### RichLog

```python
from textual.widgets import RichLog
```

Scrollable log that accepts Rich renderables (tables, panels, syntax, etc.).

| Parameter | Type | Description |
|-----------|------|-------------|
| `highlight` | bool | Syntax highlighting |
| `max_lines` | int | Max lines to keep |
| `auto_scroll` | bool | Auto-scroll to bottom |
| `markup` | bool | Parse Rich markup |
| `wrap` | bool | Wrap long lines |

```python
log = self.query_one(RichLog)
log.write("Plain text")
log.write(Syntax(code, "python"))
log.write(Table(...))
```

### Log

```python
from textual.widgets import Log
```

Simpler text-only log widget.

```python
log = self.query_one(Log)
log.write_line("Event occurred")
log.write_lines(["Line 1", "Line 2"])
log.clear()
```

---

## Layout & Navigation

### Header / Footer

```python
from textual.widgets import Header, Footer
```

**Header** displays `app.title` and `app.sub_title`. Optional clock.

```python
yield Header(show_clock=True)
```

**Footer** displays active key bindings from `BINDINGS`.

```python
yield Footer()
yield Footer(compact=True)
```

### Tabs / TabbedContent

```python
from textual.widgets import Tabs, Tab, TabbedContent, TabPane
```

**Tabs** — standalone tab bar:

```python
yield Tabs("First", "Second", "Third")

def on_tabs_tab_activated(self, event: Tabs.TabActivated) -> None:
    self.log(f"Active: {event.tab.id}")
```

**TabbedContent** — tabs + content management:

```python
with TabbedContent():
    with TabPane("Overview", id="overview"):
        yield Static("Overview content")
    with TabPane("Details", id="details"):
        yield Static("Details content")
```

Methods: `add_tab()`, `remove_tab()`, `clear()`, `enable()`, `disable()`, `show()`, `hide()`

### ContentSwitcher

```python
from textual.widgets import ContentSwitcher
```

Switch between child widgets by ID:

```python
with ContentSwitcher(initial="page1"):
    yield Static("Page 1", id="page1")
    yield Static("Page 2", id="page2")

# Switch:
self.query_one(ContentSwitcher).current = "page2"
```

### Collapsible

```python
from textual.widgets import Collapsible
```

Expandable/collapsible section:

```python
with Collapsible(title="Advanced Options", collapsed=True):
    yield Input(placeholder="Option 1")
    yield Input(placeholder="Option 2")
```

**Events:** `Collapsible.Collapsed`, `Collapsible.Expanded`

---

## Data & Lists

### DataTable

```python
from textual.widgets import DataTable
```

Interactive table with sortable columns and cursor navigation.

**Events:** `DataTable.RowSelected`, `DataTable.CellSelected`, `DataTable.HeaderSelected`, `DataTable.RowHighlighted`, `DataTable.CellHighlighted`

```python
def compose(self) -> ComposeResult:
    yield DataTable()

def on_mount(self) -> None:
    table = self.query_one(DataTable)
    table.add_columns("Name", "Age", "City")
    table.add_rows([
        ("Alice", 30, "NYC"),
        ("Bob", 25, "London"),
    ])
    table.cursor_type = "row"  # "cell", "row", "column", "none"
```

Methods: `add_column()`, `add_columns()`, `add_row()`, `add_rows()`, `remove_row()`, `clear()`, `sort()`, `update_cell()`

### Tree

```python
from textual.widgets import Tree
```

Hierarchical tree with expand/collapse.

**Events:** `Tree.NodeSelected`, `Tree.NodeExpanded`, `Tree.NodeCollapsed`, `Tree.NodeHighlighted`

```python
tree: Tree[str] = Tree("Root")
tree.root.expand()
branch = tree.root.add("Branch", expand=True)
branch.add_leaf("Leaf 1")
branch.add_leaf("Leaf 2")
yield tree
```

### DirectoryTree

```python
from textual.widgets import DirectoryTree
```

File browser tree. Inherits from Tree.

```python
yield DirectoryTree("/path/to/dir")

def on_directory_tree_file_selected(self, event: DirectoryTree.FileSelected) -> None:
    self.log(f"Selected: {event.path}")
```

### ListView / ListItem

```python
from textual.widgets import ListView, ListItem
```

Scrollable list with keyboard navigation.

**Events:** `ListView.Selected`, `ListView.Highlighted`

```python
yield ListView(
    ListItem(Label("Item 1")),
    ListItem(Label("Item 2")),
    ListItem(Label("Item 3")),
)
```

### SelectionList

```python
from textual.widgets import SelectionList
```

Multi-select list with checkboxes.

```python
yield SelectionList[str](
    ("Option A", "a"),
    ("Option B", "b", True),  # Pre-selected
    ("Option C", "c"),
)
```

**Events:** `SelectionList.SelectedChanged`

---

## Progress & Status

### ProgressBar

```python
from textual.widgets import ProgressBar
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `total` | float | Total steps (None = indeterminate) |
| `show_bar` | bool | Show progress bar |
| `show_percentage` | bool | Show percentage |
| `show_eta` | bool | Show estimated time |

```python
bar = self.query_one(ProgressBar)
bar.total = 100
bar.advance(10)  # Add 10
bar.update(progress=50)  # Set to 50
```

### LoadingIndicator

```python
from textual.widgets import LoadingIndicator
```

Animated loading dots. Blocks input to underlying widgets.

```python
yield LoadingIndicator()
```

### Toast (Notification)

Notifications via `self.notify()`:

```python
self.notify("File saved!", title="Success", severity="information")
self.notify("Something went wrong", severity="error", timeout=5)
```

Severities: `"information"`, `"warning"`, `"error"`
