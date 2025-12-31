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

### Optional API Setup

After installation, you can optionally set up API integrations for enhanced component search:

```bash
cs-setup-snapeda-api   # Component symbol/footprint search
cs-setup-digikey-api   # Component availability and pricing
```

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

### Workflow 1: New Project from Template

**Use when**: Starting a completely new circuit design

**Command:**
```bash
uv run cs-new-project my_project
```

**What it creates:**
```
my_project/
├── circuits/                   # Circuit definitions and outputs
│   ├── main.py                 # Primary circuit (ESP32-C6 example)
│   ├── [other_circuits].py     # Additional circuit definitions
│   └── [ProjectName]/          # Generated KiCad projects
├── .claude/                    # AI integration (optional)
│   ├── agents/                 # Specialized agents
│   └── commands/               # Slash commands
├── README.md                   # Project documentation
├── CLAUDE.md                   # AI-specific guidance
└── [generated KiCad files]     # .kicad_pro, .kicad_sch, .kicad_pcb
```

**Generated example**: ESP32-C6 minimal board with:
- USB-C connector
- 3.3V LDO regulator
- Power LED
- Programming interface

**Steps:**
1. Run `cs-new-project project_name`
2. Navigate: `cd project_name`
3. Review generated Python: `cat circuits/main.py`
4. Generate KiCad files: `uv run python circuits/main.py`
5. Open in KiCad: `kicad project_name.kicad_pro`

### Workflow 2: Specific Template

**Use when**: You know exactly what type of circuit you need

**Command with templates:**
```bash
# Quick start (no prompts)
uv run cs-new-project my_board --quick

# Select specific templates
uv run cs-new-project my_board --circuits STM32,USB

# Skip AI integration
uv run cs-new-project my_board --no-agents

# Developer mode (extra tools)
uv run cs-new-project my_board --developer
```

**Available templates:**
- `ESP32` - ESP32-C6 minimal setup
- `STM32` - STM32 minimal configuration
- `USB` - USB-C connectivity
- `POWER` - Power supply circuits

### Workflow 3: Add to Existing Project

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

### Workflow 4: Quick Prototype

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
| New standalone circuit project | Workflow 1 | `cs-new-project name` |
| Known template needed | Workflow 2 | `cs-new-project name --circuits TYPE` |
| Add to existing firmware project | Workflow 3 | Manual setup + `mkdir circuit-synth` |
| Quick test / proof of concept | Workflow 4 | Single Python file |

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

If working from PlatformIO/Arduino code:

```bash
# Find pin definitions
grep -r "define.*PIN" src/ include/
grep -r "const.*pin" src/ include/
grep -r "GPIO" src/ include/
```

Use these to map hardware connections in your circuit.

## Component Search and Selection

### Three Search Methods

**1. jlc-fast (Fastest, Zero AI Tokens)**

Best for: JLCPCB-specific parts, real-time stock, lowest cost

```bash
# Search by description
uv run jlc-fast search "STM32F103C8T6"
uv run jlc-fast search "USB Type-C connector"
uv run jlc-fast search "10uF 0805 capacitor"

# Find cheapest with stock
uv run jlc-fast cheapest "10k resistor" --min-stock 10000

# Search with filters
uv run jlc-fast search "LDO 3.3V" --min-stock 1000
```

Advantages:
- Instant results
- Real-time stock and pricing
- No AI token usage
- JLCPCB part numbers (C12345 format)

**2. /find-symbol (KiCad Symbol Search)**

Best for: Finding exact KiCad symbol library references

```bash
/find-symbol STM32F4
/find-symbol "USB connector"
/find-symbol "voltage regulator"
```

Returns: KiCad symbol library paths for use in Component definitions

**3. /find-parts (Multi-Source Availability)**

Best for: Cross-vendor availability, DigiKey/Mouser comparison

```bash
/find-parts STM32F103C8T6
/find-parts "10uF capacitor"
```

Returns: Availability, pricing, datasheets across multiple suppliers

### Selection Workflow

```
1. Identify requirement
   └─ Component type (MCU, regulator, passive)

2. Search options
   ├─ Exact part known → jlc-fast search "PART_NUMBER"
   ├─ Need JLCPCB part → jlc-fast search "description"
   ├─ Need KiCad symbol → /find-symbol
   └─ Multi-vendor → /find-parts

3. Evaluate results
   ├─ Stock availability (>1000 preferred)
   ├─ Price
   ├─ Package/footprint
   └─ Datasheet review

4. Find symbol and footprint
   ├─ /find-symbol if not known
   └─ Check KiCad libraries

5. Add to circuit
   └─ Component() with full properties
```

### Component Properties

**Minimum required:**
```python
Component(
    symbol="Device:R",              # REQUIRED
    ref="R",                         # REQUIRED
    footprint="Resistor_SMD:R_0805_2012Metric"  # REQUIRED
)
```

