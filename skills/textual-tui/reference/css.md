# Textual CSS (TCSS) Reference

Textual uses `.tcss` files with CSS-like syntax adapted for the terminal.

## Table of Contents

- [Selectors](#selectors)
- [Pseudo-Classes](#pseudo-classes)
- [Variables & Nesting](#variables--nesting)
- [Layout](#layout)
- [Grid](#grid)
- [Sizing](#sizing)
- [Spacing](#spacing)
- [Borders & Outlines](#borders--outlines)
- [Colors & Background](#colors--background)
- [Text Styling](#text-styling)
- [Alignment](#alignment)
- [Dock & Layers](#dock--layers)
- [Scrollbars](#scrollbars)
- [Display & Visibility](#display--visibility)
- [Animation](#animation)

---

## Selectors

| Type | Syntax | Example |
|------|--------|---------|
| Type | `WidgetClass` | `Button { }` |
| ID | `#id` | `#save { }` |
| Class | `.class` | `.warning { }` |
| Universal | `*` | `* { margin: 0; }` |
| Descendant | `A B` | `#dialog Button { }` |
| Child | `A > B` | `#sidebar > Button { }` |
| Combined | `A.class` | `Button.primary { }` |

### Specificity (highest to lowest)

1. ID selectors count
2. Class/pseudo-class count
3. Type selector count
4. Later rules win ties
5. `!important` overrides all (use sparingly)

---

## Pseudo-Classes

| Pseudo-class | Applies when |
|-------------|--------------|
| `:hover` | Mouse over widget |
| `:focus` | Widget has keyboard focus |
| `:focus-within` | Widget or descendant has focus |
| `:blur` | Widget does not have focus |
| `:disabled` | Widget is disabled |
| `:enabled` | Widget is enabled |
| `:dark` | App is in dark mode |
| `:light` | App is in light mode |
| `:first-child` | First child of parent |
| `:last-child` | Last child of parent |
| `:even` | Even-indexed child |
| `:odd` | Odd-indexed child |

```css
Button:hover { background: $primary-lighten-1; }
Button:disabled { opacity: 0.5; }
Input:focus { border: heavy $accent; }
.item:even { background: $surface; }
```

---

## Variables & Nesting

### CSS Variables

```css
$accent: #1e90ff;
$sidebar-width: 25;
$gap: 1;

#sidebar {
    width: $sidebar-width;
    background: $accent;
    padding: $gap;
}
```

### Nesting (v0.47+)

```css
#dialog {
    border: heavy $primary;

    Button {
        width: 1fr;
    }

    .ok {
        &:hover { background: $success; }
    }
}
```

`&` refers to the parent selector.

### Reset to Default

```css
Button { background: initial; }
```

### Theme Variables

Semantic colors that adapt to dark/light mode:

| Variable | Purpose |
|----------|---------|
| `$primary` | Primary brand color |
| `$secondary` | Secondary color |
| `$accent` | Accent/highlight |
| `$warning` | Warning state |
| `$error` | Error state |
| `$success` | Success state |
| `$surface` | Surface/card background |
| `$panel` | Panel background |
| `$background` | App background |
| `$foreground` | Default text color |
| `$text` | Text color |
| `$text-muted` | Muted text |
| `$text-disabled` | Disabled text |
| `$primary-lighten-1` | Lighter variant |
| `$primary-darken-1` | Darker variant |
| `$primary-muted` | Muted variant |
| `$text-primary` | Auto text color on primary bg |

---

## Layout

```css
layout: vertical;    /* Default — children stack top-to-bottom */
layout: horizontal;  /* Children flow left-to-right */
layout: grid;        /* Grid matrix layout */
```

Python:
```python
widget.styles.layout = "horizontal"
```

---

## Grid

Only applies when `layout: grid` is set on the container.

| Property | Values | Description |
|----------|--------|-------------|
| `grid-size` | `cols [rows]` | Number of columns and rows |
| `grid-columns` | scalar list | Width of each column |
| `grid-rows` | scalar list | Height of each row |
| `grid-gutter` | `v [h]` | Spacing between cells |
| `column-span` | integer | Columns a child spans |
| `row-span` | integer | Rows a child spans |

```css
#dashboard {
    layout: grid;
    grid-size: 3 2;
    grid-columns: 1fr 2fr 1fr;
    grid-rows: auto 1fr;
    grid-gutter: 1 2;
}

.wide-card { column-span: 2; }
.tall-card { row-span: 2; }
```

Python:
```python
container.styles.grid_size = (3, 2)
container.styles.grid_columns = "1fr 2fr 1fr"
container.styles.grid_gutter = (1, 2)
```

---

## Sizing

### Width & Height

```css
width: 20;           /* Fixed cells (columns) */
width: 50%;          /* Percentage of parent */
width: 1fr;          /* Fraction of remaining space */
width: 50vw;         /* 50% of viewport width */
width: 50vh;         /* 50% of viewport height */
width: 50w;          /* 50% of container width */
width: 50h;          /* 50% of container height */
width: auto;         /* Content-driven */
```

Same units for `height`.

### Constraints

```css
min-width: 20;
max-width: 80;
min-height: 5;
max-height: 30;
```

### Box Sizing

```css
box-sizing: border-box;   /* Default: padding/border reduce content */
box-sizing: content-box;  /* Padding/border expand total size */
```

### Overflow

```css
overflow: auto auto;       /* overflow-x overflow-y */
overflow-x: auto;          /* auto | hidden | scroll */
overflow-y: scroll;
```

---

## Spacing

### Padding (inside border)

```css
padding: 1;               /* All edges */
padding: 1 2;             /* Vertical horizontal */
padding: 1 2 3 4;         /* Top right bottom left */
padding-top: 1;
padding-right: 2;
padding-bottom: 1;
padding-left: 2;
```

### Margin (outside border)

```css
margin: 1;                /* All edges */
margin: 1 2;              /* Vertical horizontal */
margin: 1 2 3 4;          /* Top right bottom left */
margin-top: 1;
margin-right: 2;
margin-bottom: 1;
margin-left: 2;
```

Consecutive margins collapse to the larger value.

---

## Borders & Outlines

### Border Types

`ascii`, `blank`, `dashed`, `double`, `heavy`, `hidden`, `hkey`, `inner`, `outer`, `panel`, `round`, `solid`, `tall`, `thick`, `vkey`, `wide`

### Border Properties

```css
border: heavy white;
border: round $primary;
border-top: dashed red;
border-right: solid green;
border-bottom: double blue;
border-left: heavy yellow;
```

### Border Title

```css
border-title-align: center;     /* left | center | right */
border-subtitle-align: right;
border-title-color: $accent;
border-title-style: bold;
```

Python:
```python
widget.border_title = "Settings"
widget.border_subtitle = "v1.0"
```

### Outline

Secondary border outside the main border. Cannot coexist with border on same edge.

```css
outline: heavy $accent;
outline-top: solid red;
```

---

## Colors & Background

### Color Formats

```css
color: red;                /* Named color */
color: #ff0000;            /* Hex RGB */
color: #f00;               /* Short hex */
color: rgb(255, 0, 0);     /* RGB */
color: hsl(0, 100%, 50%);  /* HSL */
color: auto;               /* Auto high-contrast (black or white) */
```

### Opacity

```css
color: red 50%;            /* Color with opacity */
background: blue 80%;
opacity: 0.5;              /* Widget opacity */
text-opacity: 0.7;         /* Text-only opacity */
```

### Properties

```css
color: $text;              /* Text color */
background: $surface;      /* Background color */
tint: blue 20%;            /* Color overlay */
background-tint: red 10%;  /* Background overlay */
```

---

## Text Styling

```css
text-align: left;          /* left | center | right | justify */
text-style: bold;          /* bold | italic | underline | strike | reverse | none */
text-style: bold italic;   /* Combine multiple */
text-overflow: ellipsis;   /* ellipsis | fold */
text-wrap: wrap;           /* wrap | nowrap */
```

---

## Alignment

### Content Alignment (children within a container)

```css
align: center middle;
align-horizontal: center;   /* left | center | right */
align-vertical: middle;     /* top | middle | bottom */
```

### Content Align (content within the widget itself)

```css
content-align: center middle;
content-align-horizontal: center;
content-align-vertical: middle;
```

---

## Dock & Layers

### Dock

Fix widgets to container edges (stay visible on scroll):

```css
dock: top;      /* top | right | bottom | left */
```

Docked widgets are removed from normal flow. Dock order matters — first docked widget is outermost.

### Layers

Control z-order:

```css
/* On parent container */
layers: base overlay popup;

/* On children */
.background { layer: base; }
.dialog { layer: overlay; }
.tooltip { layer: popup; }
```

### Offset

Relative positioning from calculated position:

```css
offset: 2 3;        /* x y */
offset-x: 2;
offset-y: 3;
```

---

## Scrollbars

```css
scrollbar-background: $surface;
scrollbar-color: $primary;
scrollbar-corner-color: $surface;
scrollbar-background-hover: $surface-lighten-1;
scrollbar-color-hover: $primary-lighten-1;
scrollbar-background-active: $surface-lighten-2;
scrollbar-color-active: $primary-lighten-2;
scrollbar-size: 1 1;         /* Horizontal vertical width */
scrollbar-size-horizontal: 1;
scrollbar-size-vertical: 1;
```

---

## Display & Visibility

```css
display: block;     /* Normal display */
display: none;      /* Hide widget (removed from layout) */
visibility: visible;
visibility: hidden; /* Hidden but space preserved */
```

---

## Animation

Animate style properties programmatically:

```python
widget.styles.animate(
    "opacity",
    value=0.0,
    duration=2.0,           # Seconds
    easing="in_out_cubic",  # Easing function
    delay=0.5,              # Delay before start
    on_complete=callback,   # Called when done
)
```

Or with speed instead of duration:

```python
widget.styles.animate("offset", value=(10, 0), speed=50)
```

Preview easing functions: `textual easing`

Common animatable properties: `opacity`, `offset`, `background`, `color`
