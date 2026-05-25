# JSON Path Extractor & Exporter

**Product Requirements Document**

**Version:** 2.1  
**Date:** May 25, 2026  
**Status:** Approved  

---

## 1. Executive Summary

The JSON Path Extractor & Exporter is a lightweight, offline-first web utility designed for software engineers, data analysts, and QA engineers. It allows users to paste or upload raw JSON, dynamically evaluate JSONPath queries or select specific keys via an interactive panel, instantly preview filtered results with syntax highlighting, and export the output into multiple formats (TXT, CSV, PDF, Print). The application features a highly customizable, resizable interface with theme and accent color support, regex-powered key filtering, and an embedded offline regex reference guide. All dependencies (JavaScript libraries, CSS, and font files) are bundled locally, enabling full functionality without any internet connection.

Version 2.0 introduced three major enhancements: (1) a fixed Print workflow that isolates only the results output for clean printing, (2) a regex search mode for the key extraction panel that allows users to filter keys using full JavaScript regular expression syntax, and (3) an embedded Regex Quick Reference modal sourced from RexEgg that provides offline-accessible regex documentation. Version 2.1 restructured the project into an organized folder layout (`js/`, `css/`, `fonts/`) with all CDN dependencies downloaded locally, making the application fully offline-capable with zero internet dependency.

---

## 2. User Personas

### 2.1 Software Engineers

Software engineers need a fast, offline tool to test JSONPath queries during API integration or configuration parsing without server round-trips. They frequently work with nested JSON payloads from REST APIs and need to quickly extract specific fields, validate response structures, or transform data for further processing. The regex search capability allows them to find keys matching complex patterns across large API response objects, while the JSONPath engine supports advanced traversal with recursive descent, array slicing, and filter expressions.

### 2.2 Data Analysts

Data analysts need to extract specific nested arrays or keys from large JSON datasets and convert them to CSV for spreadsheet analysis. They often receive JSON exports from databases, APIs, or logging systems and need to flatten hierarchical data into tabular formats. The key extraction panel with regex filtering enables analysts to quickly locate all price-related fields, timestamp fields, or any other pattern-matched keys across complex nested structures, while the CSV export handles nested object stringification automatically.

### 2.3 QA Engineers

QA engineers need to validate API payload structures, ensure data uniqueness, and save or print PDF reports of targeted data segments for test evidence. The unique toggle allows them to quickly verify deduplication requirements, the print feature produces clean output-only reports for test documentation, and the filter expressions support conditional extraction such as finding all items where a price exceeds a threshold. The offline regex reference is particularly valuable for QA engineers who need to construct precise pattern-matching queries during testing sessions.

---

## 3. Functional Requirements

### 3.1 JSON Input Panel

#### 3.1.1 Data Entry

The application must provide a large, monospace text area (JetBrains Mono font) for pasting raw JSON content. The text area must support Tab key insertion of two spaces for manual editing convenience. When the user pastes valid JSON, the system must automatically format and indent the content for readability using 2-space indentation. Invalid JSON should remain unformatted so the user can see their original input and identify errors.

#### 3.1.2 File Upload

The system must support loading `.json` files via a file picker button (Load) and drag-and-drop. The text area must act as a drop zone with a visual overlay indicator (dashed accent-colored border with icon and text) when a file is dragged over it. Upon drop, the file must be read and populated into the text area, then automatically formatted. Only files with `.json` extension or `application/json` MIME type should be accepted; other files must trigger an error toast notification.

#### 3.1.3 Validation Status

A dynamic status indicator (colored dot) must appear next to the panel title, showing whether the current text area content is **Valid JSON** (green dot), **Invalid JSON** (red dot), or **Empty** (gray dot). This indicator must update in real-time as the user types or modifies the content.

#### 3.1.4 Action Buttons

The panel must include three action buttons:

| Button | Icon | Behavior |
|--------|------|----------|
| **Format** | `fa-align-left` | Manual pretty-print with toast confirmation |
| **Clear** | `fa-trash-can` | Reset all state including query, keys, and results |
| **Load** | `fa-folder-open` | Trigger the hidden file picker |

All buttons must have icons and be keyboard-accessible.

---

### 3.2 Key Extraction & Selection Panel

#### 3.2.1 Automatic Extraction

Upon valid JSON input, the system must automatically parse the structure and extract all leaf-node key paths. For array elements, the system uses wildcard notation (e.g., `$.store.book[*].author`) to represent all elements rather than enumerating each index separately. This deduplication ensures the key list remains concise and useful even for large arrays. The extraction algorithm walks the entire JSON tree recursively, collecting paths to all primitive values, null values, and empty containers.

#### 3.2.2 Key List UI

Extracted keys must be displayed in a scrollable list with checkboxes for multi-selection. Each key item shows the full JSONPath and supports click-to-toggle on the entire row. The list must support horizontal scrolling for long paths and show ellipsis truncation with hover tooltips.

#### 3.2.3 Search/Filter

The panel must provide a search input to filter the key list. Two modes are supported:

- **Plain text search** (default): Case-insensitive substring matching
- **Regex search** (activated via toggle): Accepts JavaScript regular expression syntax with case-insensitive matching

In regex mode, invalid patterns must display an inline error message below the search field without disrupting the application. The placeholder text must update to indicate the current mode:

| Mode | Placeholder |
|------|-------------|
| Plain text | `Filter keys...` |
| Regex | `Filter keys with regex...` |

#### 3.2.4 Regex Help

A **question-mark icon** button next to the regex toggle must open a modal dialog containing a comprehensive **Regex Quick Reference**. This reference is embedded directly in the HTML for offline use and includes tables covering the following categories:

