# SN74HC595 - 8-Bit Shift Register with Output Latches

**Manufacturer:** Texas Instruments
**Datasheet:** https://www.ti.com/lit/ds/symlink/sn74hc595.pdf
**Extracted:** 2026-01-04

---

## 1. General Information

**Component Family:** 74HC595

**Full Part Name:** SN74HC595 8-Bit Shift Registers With 3-State Output Registers

**Functional Description:**
The SN74HC595 is an 8-bit serial-in, parallel-out shift register with output storage registers and 3-state outputs. Data is shifted into the device on the positive-going edge of the shift register clock (SRCLK). The data in the shift register is transferred to the storage register on a positive-going edge of the register clock (RCLK). Both clocks are independent, allowing the shift register to continue shifting data while the storage register remains stable.

**Variants Covered:**
- `SN74HC595N` - 16-pin PDIP package, -40°C to 85°C
- `SN74HC595D` - 16-pin SOIC package, -40°C to 85°C
- `SN74HC595DR` - 16-pin SOIC package, tape and reel, -40°C to 85°C
- `SN74HC595PW` - 16-pin TSSOP package, -40°C to 85°C
- `SN74HC595PWR` - 16-pin TSSOP package, tape and reel, -40°C to 85°C

**Key Features:**
- 8-bit serial input, parallel output
- Independent shift and storage register clocks
- 3-state outputs for bus-oriented applications
- Direct clear function
- Wide operating voltage: 2V to 6V
- High-speed operation: 25 MHz typical at VCC = 4.5V
- Low power consumption
- CMOS compatible inputs and outputs

---

## 2. Pinout

### Pinout: 16-Pin DIP/SOIC/TSSOP (All Packages)

| Pin | Name | Type | Active | Voltage | Description |
|-----|------|------|--------|---------|-------------|
| 1 | QB | Output | HIGH | VCC | Parallel data output Q1 |
| 2 | QC | Output | HIGH | VCC | Parallel data output Q2 |
| 3 | QD | Output | HIGH | VCC | Parallel data output Q3 |
| 4 | QE | Output | HIGH | VCC | Parallel data output Q4 |
| 5 | QF | Output | HIGH | VCC | Parallel data output Q5 |
| 6 | QG | Output | HIGH | VCC | Parallel data output Q6 |
| 7 | QH | Output | HIGH | VCC | Parallel data output Q7 |
| 8 | GND | Power | - | 0V | Ground reference |
| 9 | QH' | Output | HIGH | VCC | Serial data output (for cascading) |
| 10 | $\overline{\text{SRCLR}}$ | Input | LOW | VCC | Shift register clear (active LOW) |
| 11 | SRCLK | Input | HIGH | VCC | Shift register clock input |
| 12 | RCLK | Input | HIGH | VCC | Storage register clock input (latch) |
| 13 | $\overline{\text{OE}}$ | Input | LOW | VCC | Output enable (active LOW, enables 3-state outputs) |
| 14 | SER | Input | HIGH | VCC | Serial data input |
| 15 | QA | Output | HIGH | VCC | Parallel data output Q0 |
| 16 | VCC | Power | - | 2-6V | Positive supply voltage |

**Note:** All packages (N, D, PW) share the same pinout.

---

## 3. Usage Information

### Operating Sequence

**Initialization:**
1. Apply power (VCC) between 2V and 6V
2. Assert $\overline{\text{SRCLR}}$ LOW to clear shift register (set all bits to 0)
3. Release $\overline{\text{SRCLR}}$ to HIGH for normal operation
4. Set $\overline{\text{OE}}$ LOW to enable outputs (or HIGH to keep outputs in high-impedance state)

**Normal Operation:**
1. Set serial data on SER input
2. Apply positive edge on SRCLK to shift data into shift register
3. Repeat steps 1-2 for all 8 bits
4. Apply positive edge on RCLK to transfer shift register contents to storage register (outputs)
5. Output pins QA-QH reflect the stored data

**Cascading Multiple Devices:**
1. Connect QH' (pin 9) of first device to SER (pin 14) of second device
2. Connect SRCLK and RCLK of both devices together
3. Shift 16 bits (or more) through the chain before latching with RCLK

### Timing Requirements

**At VCC = 4.5V to 5.5V, TA = 25°C:**

- **Maximum clock frequency:** fmax = 25 MHz (typical), 20 MHz (min)
- **SRCLK pulse width HIGH:** tw(H) = 20 ns (min)
- **SRCLK pulse width LOW:** tw(L) = 20 ns (min)
- **RCLK pulse width HIGH:** tw(H) = 20 ns (min)
- **RCLK pulse width LOW:** tw(L) = 20 ns (min)
- **SER setup time:** tsu = 15 ns (min) - data must be stable before SRCLK rising edge
- **SER hold time:** th = 5 ns (min) - data must remain stable after SRCLK rising edge
- **SRCLK to QH' propagation delay:** tpd = 13 ns (typical), 26 ns (max) at CL = 50pF
- **RCLK to QA-QH propagation delay:** tpd = 18 ns (typical), 34 ns (max) at CL = 50pF
- **$\overline{\text{SRCLR}}$ pulse width:** tw = 15 ns (min)
- **$\overline{\text{OE}}$ to output enable/disable:** tpd = 15 ns (typical), 30 ns (max) at CL = 50pF

