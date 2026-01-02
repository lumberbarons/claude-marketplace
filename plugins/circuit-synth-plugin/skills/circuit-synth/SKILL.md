---
name: circuit-synth
description: Create and design PCB circuits using Python and KiCad with AI assistance. Generate circuits from natural language descriptions, existing documentation, or templates. Search components, manage BOMs, and produce manufacturing-ready outputs.
allowed-tools: Read, Write, Bash, Glob, Grep
---

# Circuit-Synth: AI-Assisted PCB Design

Professional circuit design using Python + KiCad + AI. Define circuits in code, leverage pre-made patterns, and generate manufacturing-ready outputs.

**Tool Reference:** This skill uses [circuit-synth](https://github.com/circuit-synth/circuit-synth), a Python library for programmatic PCB design with KiCad integration.

**Additional Resources:**
- `reference.md` - Quick reference for commands, patterns, and workflows
- [Official Documentation](https://circuit-synth.readthedocs.io)

## What is Circuit-Synth?

Circuit-synth brings software engineering practices to hardware design:
- Define circuits in Python instead of GUI clicking
- Hierarchical design with modular subcircuits
- Version control friendly (text-based definitions)
- AI-accelerated design workflows
- Professional KiCad output (.kicad_pro, .kicad_sch, .kicad_pcb)
- Manufacturing exports (BOM, Gerbers, PDF schematics)

## Quick Start (60 seconds)

```python
# 1. Install
uv add circuit-synth reportlab

# 2. Create circuit (circuits/my_board.py)
from circuit_synth import circuit, Component, Net

@circuit(name="MyBoard")
def my_circuit():
    vcc = Net('VCC')
    gnd = Net('GND')

    led = Component(
        symbol="Device:LED",
        ref="D",
        value="Red",
        footprint="LED_SMD:LED_0805_2012Metric"
    )

    res = Component(
        symbol="Device:R",
        ref="R",
        value="330R",
        footprint="Resistor_SMD:R_0805_2012Metric"
    )

    # Connect: VCC -> Resistor -> LED -> GND
    res["1"] += vcc
    res["2"] += led["A"]
    led["K"] += gnd

if __name__ == "__main__":
    import os
    os.makedirs("circuits", exist_ok=True)
    my_circuit()

# 3. Generate
# uv run python circuits/my_board.py

# 4. Open in KiCad
# kicad circuits/MyBoard/MyBoard.kicad_pro
```

**That's it!** For advanced usage, see sections below or check `reference.md`.

## When to Use This Skill

Use circuit-synth when:
- Creating new PCB designs from scratch
- Generating circuits from natural language descriptions
- Converting hardware documentation/pinouts into formal schematics
- Reverse-engineering existing hardware into KiCad format
- Searching for components with real-time stock/pricing
- Designing with pre-made manufacturing-ready patterns
- Producing manufacturing outputs (BOM, Gerbers, PDF)

## When NOT to Use This Skill

Skip circuit-synth when:
- User just wants schematic symbols/footprints without circuit design
- Working with non-KiCad tools (Altium, Eagle, etc.)
- Project requires Python < 3.12
- Simple one-off circuit not worth the setup

## Installation and Setup

### Quick Start Installation

**Check if already installed:**
```bash
pip list | grep circuit-synth
```

**If not installed:**

In existing Python project (has `pyproject.toml`):
```bash
uv add circuit-synth
```

In non-Python project or new setup:
```bash
uv init --bare        # Creates pyproject.toml
uv add circuit-synth
```

Fallback with pip:
```bash
pip install circuit-synth
```

**Recommended: Install reportlab for PDF schematic generation:**
```bash
uv add reportlab
# or
pip install reportlab
```

Without reportlab, you'll see warnings and won't be able to generate PDF schematics.

That's it! No need to ask permission - just provide these commands when needed.

### Python Version Requirement

Circuit-synth requires **Python 3.12 or higher**.

Check Python version:
```bash
python --version
# or
python3 --version
```

If Python < 3.12, inform user and ask if they want to proceed with installation anyway or upgrade Python first.

### Common Installation Issues

| Error | Fix |
|-------|-----|
| `No module named 'circuit_synth'` | Run `uv add circuit-synth` or `pip install circuit-synth` |
| `No pyproject.toml found` | Run `uv init --bare` first, then `uv add circuit-synth` |
| `Python version too old` | Requires Python 3.12+, upgrade with pyenv or conda |

## Common Pin Naming Issues

**Most common source of errors!** When you get `Pin 'XXX' not found` errors, the error message shows available pins. Use those names.

### Quick Reference Table

| IC Family | Common Wrong Names | Correct Names | Example |
|-----------|-------------------|---------------|---------|
| 74xx Latches (373, 573) | 1D, 1Q, 2D, 2Q | D0-D7, O0-O7 | D0 for data in, O0 for output |
| 74xx Buffers (245) | DIR, OE | A->B, CE | Direction control renamed |
| 74xx Decoders (138, 139) | 1G, 1A0, 1Y0 | E, A0, O0 | Single decoder uses generic names |
| Polarized Caps | +, - | 1, 2 | Pin 1 is positive, pin 2 negative |
| Generic symbols | Named pins | 1, 2, 3... | Use pin numbers for placeholders |

**How to fix:** When you see the error, it lists available pins - use those exact names in your code.

```python
# Error says: Available: 'D0', 'D1', ..., 'O0', 'O1', ...
# Wrong:
latch["1D"] += data_in  # ❌

# Correct:
latch["D0"] += data_in  # ✓
```

## Incremental Development (IMPORTANT!)

**Don't write the whole circuit at once!** Build and test incrementally to catch pin name errors early.

### Step 1: Minimal Test (1-2 components)

Start with just power and one IC:

```python
from circuit_synth import circuit, Component, Net

@circuit(name="Test")
def test_circuit():
    vcc = Net('VCC_5V')
    gnd = Net('GND')

    # Test one IC first
    ic = Component(
        symbol="74xx:74HC373",  # Or whatever you need
        ref="U1",
        value="Test",
        footprint="Package_DIP:DIP-20_W7.62mm"
    )

    ic["VCC"] += vcc
    ic["GND"] += gnd
    ic["D0"] += Net('DATA0')  # Try connecting one pin
    ic["O0"] += Net('OUT0')

if __name__ == "__main__":
    c = test_circuit()
    c.generate_kicad_project(project_name="Test")
```

Run: `uv run python circuits/test.py`

### Step 2: Fix Pin Names from Error Messages

If it fails, read the error:
```
ComponentError: Pin 'D0' not found in U1 (74xx:74HC373).
Available: 'D0', 'D1', 'D2', ...
```

Use the available names shown in the error.

### Step 3: Add More Components Gradually

Once one IC works, add 2-3 more at a time, testing after each addition.

## Where Are My Files?

After running `circuit_obj.generate_kicad_project(project_name="MyBoard")`:

```
your-project/
└── circuits/                ← **CIRCUIT FOLDER**
    ├── your_circuit.py      ← Your Python circuit definition
    └── MyBoard/             ← **GENERATED HERE**
        ├── MyBoard.kicad_pro  ← Open this in KiCad
        ├── MyBoard.kicad_sch
        ├── MyBoard.json
        └── MyBoard.net
```

**Open it:** `kicad circuits/MyBoard/MyBoard.kicad_pro`

The `circuits/` folder keeps all circuit definitions and generated KiCad projects organized together.

## Common Errors & Quick Fixes

| Error | Cause | Fix |
|-------|-------|-----|
| `Pin 'X' not found` | Wrong pin name | Use exact names from error message |
| `Symbol 'XXX' not found` | Symbol doesn't exist in KiCad libraries | Try similar symbol (74HC139→74LS139) or use generic placeholder |
| `missing 1 required positional argument: 'project_name'` | Forgot parameter | Add `project_name="YourBoard"` |
| `No module named 'circuit_synth'` | Not installed | Run `uv add circuit-synth` |
| `No pyproject.toml found` | Not in uv project | Run `uv init --bare` first |

## Placeholder-First Approach for Custom/Unknown ICs

**Fast track for reverse-engineering or using ICs without KiCad symbols:**

### 1. Use Generic Symbols as Placeholders

Don't waste time searching for exact symbols. Start with placeholders:

```python
# Unknown or custom IC? Use Device:C (capacitor) as 2-pin placeholder
custom_ic = Component(
    symbol="Device:C",  # Quick 2-pin placeholder
    ref="U1",
    value="ActualPartNumber",  # Document the real part
    footprint="Package_DIP:DIP-20_W7.62mm"  # Use actual footprint
)

# Connect using pin numbers
custom_ic["1"] += vcc
custom_ic["2"] += gnd

# Document actual connections in comments for later
# Pin 1: VCC, Pin 2: GND, Pin 3: DATA0, ... etc
```

### 2. Generate Schematic Quickly

- Get project structure in place
- See component layout
- Verify netlists work
- Don't block on symbol creation

### 3. Create Custom Symbols Later in KiCad

Once schematic is generated:
- Open KiCad Symbol Editor
- Create proper symbol with all pins
- Replace placeholder in schematic
- Update connections

**When to use this:** Reverse-engineering projects, obsolete ICs, custom hardware, any time symbol search takes >5 minutes.

## Project Creation Workflows

### Workflow 1: Add to Existing Project

**Use when**: Adding circuit design to existing firmware/hardware project

**Steps:**
1. Install circuit-synth in project: `uv add circuit-synth` or `pip install circuit-synth`
2. Optionally install reportlab: `uv add reportlab` or `pip install reportlab`
3. Create directory: `mkdir -p circuits`
4. Create initial circuit file: `circuits/main.py`
5. Add to .gitignore (if desired):
   ```
   circuits/*/*.kicad_pro
   circuits/*/*.kicad_sch
   circuits/*/*.kicad_pcb
   circuits/*/*.json
   circuits/*/*.net
   *.kicad_prl
   *.kicad_sch-bak
   ```
6. Document in README: "Circuit schematics in `circuits/` - Python definitions and generated KiCad projects"

### Workflow 2: Quick Prototype

**Use when**: Testing a circuit idea without full project structure

**Single file approach:**
```python
# circuits/quick_test.py
import os
from circuit_synth import circuit, Component, Net

@circuit(name="QuickTest")
def my_circuit():
    # Define circuit here
    pass

if __name__ == "__main__":
    os.makedirs("circuits", exist_ok=True)
    my_circuit()
```

Run: `python circuits/quick_test.py`

### Project Type Decision Table

| User Intent | Workflow | Command |
|-------------|----------|---------|
| Add to existing firmware project | Workflow 1 | Manual setup + `mkdir circuit-synth` |
| Quick test / proof of concept | Workflow 2 | Single Python file |

## Circuit Generation from Natural Language

**Quick workflow:**
1. Parse requirements (components, specs, constraints)
2. Check pattern library, use if match found
3. Search components with jlc-fast
4. Generate Python circuit definition
5. Validate and run

**Component syntax:**
```python
led = Component(symbol="Device:LED", ref="D", value="Red",
                footprint="LED_SMD:LED_0805_2012Metric")
led["K"] += gnd  # Connect pins to nets
```

**Buses:**
```python
data_bus = [Net(f'D{i}') for i in range(8)]  # D0-D7
for i in range(8):
    buffer[f'A{i}'] += data_bus[i]
```

See `reference.md` for detailed examples (LDO regulator, LED circuits, hierarchical subcircuits).

## From Documentation/Code to Circuit

Use when you have firmware code, hardware docs, pinout tables, or datasheets.

**Quick workflow:**
1. Glob/Read documentation files
2. Extract components and connections
3. Generate Python circuit definition (use placeholders for unknown ICs)
4. Run and validate

See `reference.md` for complete step-by-step workflow with examples.

```python
from circuit_synth import circuit, Component, Net

@circuit(name="MyBoard")
def my_board():
    """Based on PINOUTS.md and code analysis."""

    # Define nets from documentation
    vcc, gnd = Net('VCC'), Net('GND')
    data_bus = [Net(f'D{i}') for i in range(8)]  # D0-D7

    # Components from doc (use placeholders for unknown symbols)
    ic1 = Component(
        symbol="Device:C",  # Placeholder if symbol unknown
        ref="U1",
        value="ActualPartNumber",
        footprint="Package_DIP:DIP-20_W7.62mm"
    )

    # Power connections
    ic1["1"] += vcc  # Using pin numbers for placeholder
    ic1["2"] += gnd

    # Add more components incrementally...
    # connector = Component(...)
    # other_ic = Component(...)

if __name__ == "__main__":
    c = my_board()
    c.generate_kicad_project(project_name="MyBoard")
```

**Key points:**
- Use placeholder symbols (`Device:C`) for unknown/custom ICs
- Document actual part numbers in `value` field
- Add connections incrementally, test often
- Replace placeholders with proper symbols later in KiCad

**Step 5: Generate and Test**

```bash
uv run python circuits/my_board.py
```

Check generated files in `MyBoard/` directory, open in KiCad.

**Step 6: Cross-Reference**

Verify schematic against original documentation:
- Check all component connections match docs
- Verify pin numbers/names are correct
- Compare with firmware code if available

### Tips for Code-Based Projects

If working from PlatformIO or Arduino code:

```bash
# Find pin definitions
grep -r "define.*PIN" src/ include/
grep -r "const.*pin" src/ include/
grep -r "GPIO" src/ include/
```

Use these to map hardware connections in your circuit.