| Section | Contents |
|---------|----------|
| **Characters** | `\d`, `\w`, `\s`, `\D`, `\W`, `\S`, `.`, `\` |
| **Quantifiers** | `+`, `{n}`, `{m,n}`, `{n,}`, `*`, `?`, lazy variants |
| **Logic** | `\|`, capturing groups, backreferences, non-capturing groups |
| **Character Classes** | `[...]`, ranges, negation |
| **Anchors & Boundaries** | `^`, `$`, `\b`, `\B` |
| **Lookarounds** | Positive/negative lookahead/lookbehind |
| **White-Space** | `\t`, `\n`, `\r` |
| **Inline Modifiers** | `i`, `g`, `m`, `s` (JavaScript) |
| **Other Syntax** | `\K`, `\Q...\E` |

Each table includes **Pattern**, **Meaning**, **Example**, and **Sample Match** columns. A contextual tip explains how to use regex in the key filter (e.g., `\.price$` matches all keys ending with ".price").

The modal must be closeable via:
- The **X** button
- Clicking the **overlay backdrop**
- Pressing the **Escape** key

Source attribution: Based on the [RexEgg Cheat Sheet](https://www.rexegg.com/regex-quickstart.php).

#### 3.2.5 Bulk Actions

Include **All** and **None** buttons to quickly select or deselect all visible (filtered) keys. The All button must respect the current filter (both plain text and regex modes), selecting only keys that match the current search criteria rather than all keys globally.

#### 3.2.6 Dual-Mode Evaluation

If a user selects keys without writing a JSONPath query, the tool must evaluate the selected keys and display the combined result. The result combines all selected key values into a single object keyed by their paths, or into a flat array if all values are primitives.

---

### 3.3 JSONPath Query Panel

#### 3.3.1 Query Input

A prominent text input field for entering JSONPath expressions (Placeholder: `$.store.book[*].author`). The field must default to empty on load. The input uses monospace font (JetBrains Mono) for readability of path syntax.

#### 3.3.2 Quick Chips

Provide clickable "Try" chips for common queries to populate the input field instantly:

| Chip | Query | Description |
|------|-------|-------------|
| Root $ | `$` | Return the entire root object |
| All values ..* | `..*` | Recursively return all values |
| First $[0] | `$[0]` | First element of root array |
| Last $[-1] | `$[-1]` | Last element of root array |
| Slice [:3] | `$[0:3]` | First three elements |
| Recursive ..key | `..key` | All values of "key" at any depth |

Each chip must be keyboard-accessible with focus-visible styling.

#### 3.3.3 Deduplication Toggle

A **Unique** toggle switch to filter duplicate entries from the extracted result array. When enabled, results are deduplicated based on `JSON.stringify` comparison. The toggle state is visually indicated by the track color matching the accent color when active.

#### 3.3.4 Mode Indicator

A visual pill indicator showing whether results are derived from:

| Mode | Background | Text Color | Label |
|------|-----------|------------|-------|
| Query | Blue | Dark blue | `QUERY` |
| Keys | Amber | Dark amber | `KEYS` |

The pill must be hidden when no results are displayed.

---

### 3.4 Evaluation Engine

#### 3.4.1 Real-Time Processing

Evaluation must occur automatically (debounced by **150ms**) as the user types a query, toggles deduplication, or selects/deselects keys. The debounce timer resets on each input event to prevent excessive evaluations during rapid typing.

#### 3.4.2 Custom Evaluator

The system must utilize a built-in custom JSONPath evaluation engine supporting the following syntax:

| Syntax | Description | Example |
|--------|-------------|---------|
| `$` | Root reference | `$` |
| `.property` | Child property access | `$.store` |
| `[*]` | Array wildcard (all elements) | `$.book[*]` |
| `[n]` | Array index | `$[0]` |
| `[-n]` | Negative index from end | `$[-1]` |
| `[start:end:step]` | Array slicing | `$[0:3]`, `$[::2]` |
| `..property` | Recursive descent | `..author` |
| `..*` | All descendants | `..*` |
| `[?(@.prop)]` | Filter: truthy check | `[?(@.price)]` |
| `[?(@.prop op value)]` | Filter: comparison | `[?(@.price > 10)]` |

Supported comparison operators: `==`, `!=`, `>`, `<`, `>=`, `<=`, `=~` (regex match).

No external JSONPath library is used to prevent CJS/UMD module loading errors in a browser-only context.

#### 3.4.3 Error Handling

Invalid JSON syntax or broken JSONPath expressions must display clear, inline error messages (red background in both light and dark themes) without crashing the application. The error message must be specific enough to help the user correct their expression. Errors in the key filter regex are shown inline below the search input separately from query errors.

---

### 3.5 Output & Results Panel

#### 3.5.1 Result Display

Must display the extracted data in a readable, formatted JSON structure with syntax highlighting. Different token types must be visually distinguished:

| Token Type | Light Theme | Dark Theme | Style |
|-----------|-------------|------------|-------|
| Keys | Red/pink | Light red | Normal |
| Strings | Blue | Light blue | Normal |
| Numbers | Purple | Light purple | Normal |
| Booleans | Magenta | Light magenta | Normal |
| Nulls | Gray | Gray | Italic |

#### 3.5.2 Scrollable Area

The result container must be independently scrollable both vertically and horizontally. The container uses monospace font with `pre`-formatted whitespace and 2-space tab size.

#### 3.5.3 Match Count

Display a dynamic badge showing the number of matches found. The badge uses the accent color with a light tint background and displays singular or plural form appropriately (e.g., "1 match" vs "5 matches").

#### 3.5.4 Copy to Clipboard

A **Copy** button that copies the exact JSON text to the user's clipboard using the Clipboard API, with a fallback to `document.execCommand('copy')` for older browsers. A toast notification must confirm successful copy.

---

### 3.6 Export Capabilities

#### 3.6.1 Save as Text

Triggers a browser download of a `.txt` file containing the raw JSON string. The file is generated using the Blob API with `text/plain` MIME type and downloaded via a temporary anchor element with `URL.createObjectURL`.

#### 3.6.2 Save as CSV

Flattens the extracted JSON array or objects into rows and columns. Nested objects are stringified as JSON within cells. The CSV generator handles comma, quote, and newline escaping per RFC 4180 standards. Headers are derived from object keys; arrays of primitives use a single `value` column.

#### 3.6.3 Save as PDF

Compiles the output into a clean, white-backgrounded, multi-page layout using html2pdf.js and downloads a `.pdf` file. Configuration:

| Setting | Value |
|---------|-------|
| Margin | 10mm |
| Image quality | JPEG, 0.98 |
| Scale | 2x |
| Format | A4 portrait |

Must fallback to the Print dialog if the html2pdf.js library fails to load.

#### 3.6.4 Print

Opens the native browser print window. **Only the results output section is printed** — all UI panels (header, input panels, query panel, export bar) must be hidden. The print layout displays a dedicated `#printArea` div containing:

