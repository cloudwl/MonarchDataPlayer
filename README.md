# MonarchDataPlayer — Data Replay Edition

A high-performance signal waveform and data replay tool built with Rust + egui, designed for autonomous driving and industrial testing workflows.

---

## Overview

MonarchDataPlayer loads and visualizes recorded CAN/LIN bus data and sensor signals from a variety of file formats. It supports multi-channel plotting, cursor-based measurement, physical-unit conversion via DBC files, and synchronized video playback.

---

## Supported File Formats

| Format | Extension | Description |
|--------|-----------|-------------|
| **ASAM MDF4** | `.mf4`, `.mdf`, `.dat` | Industry-standard measurement data format |
| **CANoe BLF** | `.blf` | Binary Logging Format from Vector tools |
| **CANoe ASC** | `.asc` | ASCII logging format |
| **Serial Log** | `.json`, `.txt`, `.csv`, `.dat` | Raw serial/UDP capture output |

### MDF4 Data Blocks

The MDF4 reader handles all standard ASAM MDF 4.x data container types:
- `##DT` — direct data blocks
- `##DZ` — compressed blocks (Deflate, Transposed Deflate, LZ4)
- `##HL / ##DL` — list indirection chains

Transposed-deflate blocks are unblocked via zlib inflation followed by stride-based de-transposition.

### BLF/ASC Garden

BLF and ASC files share a unified decode path:
- BLF: Object stream scanning with `LOG_CONTAINER` decompression (`zlib` / `lz4`)
- BLF CAN object types: `CAN_MESSAGE`, `CAN_MESSAGE2`, `CAN_FD_MESSAGE`, `CAN_FD_MESSAGE_64`
- ASC: Line-by-line ASCII parse
- DBC routing: exact `channel_key` match with `*` wildcard fallback
- Auto-sniff: builds CAN-ID sets from log channels and matches against loaded DBC by overlap score

---

## Signal Processing

### DBC Parsing

Load a CAN DBC file to enable:
- Automatic signal name decoding from raw CAN frames
- Physical-unit conversion (linear scaling: `phys = a * raw + b`)
- Bit-level signal extraction across byte boundaries

### Signal Pool

All discovered signals are listed in the **Signal Pool**:
- Grouped by source file
- Drag signals into chart windows to plot
- `Ctrl+A` selects all signals in the active chart

### Virtual Signals

Computed signals prefixed with `Virtual::` are created via the **Math Channel** dialog and support arithmetic expressions over existing signals. Virtual signals are deleted globally (not just removed from chart).

---

## Chart Features

### Multi-Window Layout

- Create multiple independent chart windows
- Drag the splitter to resize the signal list sidebar
- Drag signals between chart windows
- Close non-default chart windows with signal-aware confirmation

### Zoom and Pan

| Action | Control |
|--------|---------|
| **X-axis zoom** | Mouse wheel (anchored to cursor position) |
| **Y-axis zoom** | Drag left Y-axis region |
| **Y-axis pan** | `Ctrl` + drag Y-axis |
| **Marquee zoom** | `Ctrl` + drag on chart area |
| **Pan X** | Drag on chart when not zooming |
| **Undo view** | `Backspace` |
| **Show all** | Dedicated toolbar button |

Marquee zoom maintains a history stack; `Backspace` steps backward.

### Multi-Signal Chart

Signals are color-coded using a golden-angle HSV distribution to minimize collisions in large sets. Each signal carries its own style (color, stroke width, interpolation mode).

### Grid and Background

- Toggle grid via **View → Show Grid**
- Chart background follows app theme, or can be forced to **Black** / **White**

---

## Cursor Measurement

| Key | Action |
|-----|--------|
| `<` (comma) | Jump to cursor A |
| `>` (period) | Jump to cursor B |
| `?` (slash) | Clear both cursors |

When both cursors are active, the time delta Δt is displayed.

---

## Playback Controls

- **Play / Pause** — step through recorded data by time
- **Timeline drag** — seek by dragging the playback position
- **Show All** — reset view to display the full recording range
- **Export scope** — for MF4 export, choose: Whole / Visible / Cursor-A-to-C time zone

---

## Export Capabilities

| Format | Menu Entry | Notes |
|--------|-----------|-------|
| **CSV** | `File → Export to CSV...` | Wide format for serial/log; long format (`Timestamp,Channel,Value`) for MDF/BLF |
| **MF4** | `File → Export to MF4...` | Exports selected time scope as ASAM MDF4 |
| **MATLAB** | `Fancy → Export to MATLAB` | `InputScenario` (Simulink Dataset) or `Timeseries` |
| **Excel** | `Fancy → DBC to Excel...` | Converts loaded DBC to spreadsheet |

---

## Video Sync

MonarchDataPlayer can synchronize external `mpv` video windows to the data playback timeline.

### Setup

1. **View → Video Sync...**
2. Load one or more video files
3. Adjust per-video offset (`Video Time = Waveform Time − Offset`)

### mpv Resolution Order

If `mpv` is not found on `PATH`, the app searches:
1. `MONARCH_MPV_EXE` environment variable (full path)
2. `MONARCH_MPV_DIR` environment variable (directory)
3. `./runtime/mpv/`
4. `./mpv/`
5. `./bin/mpv/`
6. System `PATH`

---

## Analysis Tools

Available under **Tools** menu:

| Tool | License | Description |
|------|---------|-------------|
| **Math Channel** | Basic | Arithmetic expressions over signal names |
| **X-Y Plot** | Basic | Plot one signal vs. another |
| **Script Engine** | Standard/Pro | Rhai scripting on signal data |
| **Statistics** | Standard/Pro | Min/max/mean/RMS per signal |
| **FFT Analysis** | Standard/Pro | Frequency spectrum display |

---

## Keyboard Shortcuts

| Feature | Shortcut |
|---------|----------|
| Marquee zoom | `Ctrl` + drag |
| Y-axis zoom | Drag left Y-axis area |
| Y-axis pan | `Ctrl` + drag Y-axis |
| Cursor A | `<` |
| Cursor B | `>` |
| Clear cursors | `?` |
| Undo view | `Backspace` |
| Fullscreen | `F` |
| Select all signals | `Ctrl+A` |
| Delete selected | `Delete` |
| Copy variable name | `Ctrl+C` |

---

## Configuration and Persistence

- **Save / Load Layout** — `File → Save Configuration...` / `Load Configuration...`
- **Reset All** — top-bar `Reset All` button returns app to clean idle state
- Layout includes chart window positions, signal assignments, and view state

---

## Technical Stack

- **UI**: eframe / egui
- **Serial I/O**: serialport
- **Data formats**: MDF4 (ASAM), BLF/ASC (Vector), CSV
- **CAN DBC**: can-dbc 8.1.x
- **A2L parsing**: a2lfile 3.3.x
- **Scripting**: Rhai
- **FFT**: rustfft
- **Parallelism**: rayon
- **Video**: external `mpv` via JSON IPC
