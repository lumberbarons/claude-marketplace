# Circuit-Synth Quick Reference

Official docs: https://circuit-synth.readthedocs.io
GitHub: https://github.com/circuit-synth/circuit-synth

## Installation

```bash
# Check if installed
pip list | grep circuit-synth

# Install with uv (preferred)
uv add circuit-synth

# Install with pip
pip install circuit-synth

# Check version
python -c "import circuit_synth; print(circuit_synth.__version__)"
```

**Requirements**: Python 3.12+

## Project Creation

```bash
# New project with prompts
uv run cs-new-project my_project

# Quick start (no prompts)
uv run cs-new-project my_project --quick

# Specific templates
uv run cs-new-project my_board --circuits STM32,USB,ESP32

# Skip AI integration
uv run cs-new-project my_board --no-agents

# Navigate and generate
cd my_project
uv run python circuit-synth/main.py
```

**Templates available**: ESP32, STM32, USB, POWER

## Basic Circuit Template

```python
from circuit_synth import circuit, Component, Net

@circuit(name="MyCircuit")
def my_circuit():
    """Circuit description"""

    # Define nets
    vcc = Net('VCC')
    gnd = Net('GND')
    signal = Net('SIGNAL')

    # Create component
    resistor = Component(
        symbol="Device:R",
        ref="R",
        value="10k",
        footprint="Resistor_SMD:R_0805_2012Metric"
    )

    # Connect pins to nets
    resistor[1] += vcc
    resistor[2] += signal

if __name__ == "__main__":
    my_circuit()
```

## Component Definition

**Minimum required:**
```python
Component(
    symbol="Device:R",
    ref="R",
    footprint="Resistor_SMD:R_0805_2012Metric"
)
```

**With value:**
```python
Component(
    symbol="Device:R",
    ref="R",
    value="10k",
    footprint="Resistor_SMD:R_0805_2012Metric"
)
```

**Full BOM data:**
```python
Component(
    symbol="Device:R",
    ref="R",
    value="10k",
    footprint="Resistor_SMD:R_0805_2012Metric",
    datasheet="https://example.com/datasheet.pdf",
    manufacturer="Yageo",
    mpn="RC0805FR-0710KL",
    supplier="JLCPCB",
    supplier_pn="C17414",
    cost="0.001"
)
```

## Common Components

**Resistor:**
```python
Component(
    symbol="Device:R",
    ref="R",
    value="10k",
    footprint="Resistor_SMD:R_0805_2012Metric"
)
```

**Capacitor:**
```python
Component(
    symbol="Device:C",
    ref="C",
    value="10uF",
    footprint="Capacitor_SMD:C_0805_2012Metric"
)
```

**LED:**
```python
Component(
    symbol="Device:LED",
    ref="D",
    value="Red",
    footprint="LED_SMD:LED_0805_2012Metric"
)
```

**Voltage Regulator (LDO):**
```python
Component(
    symbol="Regulator_Linear:AMS1117-3.3",
    ref="U",
    value="AMS1117-3.3",
    footprint="Package_TO_SOT_SMD:SOT-223-3_TabPin2"
)
```

**USB Type-C Connector:**
```python
Component(
    symbol="Connector:USB_C_Receptacle_USB2.0",
    ref="J",
    value="USB-C",
    footprint="Connector_USB:USB_C_Receptacle_HRO_TYPE-C-31-M-12"
)
```

**Microcontroller (STM32):**
```python
Component(
    symbol="MCU_ST_STM32F1:STM32F103C8Tx",
    ref="U",
    value="STM32F103C8T6",
    footprint="Package_QFP:LQFP-48_7x7mm_P0.5mm"
)
```

**Microcontroller (ESP32):**
```python
Component(
    symbol="RF_Module:ESP32-WROOM-32",
    ref="U",
    value="ESP32-WROOM-32",
    footprint="RF_Module:ESP32-WROOM-32"
)
```

## Net Connections

**Connect pin to net:**
```python
led["K"] += gnd           # LED cathode to ground
led["A"] += resistor[1]   # LED anode to resistor pin 1
```