**At VCC = 2.0V, TA = 25°C:**
- **Maximum clock frequency:** fmax = 5 MHz (typical)
- Setup and hold times increase proportionally with lower VCC

### Timing Diagrams

**Figure 1 (Shift Register Timing):** Shows the relationship between SRCLK, SER input, and QH' serial output. Data shifts through the register on each SRCLK rising edge. See datasheet Figure 1 for waveform diagram.

**Figure 2 (Storage Register Timing):** Shows RCLK latching data from shift register to outputs QA-QH. Outputs change on RCLK rising edge. See datasheet Figure 2 for waveform diagram.

**Figure 3 (Clear Timing):** Shows $\overline{\text{SRCLR}}$ clearing the shift register asynchronously. See datasheet Figure 3 for waveform diagram.

### Functional Modes

**Shift Mode:**
- Data shifts into the register on SRCLK rising edges
- Storage register and outputs remain unchanged

**Latch Mode:**
- RCLK rising edge transfers all 8 bits from shift register to storage register simultaneously
- Outputs update to reflect new data

**Clear Mode:**
- $\overline{\text{SRCLR}}$ LOW clears shift register asynchronously (all bits become 0)
- Storage register and outputs are not affected by clear

**Output Disable Mode:**
- $\overline{\text{OE}}$ HIGH places all outputs in high-impedance (3-state) mode
- Useful for bus sharing or preventing output contention
- Internal registers retain their data

---

## 4. Electrical Characteristics

### Absolute Maximum Ratings

| Parameter | Symbol | Min | Max | Unit |
|-----------|--------|-----|-----|------|
| Supply voltage | VCC | -0.5 | 7.0 | V |
| Input voltage | VI | -0.5 | VCC + 0.5 | V |
| Output voltage (3-state) | VO | -0.5 | VCC + 0.5 | V |
| Continuous output current | IO | - | ±35 | mA |
| Continuous VCC or GND current | - | - | ±70 | mA |
| Power dissipation | PD | - | 500 | mW |
| Operating temperature | TA | -40 | 85 | °C |
| Storage temperature | Tstg | -65 | 150 | °C |

**WARNING:** Stresses beyond those listed may cause permanent damage. Exposure to absolute maximum ratings for extended periods may affect device reliability.

### Recommended Operating Conditions

| Parameter | Symbol | Min | Typ | Max | Unit |
|-----------|--------|-----|-----|-----|------|
| Supply voltage | VCC | 2.0 | 5.0 | 6.0 | V |
| Input HIGH voltage | VIH | 1.5 | - | VCC | V |
| Input LOW voltage | VIL | 0 | - | 0.3·VCC | V |
| Operating temperature | TA | -40 | 25 | 85 | °C |

### DC Electrical Characteristics (VCC = 5V, TA = 25°C)

| Parameter | Symbol | Min | Typ | Max | Unit | Conditions |
|-----------|--------|-----|-----|-----|------|------------|
| Output HIGH voltage | VOH | 4.4 | 4.9 | - | V | IOH = -4 mA |
| Output LOW voltage | VOL | - | 0.1 | 0.4 | V | IOL = 4 mA |
| Input leakage current | II | - | - | ±0.1 | µA | VI = VCC or GND |
| Quiescent supply current | ICC | - | - | 8.0 | µA | VI = VCC or GND |
| Output source current | IOH | -4 | - | - | mA | VOH = 4.4V |
| Output sink current | IOL | 4 | - | - | mA | VOL = 0.4V |

### AC Electrical Characteristics (VCC = 4.5V to 5.5V, TA = 25°C, CL = 50pF)

| Parameter | Symbol | Min | Typ | Max | Unit | Conditions |
|-----------|--------|-----|-----|-----|------|------------|
| Maximum clock frequency | fmax | 20 | 25 | - | MHz | - |
| SRCLK to QH' propagation delay | tpd | - | 13 | 26 | ns | CL = 50pF |
| RCLK to output propagation delay | tpd | - | 18 | 34 | ns | CL = 50pF |
| Output rise time | tr | - | 6 | 15 | ns | CL = 50pF |
| Output fall time | tf | - | 6 | 15 | ns | CL = 50pF |

---

## 5. Package Information

### Package Types Available

