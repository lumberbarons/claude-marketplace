# SN74HC595 - 8-Bit Shift Registers With 3-State Output Registers

**Manufacturer:** Texas Instruments
**Datasheet:** https://www.ti.com/lit/ds/symlink/sn74hc595.pdf (SCLS041J – October 2021)
**Extracted:** 2026-01-04

---

## 1. General Information

**Component Family:** 74HC595

**Full Part Name:** SNx4HC595 8-Bit Shift Registers With 3-State Output Registers

**Functional Description:**
The SNx4HC595 devices contain an 8-bit, serial-in, parallel-out shift register that feeds an 8-bit D-type storage register. The storage register has parallel 3-state outputs. Separate clocks are provided for both the shift and storage register. The shift register has a direct overriding clear (SRCLR) input, serial (SER) input, and serial outputs for cascading. When the output-enable (OE) input is high, the outputs are in the high-impedance state. Both the shift register clock (SRCLK) and storage register clock (RCLK) are positive-edge triggered. If both clocks are connected together, the shift register is always one clock pulse ahead of the storage register.

**Variants Covered:**
- `SN54HC595FK` - 20-pin LCCC package, -55°C to 125°C (military), 8.89 mm × 8.89 mm
- `SN54HC595J` - 16-pin CDIP package, -55°C to 125°C (military), 21.34 mm × 6.92 mm
- `SN74HC595N` - 16-pin PDIP package, -40°C to 85°C (commercial), 19.31 mm × 6.35 mm
- `SN74HC595D` - 16-pin SOIC package, -40°C to 85°C (commercial), 9.90 mm × 3.90 mm
- `SN74HC595DW` - 16-pin wide SOIC package, -40°C to 85°C (commercial), 10.30 mm × 7.50 mm
- `SN74HC595DB` - 16-pin SSOP package, -40°C to 85°C (commercial), 6.20 mm × 5.30 mm
- `SN74HC595PW` - 16-pin TSSOP package, -40°C to 85°C (commercial), 5.00 mm × 4.40 mm

Package suffix codes: FK=LCCC ceramic, J=CDIP ceramic, N=PDIP plastic DIP, D=SOIC, DW=Wide SOIC, DB=SSOP, PW=TSSOP