**Create nets:**
```python
# Individual nets
vcc = Net('VCC')
gnd = Net('GND')

# Multiple nets (bus)
data = [Net(f'D{i}') for i in range(8)]  # D0-D7
addr = [Net(f'A{i}') for i in range(16)]  # A0-A15
```

**Connect to bus:**
```python
for i in range(8):
    mcu[f'PB{i}'] += data[i]  # Connect MCU port B to data bus
```

## Circuit Patterns

**Import patterns:**
```python
from buck_converter import buck_converter
from boost_converter import boost_converter
from lipo_charger import lipo_charger
from resistor_divider import resistor_divider
from thermistor import thermistor
from opamp_follower import opamp_follower
from rs485 import rs485
```

**Use buck converter:**
```python
@circuit(name="Power")
def power_supply():
    vin = Net('VIN_12V')
    vout = Net('VOUT_5V')
    gnd = Net('GND')

    buck_converter(vin, vout, gnd,
                   output_voltage="5V",
                   max_current="3A")
```

**Use boost converter:**
```python
boost_converter(vin, vout, gnd,
                output_voltage="12V",
                max_current="1A")
```

**Use LiPo charger:**
```python
lipo_charger(vin, vbat, gnd,
             charge_current="500mA")
```

**Use resistor divider:**
```python
resistor_divider(vin, vout, gnd,
                 ratio=0.5)  # 50% division
```

**Use thermistor:**
```python
thermistor(vcc, adc_out, gnd,
           thermistor_type="NTC 10k")
```

## Hierarchical Design

**Define subcircuit:**
```python
@circuit(name="MCU_Core")
def mcu_core(vcc, gnd, tx, rx):
    """Microcontroller subcircuit"""
    mcu = Component(
        symbol="MCU_ST_STM32F1:STM32F103C8Tx",
        ref="U",
        value="STM32F103C8T6",
        footprint="Package_QFP:LQFP-48_7x7mm_P0.5mm"
    )

    mcu["VDD"] += vcc
    mcu["VSS"] += gnd
    mcu["PA9"] += tx
    mcu["PA10"] += rx
```

**Use in main circuit:**
```python
@circuit(name="MainBoard")
def main_board():
    vcc = Net('VCC_3V3')
    gnd = Net('GND')
    uart_tx = Net('UART_TX')
    uart_rx = Net('UART_RX')

    # Use subcircuit
    mcu_core(vcc, gnd, uart_tx, uart_rx)
```

## Manufacturing Outputs

**Generate BOM:**
```python
from circuit_synth import generate_bom

generate_bom(project_name="my_board")
# Output: my_board.csv
```

**Generate Gerbers:**
```python
from circuit_synth import generate_gerbers

generate_gerbers(project_name="my_board")
# Output: my_board/gerbers/ directory
```

**Generate PDF Schematic:**
```python
from circuit_synth import generate_pdf_schematic

generate_pdf_schematic(project_name="my_board")
# Output: my_board.pdf
```

**Generate JSON Netlist:**
```python
from circuit_synth import generate_json_netlist

generate_json_netlist(project_name="my_board")
# Output: my_board.net.json
```

**Complete manufacturing script:**
```python
if __name__ == "__main__":
    my_circuit()

    # Generate all outputs
    project = "MyBoard"

    generate_bom(project_name=project)
    generate_gerbers(project_name=project)
    generate_pdf_schematic(project_name=project)

    print(f"Manufacturing files ready in {project}/")
```

## Common Footprints