1. **Title header** — "JSON Path Extractor — Output"
2. **Metadata line** — Active query, match count, print timestamp
3. **Formatted JSON results** — Pre-wrapped layout on white background with black text

This is achieved by showing a hidden `#printArea` div exclusively during print and hiding all other body content via CSS `@media print` rules using `body > *:not(#printArea) { display: none !important; }`. The print area auto-cleans (hidden and emptied) 1 second after `window.print()` returns.

---

## 4. User Interface & User Experience Requirements

### 4.1 Layout & Resizers

#### 4.1.1 Split-Screen Design

A clean, **50/50 split-screen** design with Inputs on the left column (JSON Input and Key Extraction stacked vertically) and Outputs on the right column (JSONPath Query, Results, and Export bar stacked vertically). Both columns use CSS Flexbox for layout with flex-based sizing.

#### 4.1.2 Customizable Dividers

The application must feature draggable dividers between all major panels:

| Divider | Location | Direction |
|---------|----------|-----------|
| Vertical resizer | Between left and right columns | Column resize |
| Horizontal resizer | Between JSON Input and Key Extraction | Row resize |
| Mobile resizer | Between columns (below 960px) | Row resize |

#### 4.1.3 Resize Logic

Dragging a divider must resize the adjacent panels using CSS Flexbox (`flex: 0 0 [size]px`), enforcing a **minimum panel size of 80px** to prevent collapsing. The resize logic uses `mousedown`/`mousemove`/`mouseup` event handling with cursor feedback and `user-select: none` prevention during drag operations.

### 4.2 Theming & Customization

#### 4.2.1 Dark/Light Mode

A toggle switch to transition between Dark and Light themes. The selection must persist in localStorage under key `jpe-th`. Dark mode uses a dark panel background (`#1a1b23`) with light text, while light mode uses white panels with dark text. All theme values are managed via CSS custom properties on `:root` and `:root.dark` for instant, seamless transitions without page reload.

#### 4.2.2 Accent Color Picker

A dropdown menu allowing users to select from **8 preset accent colors** or input a custom hex color:

| Preset | Hex |
|--------|-----|
| Indigo | `#4f46e5` |
| Violet | `#7c3aed` |
| Blue | `#2563eb` |
| Cyan | `#0891b2` |
| Emerald | `#059669` |
| Amber | `#d97706` |
| Red | `#dc2626` |
| Pink | `#db2777` |

The chosen accent applies to buttons, highlights, resizers, focus rings, chips, toggle tracks, and the match badge. The selection persists in localStorage under key `jpe-ac`.

#### 4.2.3 CSS Variables

When the accent color changes, the application dynamically updates the following CSS custom properties on the root element:

| Variable | Derivation |
|----------|-----------|
| `--accent` | User-selected color |
| `--accent-hover` | Shaded 15 units darker |
| `--accent-light` | 10% opacity |
| `--accent-ring` | 30% opacity |

### 4.3 Accessibility & Feedback

#### 4.3.1 Toast Notifications