**Key Features:**
- 8-bit serial-in, parallel-out shift register
- Wide operating voltage range of 2 V to 6 V
- High-current 3-state outputs can drive up to 15 LSTTL loads
- Low power consumption: 80 µA (maximum) ICC
- tpd = 13 ns (typical) propagation delay at VCC = 5V
- ±6 mA output drive at 5 V
- Low input current: 1 µA (maximum)
- Shift register has direct clear (SRCLR)
- Independent shift and storage register clocks
- Serial output (QH') for cascading multiple devices
- CMOS compatible inputs and outputs
- 3-state outputs for bus-oriented applications

---

## 2. Pinout

### Pinout: 16-Pin PDIP/SOIC/SSOP/TSSOP (D, N, NS, DB, DW, PW Packages)

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
| 9 | QH' | Output | HIGH | VCC | Serial data output (for cascading devices) |
| 10 | $\overline{\text{SRCLR}}$ | Input | LOW | VCC | Shift register clear (active LOW) |
| 11 | SRCLK | Input | HIGH | VCC | Shift register clock input (positive edge triggered) |
| 12 | RCLK | Input | HIGH | VCC | Storage register clock / latch (positive edge triggered) |
| 13 | $\overline{\text{OE}}$ | Input | LOW | VCC | Output enable (active LOW, enables 3-state outputs) |
| 14 | SER | Input | HIGH | VCC | Serial data input |
| 15 | QA | Output | HIGH | VCC | Parallel data output Q0 |
| 16 | VCC | Power | - | 2-6V | Positive supply voltage |

**Note:** All 16-pin packages share the same pinout. Pin numbering is consistent across D, N, NS, DB, DW, and PW package types.

### Pinout: 20-Pin LCCC (FK Package)

For the 20-pin LCCC package, see page 3 of the datasheet. The FK package has the same signal functions but different physical pin numbers due to the leadless chip carrier format. Key differences: VCC on pin 20, GND on pin 10, signals distributed around the perimeter with NC (no connection) pins at positions 1, 6, 11, and 16.

---

## 3. Usage Information

### Operating Sequence

**Initialization:**
1. Apply power (VCC) between 2V and 6V to pin 16 (pin 20 for FK package)
2. Connect GND (pin 8, pin 10 for FK) to ground
3. Assert $\overline{\text{SRCLR}}$ LOW (pin 10) to clear shift register asynchronously (all bits set to 0)
4. Release $\overline{\text{SRCLR}}$ to HIGH for normal operation
5. Set $\overline{\text{OE}}$ LOW (pin 13) to enable outputs, or HIGH to keep outputs in high-impedance state

**Normal Operation (Shifting and Latching Data):**
1. Set serial data on SER input (pin 14)
2. Apply positive-going edge on SRCLK (pin 11) to shift data into the first stage of shift register
3. Data propagates through shift register on each subsequent SRCLK rising edge
4. Repeat steps 1-2 for all 8 bits (each SRCLK rising edge advances the data)
5. Apply positive-going edge on RCLK (pin 12) to transfer all shift register contents to storage register simultaneously
6. Output pins QA-QH (pins 15, 1-7) update to reflect the stored data

**Note:** Both clocks (SRCLK and RCLK) are independent. The shift register can continue shifting new data while the storage register/outputs remain stable. If clocks are tied together, the shift register is one clock pulse ahead of the storage register.

**Cascading Multiple Devices:**
1. Connect QH' (pin 9) of first device to SER (pin 14) of second device
2. Connect all SRCLK pins (pin 11) together for common shift clock
3. Connect all RCLK pins (pin 12) together for simultaneous latch update
4. Optionally tie $\overline{\text{OE}}$ pins together for synchronized output enable
5. For N devices cascaded: shift N×8 bits before pulsing RCLK to update all outputs

### Timing Requirements

**At VCC = 4.5V to 5.5V, SN74HC595, TA = 25°C:**

| Parameter | Symbol | Min | Max | Unit | Notes |
|-----------|--------|-----|-----|------|-------|
| Clock frequency | fclock | - | 25 | MHz | Typical at VCC = 4.5V |
| SRCLK pulse width HIGH | tw(H) | 16 | 24 | ns | |
| SRCLK pulse width LOW | tw(L) | 16 | 24 | ns | |
| RCLK pulse width HIGH | tw(H) | 16 | 24 | ns | |
| RCLK pulse width LOW | tw(L) | 16 | 24 | ns | |
| SRCLR pulse width LOW | tw | 16 | 24 | ns | |
| SER setup time (before SRCLK↑) | tsu | 20 | 30 | ns | |
| SER hold time (after SRCLK↑) | th | 0 | - | ns | Zero minimum |
| SRCLK↑ before RCLK↑ setup | tsu | 15 | 23 | ns | For stable data transfer |
| SRCLR LOW before RCLK↑ | tsu | 10 | 15 | ns | |
| SRCLR HIGH before SRCLK↑ | tsu | 10 | 15 | ns | Release clear before shifting |

**Note:** The setup time from SRCLK↑ to RCLK↑ allows the storage register to receive stable data from the shift register. Clocks can be tied together, but shift register will be one clock ahead.

**At VCC = 2.0V, TA = 25°C:**
- Maximum clock frequency: 6 MHz (typical), 4.2 MHz (minimum)
- Pulse widths increase: tw = 80 ns (min), 120 ns (max)
- Setup times increase proportionally (tsu = 100-150 ns range)

**Propagation Delays (VCC = 4.5V, TA = 25°C, CL = 50pF):**

| From | To | Min | Typ | Max | Unit |
|------|----|----|-----|-----|------|
| SRCLK↑ | QH' | - | 17 | 32 | ns |
| RCLK↑ | QA–QH | - | 17 | 30 | ns |
| SRCLR↓ | QH' | - | 18 | 35 | ns |
| $\overline{\text{OE}}$↓ | QA–QH enable | - | 15 | 30 | ns |
| $\overline{\text{OE}}$↑ | QA–QH disable | - | 23 | 40 | ns |

**Output Transition Times (CL = 50pF):**
- Rise time (tr): 8 ns (typ), 12 ns (max)
- Fall time (tf): 8 ns (typ), 12 ns (max)

### Timing Diagrams

**Figure 6-1 (page 7): Complete Timing Diagram**

Shows comprehensive timing relationships including:
- Serial data (SER) shifting on SRCLK rising edges
- Parallel outputs (QA-QH) updating on RCLK rising edge
- Serial cascade output (QH') following shift register
- Asynchronous clear function ($\overline{\text{SRCLR}}$)
- 3-state output control ($\overline{\text{OE}}$)

The timing diagram illustrates proper setup/hold times, propagation delays, and the relationship between shift and storage register clocks. Hatched areas indicate high-impedance (3-state) output condition.

### Functional Modes

**Shift Mode:**
- Data shifts into the shift register on each SRCLK positive edge
- First stage receives SER data, subsequent stages shift existing data forward
- QH' outputs the last bit shifted out (8th stage)
- Storage register and parallel outputs (QA-QH) remain unchanged
- Allows continuous data streaming

**Latch/Storage Mode:**
- RCLK positive edge transfers all 8 bits from shift register to storage register in parallel
- All outputs QA-QH update simultaneously (if $\overline{\text{OE}}$ is LOW)
- Provides glitch-free output updates
- Shift register can continue receiving new data during storage

**Clear Mode:**
- $\overline{\text{SRCLR}}$ LOW asynchronously clears shift register (all stages → 0)
- Clearing occurs immediately, independent of clocks
- Storage register and outputs QA-QH are NOT affected by clear
- QH' goes LOW when shift register is cleared
- Used for initialization or error recovery

**Output Enable/Disable Mode (3-State):**
- $\overline{\text{OE}}$ LOW: Outputs QA-QH actively drive HIGH or LOW based on storage register
- $\overline{\text{OE}}$ HIGH: Outputs QA-QH enter high-impedance (Hi-Z) state
- Useful for bus sharing, preventing contention, or disabling outputs
- Internal shift and storage registers continue to operate and retain data
- QH' serial output is NOT affected by $\overline{\text{OE}}$ (always active)

**Function Table (from page 12):**

| SER | SRCLK | SRCLR | RCLK | OE | Function |
|-----|-------|-------|------|----|----------|
| X | X | X | X | H | Outputs QA–QH are disabled (Hi-Z) |
| X | X | X | X | L | Outputs QA–QH are enabled |
| X | X | L | X | X | Shift register is cleared |
| L | ↑ | H | X | X | First stage goes LOW, other stages shift |
| H | ↑ | H | X | X | First stage goes HIGH, other stages shift |
| X | X | X | ↑ | X | Shift register data stored in storage register |

(↑ = rising edge, H = HIGH, L = LOW, X = don't care)

---

## 4. Electrical Characteristics

### Absolute Maximum Ratings
*(Over operating free-air temperature range unless otherwise noted)*

**WARNING:** Stresses beyond those listed under Absolute Maximum Ratings may cause permanent damage to the device. These are stress ratings only. Functional operation of the device at these or any other conditions beyond those indicated under Recommended Operating Conditions is not implied. Exposure to absolute-maximum-rated conditions for extended periods may affect device reliability.

| Parameter | Symbol | Min | Max | Unit | Notes |
|-----------|--------|-----|-----|------|-------|
| Supply voltage | VCC | -0.5 | 7.0 | V | |
| Input clamp current | IIK | - | ±20 | mA | VI < 0 or VI > VCC |
| Output clamp current | IOK | - | ±20 | mA | VO < 0 or VO > VCC |
| Continuous output current | IO | - | ±35 | mA | VO = 0 to VCC |
| Continuous current through VCC or GND | - | - | ±70 | mA | |
| Junction temperature | TJ | - | 150 | °C | |
| Storage temperature | Tstg | -65 | 150 | °C | |

**Note:** Input and output voltage ratings may be exceeded if input and output current ratings are observed.

### Recommended Operating Conditions

| Parameter | Symbol | SN54HC595 | SN74HC595 | Unit | Conditions |
|-----------|--------|-----------|-----------|------|------------|
| | | Min / Nom / Max | Min / Nom / Max | | |
| Supply voltage | VCC | 2 / 5 / 6 | 2 / 5 / 6 | V | |
| Input HIGH voltage | VIH (VCC=2V) | 1.5 | 1.5 | V | Minimum |
| | VIH (VCC=4.5V) | 3.15 | 3.15 | V | Minimum |
| | VIH (VCC=6V) | 4.2 | 4.2 | V | Minimum |
| Input LOW voltage | VIL (VCC=2V) | 0.5 | 0.5 | V | Maximum |
| | VIL (VCC=4.5V) | 1.35 | 1.35 | V | Maximum |
| | VIL (VCC=6V) | 1.8 | 1.8 | V | Maximum |
| Input voltage | VI | 0 to VCC | 0 to VCC | V | |
| Output voltage | VO | 0 to VCC | 0 to VCC | V | |
| Input rise/fall time | Δt/Δv (VCC=2V) | - / 1000 | - / 1000 | ns | Maximum |
| | Δt/Δv (VCC=4.5V) | - / 500 | - / 500 | ns | Maximum |
| | Δt/Δv (VCC=6V) | - / 400 | - / 400 | ns | Maximum |
| Operating temperature | TA | -55 / - / 125 | -40 / - / 85 | °C | |

**Important:** All unused inputs of the device must be held at VCC or GND to ensure proper device operation. See TI application report SCBA004, "Implications of Slow or Floating CMOS Inputs."

### DC Electrical Characteristics
*(Over recommended operating free-air temperature range, unless otherwise noted)*

**At VCC = 5V, TA = 25°C:**

| Parameter | Test Conditions | Min | Typ | Max | Unit |
|-----------|----------------|-----|-----|-----|------|
| VOH (IOH = -20 µA) | VI = VIH or VIL | 4.4 | 4.999 | - | V |
| VOH (QH', IOH = -4 mA) | VCC = 4.5V | 3.98 | 4.3 | - | V |
| VOH (QA-QH, IOH = -6 mA) | VCC = 4.5V | 3.98 | 4.3 | - | V |
| VOL (IOL = 20 µA) | VI = VIH or VIL | - | 0.001 | 0.1 | V |
| VOL (QH', IOL = 4 mA) | VCC = 4.5V | - | 0.17 | 0.4 | V |
| VOL (QA-QH, IOL = 6 mA) | VCC = 4.5V | - | 0.17 | 0.4 | V |
| II (input leakage) | VI = VCC or 0, VCC = 6V | - | ±0.1 | ±1000 | nA |
| IOZ (3-state leakage) | VO = VCC or 0, VCC = 6V | - | ±0.01 | ±10 | µA |
| ICC (quiescent) | VI = VCC or 0, IO = 0, VCC = 6V | - | 8 | 80 | µA |
| CI (input capacitance) | VCC = 2V to 6V | - | 3 | 10 | pF |

**At VCC = 6V:**
- VOH (QH', IOH = -5.2 mA): 5.48V typ, 5.8V min
- VOH (QA-QH, IOH = -7.8 mA): 5.48V typ, 5.8V min
- VOL (QH', IOL = 5.2 mA): 0.15V typ, 0.26V max
- VOL (QA-QH, IOL = 7.8 mA): 0.15V typ, 0.26V max

**Drive Capabilities:**
- Can drive up to 15 LSTTL loads
- Output source/sink current: ±6 mA at VCC = 5V

### AC Electrical Characteristics / Switching Characteristics
*(Over recommended operating free-air temperature range)*

**At VCC = 4.5V to 5.5V, TA = 25°C, CL = 50pF:**

| Parameter | From | To | Min | Typ | Max | Unit |
|-----------|------|----|----|-----|-----|------|
| fmax | - | - | 21 | 31 | - | MHz |
| tpd | SRCLK | QH' | - | 17 | 32 | ns |
| tpd | RCLK | QA–QH | - | 17 | 30 | ns |
| tPHL | SRCLR | QH' | - | 18 | 35 | ns |
| ten | $\overline{\text{OE}}$ | QA–QH | - | 15 | 30 | ns |
| tdis | $\overline{\text{OE}}$ | QA–QH | - | 23 | 40 | ns |
| tt (QA–QH) | - | - | - | 8 | 12 | ns |
| tt (QH') | - | - | - | 8 | 15 | ns |

**At CL = 150pF (heavier load):**
- tpd (RCLK to QA-QH): 22 ns (typ), 40 ns (max)
- ten: 23 ns (typ), 40 ns (max)
- tt (QA-QH): 17 ns (typ), 42 ns (max)

**Operating Characteristics:**
- Power dissipation capacitance (Cpd): 400 pF (typical, no load)

---

**End of Datasheet Extraction**

**Note:** This summary is extracted from the SN74HC595 datasheet (SCLS041J, October 2021). For complete information including:
- Detailed timing diagrams (Figure 6-1, page 7)
- Parameter measurement waveforms (page 10)
- Complete package mechanical drawings (pages 23-40)
- Full electrical specifications across all temperatures and voltages
- Application circuit details (pages 13-14)

Please refer to the original datasheet at: https://www.ti.com/lit/ds/symlink/sn74hc595.pdf