| Component | Package | Footprint Path |
|-----------|---------|----------------|
| Resistor 0805 | 2012 Metric | `Resistor_SMD:R_0805_2012Metric` |
| Resistor 0603 | 1608 Metric | `Resistor_SMD:R_0603_1608Metric` |
| Resistor 0402 | 1005 Metric | `Resistor_SMD:R_0402_1005Metric` |
| Capacitor 0805 | 2012 Metric | `Capacitor_SMD:C_0805_2012Metric` |
| Capacitor 0603 | 1608 Metric | `Capacitor_SMD:C_0603_1608Metric` |
| Capacitor 0402 | 1005 Metric | `Capacitor_SMD:C_0402_1005Metric` |
| LED 0805 | 2012 Metric | `LED_SMD:LED_0805_2012Metric` |
| LED 0603 | 1608 Metric | `LED_SMD:LED_0603_1608Metric` |
| SOT-23 | SOT-23 | `Package_TO_SOT_SMD:SOT-23` |
| SOT-23-5 | SOT-23-5 | `Package_TO_SOT_SMD:SOT-23-5` |
| SOT-223 | SOT-223 | `Package_TO_SOT_SMD:SOT-223-3_TabPin2` |
| TSSOP-20 | TSSOP-20 | `Package_SO:TSSOP-20_4.4x6.5mm_P0.65mm` |
| QFP-48 | LQFP-48 | `Package_QFP:LQFP-48_7x7mm_P0.5mm` |
| QFP-64 | LQFP-64 | `Package_QFP:LQFP-64_10x10mm_P0.5mm` |
| DIP-8 | DIP-8 | `Package_DIP:DIP-8_W7.62mm` |
| DIP-14 | DIP-14 | `Package_DIP:DIP-14_W7.62mm` |
| DIP-16 | DIP-16 | `Package_DIP:DIP-16_W7.62mm` |
| DIP-20 | DIP-20 | `Package_DIP:DIP-20_W7.62mm` |

## Symbol Libraries

| Component Type | Library Name |
|----------------|--------------|
| Resistor, Capacitor, LED | `Device` |
| Linear regulators | `Regulator_Linear` |
| Switching regulators | `Regulator_Switching` |
| STM32 MCUs | `MCU_ST_STM32F1`, `MCU_ST_STM32F4`, etc. |
| Generic connectors | `Connector_Generic` |
| USB connectors | `Connector` |
| RF modules (ESP32) | `RF_Module` |
| Logic gates | `74xx` |
| Op-amps | `Amplifier_Operational` |
| Transistors | `Device` |
| Diodes | `Device` or `Diode` |

## Workflow Cheat Sheet

### Create New Project

```bash
uv run cs-new-project my_project
cd my_project
uv run python circuit-synth/main.py
kicad my_project.kicad_pro
```

### Generate Circuit from Description

1. Parse requirements (components, specs)
2. Search components: `jlc-fast search "part description"`
3. Write Python circuit definition
4. Run: `python circuit-synth/main.py`
5. Review in KiCad

### Convert Documentation to Circuit

1. Find docs: `glob **/*PINOUT*.md`
2. Read files: `read BOARD_PINOUTS.md`
3. Extract: IC list, connections, nets
4. Write Python with Component() and Net()
5. Map connections from tables
6. Generate and verify

## Troubleshooting

| Error | Cause | Solution |
|-------|-------|----------|
| `ModuleNotFoundError: circuit_synth` | Not installed | `uv add circuit-synth` or `pip install circuit-synth` |
| `Command not found: cs-new-project` | Not in PATH | Use `python -m circuit_synth.cli.new_project` |
| `Symbol not found: XYZ` | Invalid KiCad symbol | Use `/find-symbol` or check library:name format |
| `Pin name 'ABC' not found` | Wrong pin name | Check datasheet, verify symbol pinout |
| `Python version too old` | Python < 3.12 | Upgrade Python to 3.12+ |
| `Net conflict: multiple drivers` | Incorrect connections | Review netlist, check for shorts |
| Missing footprint | Not specified | Add `footprint=` parameter to Component() |

## Common Symbols

| Component | Symbol Path |
|-----------|-------------|
| Resistor | `Device:R` |
| Capacitor | `Device:C` |
| LED | `Device:LED` |
| Transistor NPN | `Device:Q_NPN_BCE` |
| Transistor PNP | `Device:Q_PNP_BCE` |
| Diode | `Device:D` |
| Schottky Diode | `Device:D_Schottky` |
| Zener Diode | `Device:D_Zener` |
| LDO (AMS1117) | `Regulator_Linear:AMS1117-3.3` |
| Buck Converter (LM2596) | `Regulator_Switching:LM2596` |
| Op-Amp (LM358) | `Amplifier_Operational:LM358` |
| USB Type-C | `Connector:USB_C_Receptacle_USB2.0` |
| Pin Header 1x4 | `Connector_Generic:Conn_01x04` |
| UART (generic) | `Interface_UART:MAX232` |