Non-intrusive, animated toast messages for user actions. Toasts appear in the bottom-right corner with slide-in animation and auto-dismiss after 2.5 seconds with fade-out. Toast colors adapt to the current theme (dark background in light mode, light background in dark mode).

| Action | Icon | Message |
|--------|------|---------|
| Copy success | `fa-circle-check` | "Copied to clipboard" |
| Export TXT | `fa-circle-check` | "Exported as TXT" |
| Export CSV | `fa-circle-check` | "Exported as CSV" |
| Export PDF | `fa-circle-check` | "Exported as PDF" |
| Format | `fa-align-left` | "JSON formatted" |
| Clear | `fa-trash-can` | "Cleared" |
| File loaded | `fa-folder-open` | "File loaded: {name}" |
| Invalid JSON format | `fa-circle-exclamation` | "Invalid JSON — cannot format" |
| Nothing to export | `fa-circle-exclamation` | "Nothing to export" |
| Wrong file type | `fa-circle-exclamation` | "Please drop a .json file" |

#### 4.3.2 Keyboard Navigation

Interactive toggles (theme, unique, regex) and buttons must be focusable and operable via keyboard (Enter/Space). Focus-visible outlines use the accent ring color. The regex modal must close on Escape key press. All chips are keyboard-navigable with focus-visible styling.

#### 4.3.3 Responsive Design

On screens smaller than **960px**, the layout must stack vertically with the left column above the right column. The vertical column resizer must be hidden and replaced by a horizontal resizer. Both columns use full width with percentage-based flex sizing (45%/55% split).

---

## 5. Technical Constraints & Dependencies

### 5.1 Architecture

The application is delivered as a **local folder structure** containing a primary HTML file and supporting local assets. No build step, server-side backend, or Node.js runtime is required. The entire application runs directly in a modern browser and functions **fully offline** with zero internet dependency — all JavaScript libraries, CSS frameworks, and font files are bundled locally.

**Project folder structure:**

```
json-path-extractor/
├── json-path-extractor.html      # Main application entry point
├── css/
│   ├── google-fonts.css          # @font-face declarations for DM Sans & JetBrains Mono
│   └── fontawesome-all.min.css   # Font Awesome 6.5.1 icon styles
├── js/
│   └── html2pdf.bundle.min.js    # html2pdf.js 0.10.1 (PDF generation)
├── fonts/
│   ├── fa-solid-900.woff2        # Font Awesome Solid icons (woff2)
│   ├── fa-solid-900.ttf          # Font Awesome Solid icons (ttf fallback)
│   ├── fa-regular-400.woff2      # Font Awesome Regular icons (woff2)
│   ├── fa-regular-400.ttf        # Font Awesome Regular icons (ttf fallback)
│   ├── fa-brands-400.woff2       # Font Awesome Brands icons (woff2)
│   ├── fa-brands-400.ttf         # Font Awesome Brands icons (ttf fallback)
│   ├── fa-v4compatibility.woff2  # Font Awesome v4 compatibility (woff2)
│   ├── fa-v4compatibility.ttf    # Font Awesome v4 compatibility (ttf fallback)
│   ├── rP2tp2ywxg089UriI5-...ttf # DM Sans 400
│   ├── rP2tp2ywxg089UriI5-...ttf # DM Sans 500
│   ├── rP2tp2ywxg089UriI5-...ttf # DM Sans 600
│   ├── rP2tp2ywxg089UriI5-...ttf # DM Sans 700
│   ├── tDbY2o-flEEny0FZ-...ttf   # JetBrains Mono 400
│   └── tDbY2o-flEEny0FZ-...ttf   # JetBrains Mono 500
└── PRD/
    └── JSON-Path-Extractor-PRD-v2.0.md  # This document
```

### 5.2 Local Libraries (Offline-Bundled)

All external dependencies have been downloaded and are referenced via **relative paths** from the HTML file. No CDN calls are made at runtime. The application works without any internet connection.