**Recommended for BOM:**
```python
Component(
    symbol="Device:R",
    ref="R",
    value="10k",                     # Add value
    footprint="Resistor_SMD:R_0805_2012Metric",
    datasheet="https://..."          # Add datasheet URL
)
```

**Full BOM with sourcing:**
```python
Component(
    symbol="Device:R",
    ref="R",
    value="10k",
    footprint="Resistor_SMD:R_0805_2012Metric",
    datasheet="https://example.com/datasheet.pdf",
    manufacturer="Yageo",
    mpn="RC0805FR-0710KL",           # Manufacturer part number
    supplier="JLCPCB",
    supplier_pn="C17414",            # JLCPCB part number
    cost="0.001"
)
```

### Stock Availability Guidelines

| Stock Level | Assessment | Action |
|-------------|------------|--------|
| > 10,000 | Very safe | Preferred choice |
| 1,000 - 10,000 | Safe | Good for production |
| 100 - 1,000 | Acceptable | Consider for prototypes |
| < 100 | Risky | Find alternative |
| 0 | Out of stock | Must find substitute |

### Common Component Searches

**Power supplies:**
```bash
jlc-fast search "LDO regulator 3.3V"
jlc-fast search "buck converter 5V"
jlc-fast search "boost converter"
```

**Microcontrollers:**
```bash
jlc-fast search "STM32F103C8T6"
jlc-fast search "ESP32-S3"
jlc-fast search "ATmega328P"
```

**Connectors:**
```bash
jlc-fast search "USB Type-C connector"
jlc-fast search "JST connector 2 pin"
jlc-fast search "pin header 2.54mm"
```

**Passives:**
```bash
jlc-fast search "10uF 0805 capacitor"
jlc-fast search "100nF 0603 ceramic"
jlc-fast cheapest "10k resistor 0805" --min-stock 10000
```

## Circuit Patterns Library

Seven pre-built, manufacturing-ready circuit patterns:

1. **buck_converter** - Step-down DC-DC converter
2. **boost_converter** - Step-up DC-DC converter
3. **lipo_charger** - Lithium battery charging
4. **resistor_divider** - Voltage divider
5. **thermistor** - Temperature sensor with ADC
6. **opamp_follower** - Unity-gain buffer
7. **rs485** - RS-485 communication interface

**Quick example:**
```python
from buck_converter import buck_converter

buck_converter(vin_12v, vout_5v, gnd,
               output_voltage="5V", max_current="3A")
```

See `reference.md` for detailed parameters, examples, and customization strategies for each pattern.

## Manufacturing Outputs

### Four Output Types

**1. Bill of Materials (BOM)**

Generate CSV with all components:

```python
import os
os.makedirs("circuits", exist_ok=True)
circuit.generate_bom(project_name="circuits/my_board")
```

Output: `circuits/my_board.csv`

**CSV format:**
```
Reference,Value,Footprint,Quantity,Manufacturer,MPN,Supplier,Supplier PN,Cost
R1,10k,Resistor_SMD:R_0805_2012Metric,1,Yageo,RC0805FR-0710KL,JLCPCB,C17414,0.001
C1,10uF,Capacitor_SMD:C_0805_2012Metric,1,Samsung,CL21A106KAYNNNE,JLCPCB,C15850,0.005
U1,STM32F103C8T6,Package_QFP:LQFP-48_7x7mm_P0.5mm,1,STMicro,STM32F103C8T6,JLCPCB,C8734,2.50
```

**2. Gerber Files**

Generate manufacturing files for PCB fabrication:

```python
import os
os.makedirs("circuits", exist_ok=True)
circuit.generate_gerbers(project_name="circuits/my_board")
```

Output directory: `circuits/my_board/gerbers/`

**Files generated:**
- `my_board-F.Cu.gbr` - Front copper layer
- `my_board-B.Cu.gbr` - Back copper layer
- `my_board-F.Mask.gbr` - Front solder mask
- `my_board-B.Mask.gbr` - Back solder mask
- `my_board-F.Silkscreen.gbr` - Front silkscreen
- `my_board-Edge.Cuts.gbr` - Board outline
- `my_board-PTH.xln` - Plated through-hole drill file
- `my_board-NPTH.xln` - Non-plated drill file

**3. PDF Schematic**

Generate human-readable schematic (requires reportlab):

```python
import os
os.makedirs("circuits", exist_ok=True)
circuit.generate_pdf_schematic(project_name="circuits/my_board")
```

Output: `circuits/my_board.pdf`

Use for: Documentation, reviews, manufacturing reference

**Note:** Install reportlab first: `uv add reportlab` or `pip install reportlab`

**4. JSON Netlist**

Generate machine-readable netlist:

```python
import os
os.makedirs("circuits", exist_ok=True)
circuit.generate_json_netlist(project_name="circuits/my_board")
```

Output: `circuits/my_board.net.json`

Use for: Circuit analysis, custom tooling, verification

### BOM Optimization for JLCPCB

To optimize for JLCPCB assembly, include JLCPCB part numbers:

