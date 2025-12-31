# WaveJSON Quick Reference Card

Original spec: https://github.com/wavedrom/schema/blob/master/WaveJSON.md

## Basic Template

```json
{
  "signal": [
    { "name": "signal_name", "wave": "0.1.0." }
  ]
}
```

## Wave Characters Cheat Sheet

| Character | Meaning | Use Case |
|-----------|---------|----------|
| `p` | Positive clock | Clock signal, rising edge |
| `n` | Negative clock | Clock signal, falling edge |
| `P`, `N` | Clock with arrow | Highlight important edge |
| `0` | Logic low | Control signals, chip selects |
| `1` | Logic high | Active signals, enables |
| `.` | Extend previous | Hold current state |
| `=` | Data value (color 2) | Bus data, variables |
| `2`-`5` | Data value (color variants) | Multiple data values |
| `x` | Undefined | Initial state, don't care |
| `z` | High-Z | Tri-state, disabled output |
| `u`, `d` | Pull-up/down | Resistor-biased signals |
| `\|` | Gap + extend | Timing separation |

## Common Patterns

### Clock Signal
```json
{ "name": "CLK", "wave": "p......." }
```

### Chip Select (Active Low)
```json
{ "name": "CS#", "wave": "1.0...1." }
```

### Data Bus with Labels
```json
{ "name": "DATA[7:0]", "wave": "x.2.3.4.x", "data": ["0x00", "0xFF", "0xA5"] }
```

### Signal Group
```json
["Group Name",
  { "name": "sig1", "wave": "0.1.0." },
  { "name": "sig2", "wave": "1.0.1." }
]
```

### Spacer Between Groups
```json
{ "name": "sig1", "wave": "0.1.0." },
{},  // Empty object = gap
{ "name": "sig2", "wave": "1.0.1." }
```

## Timing Arrows

### Add Nodes to Signals
```json
{ "name": "CLK",  "wave": "0.1.0.", "node": ".a.b." }
{ "name": "DATA", "wave": "x.2.x.", "node": "..c..", "data": ["valid"] }
```

### Draw Arrows Between Nodes
```json
"edge": [
  "a~>c Setup time",
  "b~>c Hold time"
]
```

### Arrow Syntax
- `a~>b` - Curvy arrow from node `a` to `b`
- `a->b` - Straight horizontal arrow
- `a-|>b` - Horizontal then vertical arrow
- `a~|~>b` - Curvy with vertical section

Add label after first space: `"a~>b This is the label"`

## Properties Reference

### Signal Properties

```json
{
  "name": "signal_name",    // Label shown on diagram
  "wave": "0.1.0.",         // Timing pattern
  "data": ["A", "B", "C"],  // Labels for data values (=, 2-5)
  "period": 2,              // Repetition period (cycles)
  "phase": 0.5,             // Phase shift (positive = future)
  "node": "..a.b."          // Markers for arrows
}
```

### Config Options

```json
{
  "signal": [...],
  "config": {
    "hscale": 2    // Horizontal scale (1=compact, 2+=wider)
  }
}
```

### Header/Footer

```json
{
  "signal": [...],
  "head": {
    "text": "Diagram Title"
  },
  "foot": {
    "text": "Footer note"
  }
}
```

## Hardware-Specific Shortcuts

### Write Strobe (74HCT139P)
```json
{ "name": "WR#", "wave": "1.01." }
```

### Latch Enable (74HCT373N)
```json
{ "name": "LE", "wave": "0.10." }
```

### Control Byte (8 bits)
```json
{ "name": "D[7:0]", "wave": "x.3.x", "data": ["CE0|ADDR|RST"] }
```

### Character Data
```json
{ "name": "Char", "wave": "x.=.x", "data": ["'A'"] }
```

### Display Select (CE0, Active Low)
```json
["CE0 (Active Low)",
  { "name": "CE0[3]", "wave": "10.1" },
  { "name": "CE0[2]", "wave": "11.." },
  { "name": "CE0[1]", "wave": "11.." },
  { "name": "CE0[0]", "wave": "11.." }
]
```

## Validation Checklist

Before outputting WaveJSON:

- [ ] Valid JSON syntax (double quotes, no trailing commas)
- [ ] All `wave` strings represent intended timing
- [ ] `data` array length matches number of data symbols (`=`, `2`-`5`) in `wave`
- [ ] `node` string length matches `wave` string length
- [ ] All edge references (`a`, `b`, etc.) exist in node markers
- [ ] `hscale` set appropriately (2+ for complex timing)
- [ ] Signal groups properly formatted with label string first
- [ ] Empty objects `{}` used for spacing, not `null`

## Testing Your WaveJSON

1. Copy JSON to clipboard
2. Visit https://wavedrom.com/editor.html
3. Paste into left editor pane
4. Check rendered diagram on right
5. Fix any syntax errors shown in browser console

## Common Errors

| Error | Cause | Fix |
|-------|-------|-----|
| "Unexpected token" | Invalid JSON | Check quotes, commas, brackets |
| Signals don't align | Different wave lengths | Pad with `.` to match length |
| Arrows missing | Node not found | Verify node markers in `node` property |
| Data labels missing | No `data` array | Add `"data": ["label1", "label2"]` |
| Diagram too narrow | Default hscale=1 | Increase: `"hscale": 2` or higher |

## Color Coding (Data Values)

WaveDrom assigns colors to data symbols:
- `=` - Default color (typically blue/teal)
- `2` - Color variant 1 (green)
- `3` - Color variant 2 (yellow)
- `4` - Color variant 3 (orange)
- `5` - Color variant 4 (red)

Use different colors to distinguish:
- Control vs. data phases
- Different data sources
- Valid vs. invalid data
- Address vs. data buses

## Example: Minimal Complete Diagram

```json
{
  "signal": [
    { "name": "CLK",  "wave": "p....." },
    { "name": "DATA", "wave": "x2.3.x", "data": ["A", "B"] },
    { "name": "EN",   "wave": "01...0" }
  ],
  "config": { "hscale": 2 }
}
```

## Pro Tips

1. **Start simple**: Get basic signals working before adding arrows/groups
2. **Use hscale liberally**: `hscale: 2` or `3` makes complex timing readable
3. **Group related signals**: Makes diagrams easier to understand
4. **Label data meaningfully**: Use hex `0x42`, ASCII `'A'`, or descriptive names
5. **Add spacing**: Empty objects `{}` create visual separation
6. **Plan nodes first**: Sketch which events need arrows before adding markers
7. **Validate incrementally**: Test JSON after each major addition
8. **Match code timing**: Reflect actual delays/sequences from hardware docs
9. **Keep wave lengths consistent**: Signals in same time domain should match length
10. **Document assumptions**: Use head/foot text to note timing values

## Integration with This Project

Common scenarios for WaveJSON generation:

1. **Debugging timing issues**: Visualize what code is supposed to do
2. **Documenting functions**: Add timing diagrams to code comments
3. **Hardware validation**: Compare expected vs. measured waveforms
4. **Planning new features**: Sketch timing before implementing
5. **Bug reports**: Illustrate timing problems clearly
6. **Datasheet compliance**: Verify code meets IC timing specs
7. **Code reviews**: Show control flow visually
8. **Education**: Explain how hardware interface works

Ask Claude: "Show me the WaveJSON for [function/sequence]" or "Generate a timing diagram for [scenario]".