| Package Code | Package Type | Pin Count | Description |
|--------------|--------------|-----------|-------------|
| N | PDIP (Plastic DIP) | 16 | Through-hole, 0.3" width |
| D | SOIC | 16 | Surface mount, wide body |
| PW | TSSOP | 16 | Surface mount, thin profile |

### Package Dimensions

**PDIP (N Package):**
- Length: 19.3 mm (nominal)
- Width: 6.35 mm (0.3" body width)
- Height: 3.9 mm (max)
- Pin pitch: 2.54 mm (0.1")

**SOIC (D Package):**
- Length: 9.9 mm (nominal)
- Width: 3.9 mm (body width)
- Height: 1.75 mm (max)
- Pin pitch: 1.27 mm (0.05")

**TSSOP (PW Package):**
- Length: 5.0 mm (nominal)
- Width: 4.4 mm (body width)
- Height: 1.2 mm (max)
- Pin pitch: 0.65 mm

### Thermal Characteristics

| Package | θJA (°C/W) | θJC (°C/W) |
|---------|-----------|-----------|
| PDIP (N) | 80 | 45 |
| SOIC (D) | 105 | 50 |
| TSSOP (PW) | 125 | 55 |

### Additional Package Information

- **Moisture Sensitivity Level:** MSL 1 (unlimited floor life at ≤30°C / 85% RH)
- **Lead finish:** Matte tin or NiPdAu
- **RoHS compliance:** RoHS compliant, lead-free
- **Peak reflow temperature:** 260°C (lead-free soldering)

---

## 6. Application Examples

### Typical Application Circuit

**Serial-to-Parallel Conversion (Figure 9 in datasheet):**

The most common application uses the 74HC595 to expand microcontroller I/O pins. The MCU uses 3 GPIO pins (data, clock, latch) to control 8 output pins.

**Circuit:**
- MCU GPIO1 → SER (pin 14): Serial data
- MCU GPIO2 → SRCLK (pin 11): Shift clock
- MCU GPIO3 → RCLK (pin 12): Latch clock
- VCC (pin 16) → +5V (or +3.3V)
- GND (pin 8) → Ground
- $\overline{\text{SRCLR}}$ (pin 10) → VCC (or MCU GPIO for clear control)
- $\overline{\text{OE}}$ (pin 13) → GND (outputs always enabled)
- QA-QH (pins 15, 1-7) → LEDs with current-limiting resistors, or other loads

**Component Values:**
- Current-limiting resistors for LED loads: 330Ω to 470Ω (for 5V operation, 20mA LED current)
- Decoupling capacitor: 0.1µF ceramic between VCC and GND, placed close to IC

### Cascading Multiple Shift Registers

**Daisy-Chain Configuration:**

To control more than 8 outputs, cascade multiple 74HC595 devices:

1. Connect QH' (pin 9) of first IC to SER (pin 14) of second IC
2. Connect all SRCLK pins together (parallel clock)
3. Connect all RCLK pins together (parallel latch)
4. For 16 outputs, shift 16 bits before latching
5. First 8 bits appear on second IC, next 8 on first IC

**Example for 24 outputs (3× 74HC595):**
- IC1 (closest to MCU) QH' → IC2 SER
- IC2 QH' → IC3 SER
- All SRCLK connected together
- All RCLK connected together
- Shift 24 bits, then pulse RCLK once to update all outputs simultaneously

### LED Matrix Driving

Use multiple 74HC595 devices for row/column scanning of LED matrices. One set of shift registers drives rows, another drives columns, with multiplexing controlled by the MCU.

### Layout Recommendations

- Place 0.1µF decoupling capacitor as close as possible to VCC pin
- Keep clock traces (SRCLK, RCLK) short and away from sensitive analog signals
- Use ground plane for digital ground return
- For high-speed operation (>10 MHz), consider series termination resistors (22-33Ω) on clock lines

### Design Considerations

1. **Output Current:** Each output can source/sink 4-6mA safely. For higher currents, use transistor buffers or MOSFET drivers.
2. **Propagation Delay:** When cascading, account for cumulative delays. At 25 MHz, 8-bit cascade adds ~100ns delay.
3. **Power Consumption:** Quiescent current is <10µA. Dynamic current depends on frequency and load capacitance.
4. **Voltage Compatibility:** Device works at 3.3V or 5V. When interfacing between voltage domains, ensure VIH/VIL compatibility.
5. **Clear Function:** Use $\overline{\text{SRCLR}}$ for known initial state. Consider pull-up resistor if controlled by open-drain output.

### Common Use Cases

- LED display drivers (7-segment, dot matrix, bar graphs)
- GPIO expansion for microcontrollers
- Control panel interfaces (multiple buttons/switches)
- Relay control boards
- LCD backlight control
- Addressable output expansion
- Bus buffering with 3-state control

---

**End of datasheet extraction. For detailed timing diagrams, package drawings, and application circuits, refer to the original datasheet.**