| Library | Version | Purpose | Local Path |
|---------|---------|---------|------------|
| html2pdf.js | 0.10.1 | Client-side PDF generation | `js/html2pdf.bundle.min.js` |
| Font Awesome | 6.5.1 | UI icons (solid + regular) | `css/fontawesome-all.min.css` + `fonts/` |
| Google Fonts | N/A | DM Sans (400–700) + JetBrains Mono (400, 500) | `css/google-fonts.css` + `fonts/` |

**Note:** Tailwind CSS was listed as a dependency in earlier versions but has been removed — the application uses entirely custom CSS via a `<style>` block, making Tailwind unnecessary.

### 5.3 CSS Path Resolution

Both CSS files reference font files via relative `../fonts/` paths:

- `css/google-fonts.css` → `../fonts/*.ttf` (DM Sans, JetBrains Mono)
- `css/fontawesome-all.min.css` → `../fonts/fa-*.woff2` and `../fonts/fa-*.ttf` (Font Awesome icons)

These paths resolve correctly when the folder structure is preserved as documented above.

### 5.4 Custom JSONPath Engine

JSONPath evaluation is handled natively by a custom tokenizer and evaluator written in vanilla JavaScript without external libraries. This design choice prevents CJS/UMD module loading errors that commonly occur when integrating Node.js-focused JSONPath libraries (such as `jsonpath-plus`) in browser environments. The engine supports a comprehensive subset of JSONPath syntax including root references, property access, array operations, slicing, recursive descent, wildcard matching, and filter expressions with comparison operators.

### 5.5 Storage

LocalStorage is used exclusively for persisting user preferences:

| Key | Value | Default |
|-----|-------|---------|
| `jpe-th` | `"light"` or `"dark"` | `"light"` |
| `jpe-ac` | Hex color string | `"#4f46e5"` |

No other data, including JSON input or query history, is persisted to storage.

### 5.6 Browser APIs

| API | Usage | Fallback |
|-----|-------|----------|
| File API | Reading uploaded/dragged JSON files | None (basic browser support) |
| Clipboard API | Copy Result to clipboard | `document.execCommand('copy')` |
| Blob API | Generating downloadable files (TXT, CSV) | None (basic browser support) |
| Window Print API | Print export with dedicated print area | None (basic browser support) |
| RegExp | Regex mode key filtering | Plain text fallback |

---

## 6. Version History

| Version | Date | Changes |
|---------|------|---------|
| **1.0** | May 24, 2026 | Initial release with core features: JSON input with auto-format and drag-drop, key extraction with leaf-node paths, JSONPath evaluation engine, syntax highlighting, export (TXT/CSV/PDF/Print), dark/light theme, 8 accent colors + custom, resizable panels, toast notifications |
| **2.0** | May 25, 2026 | **Fixed Print** to output-only layout with dedicated `#printArea` and metadata; **Added Regex search mode** for key filtering with inline error handling and toggle switch; **Embedded Regex Quick Reference modal** sourced from RexEgg for offline use; Improved leaf-node key extraction with array wildcard deduplication; Added auto-format on file load |
| **2.1** | May 25, 2026 | **Offline-first restructure**: All CDN dependencies (html2pdf.js, Font Awesome CSS + webfonts, Google Fonts CSS + TTF files) downloaded and organized into local `js/`, `css/`, and `fonts/` folders; HTML updated to reference relative local paths; Tailwind CSS removed (unused — all styling is custom CSS); Application now runs fully offline with zero internet dependency |

---

## 7. Future Considerations

The following features are out of scope for the current version but represent natural extensions for future development:

- **JSON Schema validation** against user-provided schemas
- **Query history** with localStorage persistence
- **Comparison mode** for diffing two JSON documents
- **JSON5 and JSONC support** (with comments) for input formats
- **Inline editing** of results with round-trip synchronization to the input
- **WebSocket integration** for live JSON stream processing

Each of these features would require careful consideration of the offline-first architecture constraint and should be bundled locally to maintain zero-internet-dependency operation.