## Example: Complete LED Circuit

```python
from circuit_synth import circuit, Component, Net

@circuit(name="LED_Circuit")
def led_circuit():
    """Simple LED with current limiting resistor"""

    # Define nets
    vcc = Net('VCC_5V')
    gnd = Net('GND')

    # LED
    led = Component(
        symbol="Device:LED",
        ref="D",
        value="Red",
        footprint="LED_SMD:LED_0805_2012Metric",
        supplier="JLCPCB",
        supplier_pn="C2286"  # Example JLCPCB part
    )

    # Current limiting resistor
    # 5V - 2V (LED drop) = 3V
    # 3V / 10mA = 300Ω, use 330Ω
    resistor = Component(
        symbol="Device:R",
        ref="R",
        value="330R",
        footprint="Resistor_SMD:R_0805_2012Metric",
        supplier="JLCPCB",
        supplier_pn="C17630"
    )

    # Connect
    resistor[1] += vcc
    resistor[2] += led["A"]  # Anode
    led["K"] += gnd          # Cathode

if __name__ == "__main__":
    led_circuit()
```

## Example: Power Supply

```python
from circuit_synth import circuit, Component, Net

@circuit(name="Power_Supply_3V3")
def power_supply():
    """3.3V LDO regulator with input/output caps"""

    # Nets
    vin = Net('VIN_5V')
    vout = Net('VOUT_3V3')
    gnd = Net('GND')

    # LDO Regulator
    reg = Component(
        symbol="Regulator_Linear:AMS1117-3.3",
        ref="U",
        value="AMS1117-3.3",
        footprint="Package_TO_SOT_SMD:SOT-223-3_TabPin2",
        supplier="JLCPCB",
        supplier_pn="C6186"
    )

    # Input capacitor (10uF)
    cin = Component(
        symbol="Device:C",
        ref="C",
        value="10uF",
        footprint="Capacitor_SMD:C_0805_2012Metric",
        supplier="JLCPCB",
        supplier_pn="C15850"
    )

    # Output capacitor (22uF)
    cout = Component(
        symbol="Device:C",
        ref="C",
        value="22uF",
        footprint="Capacitor_SMD:C_0805_2012Metric",
        supplier="JLCPCB",
        supplier_pn="C45783"
    )

    # Connect regulator
    reg["VI"] += vin
    reg["VO"] += vout
    reg["GND"] += gnd

    # Connect capacitors
    cin[1] += vin
    cin[2] += gnd

    cout[1] += vout
    cout[2] += gnd

if __name__ == "__main__":
    power_supply()

    # Generate outputs
    from circuit_synth import generate_bom, generate_pdf_schematic

    generate_bom(project_name="Power_Supply_3V3")
    generate_pdf_schematic(project_name="Power_Supply_3V3")
```

## Tips


1. **Use meaningful net names**
   - Good: `vcc_3v3`, `usb_dp`, `spi_mosi`
   - Bad: `net1`, `sig`, `x`

2. **Break large circuits into subcircuits**
   - Power supply
   - MCU core
   - Peripherals
   - Communication interfaces

3. **Document assumptions in docstrings**
   ```python
   @circuit(name="MyBoard")
   def my_board():
       """
       Main board circuit.
       Assumes 5V USB input, generates 3.3V for logic.
       Max current: 500mA from USB, 300mA available for logic.
       """
   ```

4. **Validate incrementally**
   - Test simple circuits first
   - Add complexity gradually
   - Generate KiCad after each major addition

5. **Use patterns when possible**
   - Pre-verified, manufacturing-ready
   - Saves time and reduces errors
   - Customize with parameters

6. **Cross-reference documentation**
   - Link to datasheets in comments
   - Reference source docs for reverse-engineered designs
   - Keep Python code and docs in sync

## Resources

- **Documentation**: https://circuit-synth.readthedocs.io
- **GitHub**: https://github.com/circuit-synth/circuit-synth
- **KiCad Libraries**: https://kicad.github.io/symbols/
- **Component Search**: https://componentsearchengine.com/
