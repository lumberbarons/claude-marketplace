---
name: wavejson
description: Generate WaveJSON timing diagrams for digital signals and create HTML viewers to display them. Use when documenting signal timing, creating timing diagrams, analyzing protocol sequences, visualizing digital waveforms (clock, data, control signals), or displaying/rendering WaveDrom diagrams locally.
allowed-tools: Read, Write, Bash
---

# WaveJSON Timing Diagram Generator

Generate WaveJSON format timing diagrams for WaveDrom rendering. Perfect for documenting hardware interfaces, signal timing, protocol sequences, and any digital signal visualization.

**Format Reference:** This skill uses the [WaveJSON specification](https://github.com/wavedrom/schema/blob/master/WaveJSON.md)
and [WaveDrom](https://github.com/wavedrom/wavedrom) rendering engine, both licensed under MIT.

**Additional Resources:**
- `reference.md` - Quick reference card with wave characters, patterns, and validation checklist

## Important

- Keep signal names short, space is at a premium

## WaveJSON Format Overview

WaveJSON uses JSON to describe timing diagrams with these key components:

### Signal Object Structure

```json
{
  "signal": [
    { "name": "clk",  "wave": "p......." },
    { "name": "data", "wave": "x.345x..", "data": ["head", "body", "tail"] },
    { "name": "req",  "wave": "0.1...0." }
  ]
}
```

### Wave Characters

**Clock waves:**
- `p` - positive edge clock (rising edge)
- `n` - negative edge clock (falling edge)
- `P` - positive edge with arrow
- `N` - negative edge with arrow

**Logic levels:**
- `0` - low level
- `1` - high level
- `.` - extends previous state

**Data values:**
- `=` - data value (color 2)
- `2`, `3`, `4`, `5` - data values with different colors
- `x` - undefined/don't care
- `z` - high impedance (tri-state)
- `u` - pull-up
- `d` - pull-down

**Special:**
- `|` - extend cycle and add gap

### Signal Properties

- `name` - Signal label (string)
- `wave` - Timing pattern (string, one char per cycle)
- `data` - Array of labels for data values (corresponds to `=`, `2`-`5` in wave)
- `period` - Signal repetition period (number)
- `phase` - Phase shift, positive=future, negative=delay (number)
- `node` - Markers for arrows: `.` (none), `A-Z` (invisible), others (visible)

### Signal Groups

Create hierarchical groups by using arrays with a label string:

```json
{
  "signal": [
    ["Control Signals",
      { "name": "LE",  "wave": "01.0...." },
      { "name": "WR",  "wave": "1..01.0." }
    ],
    ["Data Bus",
      { "name": "D[7:0]", "wave": "x.3.4..x", "data": ["CTRL", "DATA"] }
    ]
  ]
}
```

### Edges (Arrows)

Add dependency arrows between signal markers:

```json
{
  "signal": [...],
  "edge": [
    "a~>b Setup time",
    "c-~d Hold time",
    "e~|~f Pulse width"
  ]
}
```

**Edge syntax:**
- First char: source node (from `node` property)
- Last char: destination node
- `>` before last char: arrow head
- Shape: `-` (horizontal), `|` (vertical), `~` (curvy), `/` (diagonal)
- `#` marks label position
- Text after first word: arrow label

### Config

```json
{
  "signal": [...],
  "config": { "hscale": 2 }
}
```

- `hscale` - Horizontal scale (≥1, default 1)

## Generating HTML Viewers

Create local HTML files to render WaveJSON diagrams using the WaveDrom library.

### When to Generate HTML Viewers

Generate an HTML viewer when:
- User wants to "view", "display", "render", or "visualize" a diagram locally
- User wants to save diagrams for documentation
- User needs to see multiple diagrams on one page
- User wants an offline/local viewer instead of online editor

### HTML Template Structure

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>WaveDrom Timing Diagram</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/wavedrom/3.1.0/skins/default.js" type="text/javascript"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/wavedrom/3.1.0/wavedrom.min.js" type="text/javascript"></script>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 20px;
      background-color: #f5f5f5;
    }
    h1, h2 {
      color: #333;
    }
    .diagram-container {
      background: white;
      padding: 20px;
      margin: 20px 0;
      border-radius: 5px;
      box-shadow: 0 2px 4px rgba(0,0,0,0.1);
    }
  </style>
</head>
<body onload="WaveDrom.ProcessAll()">
  <h1>Timing Diagrams</h1>

  <div class="diagram-container">
    <h2>Diagram Title Here</h2>
    <script type="WaveDrom">
    {
      "signal": [
        { "name": "clk", "wave": "p......." },
        { "name": "data", "wave": "x.345x..", "data": ["head", "body", "tail"] }
      ]
    }
    </script>
  </div>

  <!-- Add more diagrams by adding more diagram-container divs -->

</body>
</html>
```

### Multi-Diagram HTML Template

For pages with multiple timing diagrams:

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>Hardware Timing Reference</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/wavedrom/3.1.0/skins/default.js" type="text/javascript"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/wavedrom/3.1.0/wavedrom.min.js" type="text/javascript"></script>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 20px;
      background-color: #f5f5f5;
    }
    h1 { color: #333; }
    h2 { color: #555; margin-top: 0; }
    .diagram-container {
      background: white;
      padding: 20px;
      margin: 20px 0;
      border-radius: 5px;
      box-shadow: 0 2px 4px rgba(0,0,0,0.1);
    }
    .description {
      color: #666;
      margin: 10px 0;
      font-size: 14px;
    }
  </style>
</head>
<body onload="WaveDrom.ProcessAll()">
  <h1>Project Timing Diagrams</h1>

  <div class="diagram-container">
    <h2>Two-Phase Write Sequence</h2>
    <p class="description">Shows the complete writeByte() operation with control latch and character write.</p>
    <script type="WaveDrom">
    { /* First WaveJSON here */ }
    </script>
  </div>

  <div class="diagram-container">
    <h2>Control Word Timing</h2>
    <p class="description">D7 clear bit requires 1.0µs delay per PD2437 datasheet.</p>
    <script type="WaveDrom">
    { /* Second WaveJSON here */ }
    </script>
  </div>

</body>
</html>
```

### HTML Generation Workflow

When user requests HTML viewer:

1. **Generate or use existing WaveJSON** for the timing diagram(s)
2. **Create HTML file** using the template above
3. **Embed WaveJSON** in `<script type="WaveDrom">` tags
4. **Add descriptive titles** (h2) and descriptions for each diagram
5. **Save to appropriate location** (suggest `docs/timing_diagrams.html`)
6. **Offer to open** in browser using `open` command (macOS) or `xdg-open` (Linux)

### Opening HTML in Browser

After creating the HTML file:

```bash
# macOS
open /path/to/timing_diagrams.html

# Linux
xdg-open /path/to/timing_diagrams.html

# Windows (Git Bash)
start /path/to/timing_diagrams.html
```

### Example HTML Generation

**User request:** "Create an HTML page showing the SPI transfer timing"

**Claude should:**
1. Generate (or retrieve) the WaveJSON for SPI transfer
2. Create HTML file with WaveDrom CDN scripts
3. Embed the WaveJSON in `<script type="WaveDrom">` tag
4. Add meaningful title and description
5. Save to `docs/spi_transfer_timing.html`
6. Ask if user wants to open it in browser

### Suggested File Locations

- Single diagrams: `docs/timing_[description].html`
- Protocol reference: `docs/protocol_timing.html`
- Interface-specific: `docs/[interface]_timing.html`
- Complete hardware reference: `docs/hardware_timing.html`

### Including Multiple Diagrams

When creating comprehensive timing documentation:

1. **Group related diagrams**: Organize by protocol, interface, or functional block
2. **Add navigation**: Consider adding a table of contents for pages with 5+ diagrams
3. **Include code references**: Link to specific functions or modules in descriptions
4. **Add context**: Explain timing requirements, constraints, and why they matter

### Validation Before Opening

Before generating HTML:
- Ensure WaveJSON is valid (proper JSON syntax)
- Test WaveJSON in online editor if uncertain
- Check that all required CDN scripts are included
- Verify `onload="WaveDrom.ProcessAll()"` is present in `<body>` tag
- Confirm `<script type="WaveDrom">` tags are properly formed

### WaveDrom CDN Version

Current stable version: **3.1.0**

If user reports rendering issues:
- Check browser console for errors
- Verify internet connection (CDN requires network access)
- Try online editor to isolate WaveJSON vs HTML issues
- Consider downloading WaveDrom for offline use

## Common Digital Signal Patterns

### SPI/Serial Bus Transfer

```json
{
  "signal": [
    { "name": "CLK",  "wave": "p......." },
    { "name": "MOSI", "wave": "x2345678x", "data": ["D7", "D6", "D5", "D4", "D3", "D2", "D1", "D0"] },
    { "name": "MISO", "wave": "x........" },
    { "name": "CS",   "wave": "10.....1." }
  ],
  "config": { "hscale": 2 }
}
```

### I2C Transaction

```json
{
  "signal": [
    { "name": "SCL",  "wave": "0.p......." },
    { "name": "SDA",  "wave": "10.3.4.5.1", "data": ["ADDR", "ACK", "DATA"], "node": ".a.b.c.d.e" },
    {},
    { "name": "Event", "wave": "x2.3.4.5.x", "data": ["START", "ADDR", "ACK", "DATA", "STOP"] }
  ],
  "edge": [
    "a~>b Start condition",
    "c~>d ACK timing"
  ],
  "config": { "hscale": 2 }
}
```

### Bus Write Cycle with Strobes

```json
{
  "signal": [
    { "name": "ADDR", "wave": "x.2.....x", "data": ["0x100"] },
    { "name": "DATA", "wave": "x...3...x", "data": ["0xAB"], "node": "....a..." },
    { "name": "WR",   "wave": "1....01..", "node": ".....bc." },
    { "name": "CS",   "wave": "1.0.....1" }
  ],
  "edge": [
    "a~>b Data setup time",
    "c~>a Data hold time"
  ],
  "config": { "hscale": 2 }
}
```

### Chip Select Decoding (Active Low)

```json
{
  "signal": [
    { "name": "ADDR[1:0]", "wave": "2.3.4.5.", "data": ["00", "01", "10", "11"] },
    {},
    ["Chip Selects (Active Low)",
      { "name": "CS0", "wave": "0.1.1.1." },
      { "name": "CS1", "wave": "1.0.1.1." },
      { "name": "CS2", "wave": "1.1.0.1." },
      { "name": "CS3", "wave": "1.1.1.0." }
    ]
  ],
  "config": { "hscale": 2 }
}
```

### Clock Domain Crossing

```json
{
  "signal": [
    ["Clock Domain A",
      { "name": "CLK_A", "wave": "p........" },
      { "name": "DATA_A", "wave": "x2.....x.", "data": ["DATA"], "node": ".a......" }
    ],
    {},
    ["Synchronizer",
      { "name": "SYNC1", "wave": "x..2...x.", "node": "...b...." },
      { "name": "SYNC2", "wave": "x....2.x.", "node": ".....c.." }
    ],
    {},
    ["Clock Domain B",
      { "name": "CLK_B", "wave": "P........" },
      { "name": "DATA_B", "wave": "x......2x", "data": ["DATA"], "node": "......d." }
    ]
  ],
  "edge": [
    "a~>b Meta-stability",
    "c~>d Synchronized"
  ],
  "config": { "hscale": 2 }
}
```

### Setup and Hold Timing

```json
{
  "signal": [
    { "name": "CLK",  "wave": "0...1.0...", "node": "....a.b..." },
    { "name": "DATA", "wave": "x.2...3..x", "data": ["Old", "New"], "node": "..c.....d." },
    {},
    { "name": "Setup",  "wave": "x.2.x.....", "data": ["t_su"], "node": "..e......." },
    { "name": "Hold",   "wave": "x......2.x", "data": ["t_h"], "node": ".......f.." }
  ],
  "edge": [
    "c~>a Setup time (data before clock)",
    "b~>d Hold time (data after clock)"
  ],
  "config": { "hscale": 3 }
}
```

## Instructions for Claude

### Generating WaveJSON Only

When asked to generate WaveJSON timing diagrams:

1. **Understand the sequence**: Identify all signals involved and their timing relationships
2. **Choose appropriate wave characters**: Use clocks (`p`/`n`) for clocks, logic levels (`0`/`1`) for control, data (`=`, `2`-`5`) for bus values
3. **Add signal groups**: Organize related signals hierarchically for clarity
4. **Include data labels**: Use the `data` array to label bus values meaningfully
5. **Add edges/arrows**: Show timing relationships (setup, hold, propagation delay) using the `edge` array
6. **Set appropriate hscale**: Use `hscale: 2` or higher for complex timing to make diagrams readable
7. **Add nodes**: Place markers (using `node` property) where arrows should connect
8. **Output valid JSON**: Ensure proper JSON syntax (double quotes, no trailing commas)
9. **Test with context**: Consider IC datasheets and actual timing constraints from hardware docs

### Generating HTML Viewer

When asked to "view", "display", "render", or "visualize" timing diagrams locally:

1. **Generate WaveJSON first** (or retrieve from examples.md if project-specific examples exist)
2. **Create docs directory** if it doesn't exist: `mkdir -p docs`
3. **Write HTML file** using the template structure:
   - Include WaveDrom CDN scripts (v3.1.0)
   - Add `onload="WaveDrom.ProcessAll()"` to body tag
   - Embed WaveJSON in `<script type="WaveDrom">` tags
   - Add meaningful titles and descriptions
4. **Save with descriptive filename** in `docs/` directory
5. **Offer to open** in browser (macOS: `open`, Linux: `xdg-open`, Windows: `start`)

### Decision: WaveJSON vs HTML

- **Just WaveJSON**: User asks to "generate" or "create" a diagram, wants to paste in online editor
- **HTML Viewer**: User asks to "view", "display", "show", "render", or wants local/saved documentation

When in doubt, generate WaveJSON and ask: "Would you like me to create an HTML viewer to display this locally?"

### Node Placement for Arrows

To draw arrows between events:

1. Add `node` property to signals with timing markers
2. Use `.` for cycles without markers
3. Use letters `a-z` or `A-Z` for marker positions
4. Markers align with wave characters (one char per cycle)

Example:
```json
{ "name": "CLK",  "wave": "p....", "node": ".a.b." }
{ "name": "DATA", "wave": "x.2.x", "node": "..c..", "data": ["valid"] }
```

Then add edges:
```json
"edge": [
  "a~>c Setup",
  "b~>c Hold"
]
```

## Output Format

Generate WaveJSON as properly formatted JSON that can be:
1. Pasted into [WaveDrom Editor](https://wavedrom.com/editor.html) for rendering
2. Saved as `.json` files
3. Embedded in markdown with WaveDrom plugins

Always validate JSON syntax and test logic before outputting.

## Context and Timing Considerations

When generating timing diagrams for hardware projects:

- **Setup/Hold times**: Show data stability before and after clock edges
- **Propagation delays**: Illustrate signal delays through logic and buffers
- **Bus protocols**: Display address/data multiplexing, strobes, and handshaking
- **Clock domains**: Show synchronization between different clock domains
- **Level conversion**: Consider voltage level timing when crossing logic families

Reference datasheets, project documentation, and code for actual timing values.

## Example Usage

**User:** "Generate a WaveJSON diagram showing an SPI byte transfer"

**Claude should:**
1. Read relevant code/documentation to understand timing
2. Create signals for CLK, MOSI, MISO, CS
3. Show chip select assertion
4. Show 8 clock cycles with data shifting
5. Show chip select de-assertion
6. Add appropriate hscale and labels
7. Output valid WaveJSON

## Validation

Before outputting WaveJSON:
- Check JSON syntax (use linter if needed)
- Verify wave strings match in length (if they should)
- Ensure `data` array matches number of data values in `wave`
- Confirm node markers align with cycles
- Test edge references point to existing nodes