```python
resistor = Component(
    symbol="Device:R",
    ref="R",
    value="10k",
    footprint="Resistor_SMD:R_0805_2012Metric",
    supplier="JLCPCB",
    supplier_pn="C17414"  # JLCPCB part number
)
```

Find part numbers:
```bash
uv run jlc-fast search "10k resistor 0805"
# Returns: C17414 (JLCPCB part number)
```

### Manufacturing Checklist

Before sending to fabrication:

- [ ] BOM generated and reviewed
  - [ ] All components have supplier part numbers
  - [ ] Stock verified (all parts >100 in stock)
  - [ ] Costs calculated and within budget
- [ ] Gerbers generated without errors
  - [ ] All layers present (copper, mask, silkscreen, edge cuts)
  - [ ] Drill files generated (PTH and NPTH)
  - [ ] Preview in Gerber viewer (gerbv, KiCad)
- [ ] PDF schematic reviewed
  - [ ] All connections correct
  - [ ] No floating nets or unconnected pins
  - [ ] Reference designators sequential
- [ ] KiCad DRC passed
  - [ ] No electrical rule violations
  - [ ] No overlapping components
  - [ ] Proper clearances
- [ ] Part numbers verified on JLCPCB
  - [ ] Search each part on JLCPCB website
  - [ ] Confirm availability and price match

### Complete Manufacturing Workflow

Generate all manufacturing outputs with:

```python
circuit.generate_bom(project_name="circuits/my_board")
circuit.generate_gerbers(project_name="circuits/my_board")
circuit.generate_pdf_schematic(project_name="circuits/my_board")
```

See `reference.md` for complete workflow example with error checking and status output.

## Instructions for Claude

### Initial Actions

When circuit-synth skill is invoked:

1. **Check installation**
   - Run: `pip list | grep circuit-synth`
   - If not installed → provide install commands (see "Quick Start Installation")

2. **Understand user intent**
   - New project? → Project creation workflow
   - Generate circuit? → Natural language or documentation workflow
   - Search components? → Component search workflow
   - Manufacturing outputs? → Export workflow

3. **Start incrementally**
   - Don't generate entire circuit at once
   - Start with 1-2 components, test, then add more
   - Catch pin naming errors early

### Circuit Generation Process

**From natural language:**
1. Parse requirements (components, specs, constraints)
2. Check pattern library → use if match, otherwise custom
3. Search components with jlc-fast
4. Generate Python code incrementally (start with 1-2 components)
5. Validate and run
6. Review output and summarize

See `reference.md` for detailed workflow steps.

### Documentation-to-Circuit Workflow

**From hardware docs:**
1. Read documentation (Glob for pinout files, datasheets, wiring guides)
2. Extract components and connections from tables
3. Create Component() objects incrementally (use placeholders if needed)
4. Define Net() objects and connect pins
5. Generate, validate, cross-reference against docs

See `reference.md` for detailed step-by-step workflow.

### Error Handling

**Most common: Pin naming errors**
- Error message shows available pins - use those exact names
- See "Common Pin Naming Issues" section for quick reference
- When in doubt, use pin numbers ("1", "2") for placeholders

**For all other errors**, see "Common Errors & Quick Fixes" table in the main skill doc.

### Decision Trees (Summary)

**Create circuit:** Check if request matches a pattern (buck, boost, etc.), use pattern if yes, otherwise custom circuit
**From documentation:** Read docs/code, parse connections, generate circuit incrementally
**Find components:** Use jlc-fast for searches, /find-symbol for KiCad symbols

### Best Practices

1. **ALWAYS develop incrementally** - Start with 1-2 components, test, add more
2. **Use placeholder-first** - Use `Device:C` for unknown ICs, replace later in KiCad
3. **Check pin names in error messages** - Use exact names shown in errors
4. **Use descriptive net names** - `vcc_3v3` not `net1`
5. **Document assumptions in comments**
6. **Use hierarchical design** for >5 components

## Output Format

When using this skill, provide:

1. **Clear summary** of actions taken
2. **Files created** with paths
3. **Next steps** for user (open KiCad, review BOM, etc.)
4. **Any warnings** (custom symbols needed, out-of-stock parts, etc.)

Example output:
```
Created circuit-synth KiCad project:

Files generated:
  - circuits/my_board.py (circuit definition)
  - MyBoard/MyBoard.kicad_pro
  - MyBoard/MyBoard.kicad_sch
  - MyBoard/MyBoard.json (netlist)

Circuit includes:
  - 3 ICs from documentation
  - Power connector
  - Bypass capacitors

Next steps:
  1. Open: kicad MyBoard/MyBoard.kicad_pro
  2. Review schematic connections
  3. Replace placeholder symbols with proper KiCad symbols (if needed)
  4. Design PCB layout

Warnings:
  - Some ICs using generic placeholders (Device:C) - create custom symbols in KiCad
  - Verify all connections match original documentation
```
